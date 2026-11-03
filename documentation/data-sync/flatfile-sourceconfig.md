---
title: Flat File Source Configuration
description: Learn how to configure snapshot-based sources, including CSV and LDIF files, LDAP directories, and Data Sync Config directories.
---

# Flat File Source Configuration

This document explains how to configure pipeline source objects to read from **snapshots** instead of the real-time Data Sync Config path. Supported sources include CSV and LDIF files in the object store, external LDAP directories, and Data Sync Config directories.

It covers the configuration structure, source-specific examples, SQL transforms for CSV files, polling schedules, retry policies, and configuration fields. It also shows how to combine snapshot-based and real-time sources in a single pipeline.

To configure a snapshot-based source, add a `data-dock` block to the source object in your pipeline configuration. Configure vertices, edges, computed attributes, and reconciliation as usual.

For a source object with a `data-dock` block, `datasource` and `object-name` are user-defined names. The orchestrator uses them to create the event stream `idg_graph_<datasource>_<object-name>`, converting the name to lowercase and replacing hyphens (`-`), periods (`.`), forward slashes (`/`), and spaces with underscores (`_`).

## Configuration structure

The `data-dock` block defines the source, polling schedule, retry policy, optional transform, and comparison settings.

The source location determines where you declare it:

- Object-store sources: Use the top-level `source` block to specify the file location and `delta-sync.dataset` to define the file format.
- LDAP and Data Sync Config sources: Use `delta-sync.source` to define the directory source and search settings.

Declare the source in only one of these locations.

The following example shows the structure for an object-store source. The placeholders indicate where to add the SQL statements and CSV settings.

```
data-dock:
  source:
    type: OBJECT_STORE
    object-store:
      key-prefix: "hr" # Drop-zone folder
      key: "people.csv" # Literal filename, or a glob when a transform is configured
  scheduling:
    polling-interval-seconds: 300
  retry:
    max-attempts: 3
  transform: # Optional; required when key is a glob
    sql: [ ... ]
    output:
      from: result
  delta-sync: # Required comparison stage
    dataset:
      format: CSV
      csv: { ... }
```

For an LDAP or Data Sync Config source, omit the top-level `source`, `transform`, and `delta-sync.dataset` blocks. Define the source under `delta-sync.source` instead:

```
data-dock:
  scheduling:
    polling-interval-seconds: 300
  retry:
    max-attempts: 3
  delta-sync:
    source:
      type: IDDM
      iddm:
        base-dn: "ou=People,o=vds"
        object-class: inetOrgPerson
```

### Validation rules

The following rules are checked when the configuration is applied:

- `delta-sync` is required.
- Declare the source in exactly one location: the top-level `source` block for an object-store source, or `delta-sync.source` for an LDAP or Data Sync Config source.
- An object-store source must use `type: OBJECT_STORE` and specify a non-blank `key-prefix`.
- A glob `key`, containing `*`, `?`, `[` or `{`, requires a `transform` block. Without a transform, `key` must be a literal filename.
- A `transform` block is supported only for object-store sources.
- An object-store source requires `delta-sync.dataset`.
- When `format` is `CSV`, the dataset must include a `csv` block.

A source object with a `data-dock` block cannot use the source-level `filter` or `limits` fields. To limit the entries included in an LDAP or Data Sync Config snapshot, use `filter` in the corresponding `ldap` or `iddm` block.

### Attribute selection

Use the source object's `backend-attributes` to select the attributes to emit:

- CSV sources: Columns to retain, matched by name.
- LDIF sources: Attributes to retain.
- LDAP and Data Sync Config sources: Attributes to request from the directory server.

### Shared settings

Declare `scheduling` and `retry` once at the `data-dock` level. These settings apply to every stage.

## Combine snapshot-based and real-time sources

A single pipeline configuration can combine snapshot-based and real-time Data Sync Config sources. The following example builds an identity graph from four source objects:

- A CSV file in the object store.
- An LDIF file in the object store.
- A Data Sync Config snapshot.
- A real-time Data Sync Config source.

```
version: v1
templates:
  - ref: functions_v1

source-objects:
  # Single CSV file in the object store, without a transform
  - name: hr_employees
    datasource: hr-system
    object-name: employees
    data-dock:
      source:
        type: OBJECT_STORE
        object-store:
          key-prefix: "hr"
          key: "employees.csv"
      scheduling:
        polling-interval-seconds: 300
      delta-sync:
        dataset:
          format: CSV
          csv:
            separator: ","
            has-header: true
            key-columns: [employee_id]
            rdn-attribute: employee_id
            base-dn: "ou=employees,dc=hr"
            entity-type-constant: inetOrgPerson
    backend-attributes:
      - name: employee_id
      - name: email
      - name: department

  # LDIF file in the object store
  - name: directory_users
    datasource: corporate-directory
    object-name: users
    data-dock:
      source:
        type: OBJECT_STORE
        object-store:
          key-prefix: "exports"
          key: "users.ldif"
      scheduling:
        polling-interval-seconds: 600
      delta-sync:
        dataset:
          format: LDIF
          ignored-attributes:
            - createTimestamp
            - modifyTimestamp
    backend-attributes:
      - name: uid
      - name: cn
      - name: mail

  # Data Sync Config snapshot
  - name: vds_users
    datasource: identity-hub
    object-name: vds-users
    data-dock:
      scheduling:
        polling-interval-seconds: 60
      delta-sync:
        source:
          type: IDDM
          iddm:
            base-dn: "ou=People,o=vds"
            object-class: inetOrgPerson
            change-detection: CHANGELOG
    backend-attributes:
      - name: uid
      - name: cn
      - name: mail

  # Real-time Data Sync Config source, without a data-dock block
  - name: ad_users
    datasource: ad-corp
    object-name: userX
    backend-attributes:
      - name: cn
      - name: sAMAccountName
      - name: mail

vertices:
  - name: rt_identity
    linkage:
      - source-object-name: hr_employees
        vertex-uid: employee_id
        properties:
          - name: email
            attribute-name: email
      - source-object-name: directory_users
        vertex-uid: uid
        properties:
          - name: email
            attribute-name: mail
      - source-object-name: vds_users
        vertex-uid: uid
        properties:
          - name: email
            attribute-name: mail

  - name: rt_account
    linkage:
      - source-object-name: ad_users
        vertex-uid: sAMAccountName
        properties:
          - name: name
            attribute-name: cn
          - name: email
            attribute-name: mail

edges:
  - name: is_owned_by
    source-vertex-name: rt_account
    target-vertex-name: rt_identity
    rules:
      - name: account_identity_lookup
        event-sources: [ad_users]
        event-types: [INSERT]
        link-condition:
          predicates:
            - field: email
              operator: EQUALS
              value: mail
```

## Object-store source: CSV

To read a CSV file, specify its location in the object store and define its format in `delta-sync.dataset.csv`.

Each row becomes an entity with the DN `{rdn-attribute}={key},{base-dn}`, where `{key}` combines the `key-columns` values, separated by underscores (`_`).

The key is emitted as the `rdn-attribute`, not as separate key-column attributes. When using a single key column, set `rdn-attribute` to the same column name, as shown below. Use that attribute as the vertex's `vertex-uid`.

The following example reads a single CSV file without a transform:

```
data-dock:
  source:
    type: OBJECT_STORE
    object-store:
      key-prefix: "hr"
      key: "people.csv"
  scheduling:
    polling-interval-seconds: 5
  delta-sync:
    dataset:
      format: CSV
      csv:
        separator: ","
        has-header: true
        key-columns: [cn]
        rdn-attribute: cn
        base-dn: "dc=hr"
        entity-type-constant: inetOrgPerson
```

### CSV files without a header

For a CSV file without a header row, set `has-header` to `false` and use `columns` to list the column names in file order:

```
delta-sync:
  dataset:
    format: CSV
    csv:
      separator: ","
      has-header: false
      columns: [cn, uid, sn, mobile, mail, employeeNumber, employeeType, displayName]
      key-columns: [cn]
      rdn-attribute: cn
      base-dn: "dc=hr"
      entity-type-constant: inetOrgPerson
```

## Object-store source: LDIF

To read an LDIF file, specify its location in the object store and set `format` to `LDIF`. Optionally, use `ignored-attributes` to exclude attributes from comparison and prevent changes to those attributes from triggering unnecessary update events.

```
data-dock:
  source:
    type: OBJECT_STORE
    object-store:
      key-prefix: "exports"
      key: "users.ldif"
  scheduling:
    polling-interval-seconds: 600
  delta-sync:
    dataset:
      format: LDIF
      ignored-attributes:
        - createTimestamp
        - modifyTimestamp
```

## Object-store source: CSV with a transform

To combine multiple CSV files in the drop-zone folder, use a glob pattern for `key` and add a `transform` block. The transform uses SQL to combine the files into a single CSV dataset that matches the settings in `delta-sync.dataset.csv`.

The following example reads all matching CSV files, combines their columns by name, and requires the result to contain at least one row and the specified columns:

```
data-dock:
  source:
    type: OBJECT_STORE
    object-store:
      key-prefix: "hr-transform"
      key: "*.csv"
  scheduling:
    polling-interval-seconds: 30
  delta-sync:
    dataset:
      format: CSV
      csv:
        separator: ","
        has-header: true
        key-columns: [cn]
        rdn-attribute: cn
        base-dn: "dc=hr"
        entity-type-constant: inetOrgPerson
  transform:
    sql:
      - name: all_people
        sql: |
          CREATE VIEW all_people AS
          SELECT *
          FROM read_csv_auto(
            '${raw}/*.csv', header=true, union_by_name=true, all_varchar=true
          );
    output:
      from: all_people
    validation:
      min-row-count: 1
      required-columns: [cn, mail, employeeNumber]
```

## External LDAP source

To read snapshots from an external LDAP server or virtual view, define the connection and search settings under `delta-sync.source.ldap` and set `type` to `LDAP`.

Use `change-detection` to control when the directory data is exported again:

- `POLLING` (default): Performs a full export during every polling cycle. Use this mode for virtual naming contexts that have neither a changelog nor a CSN.
- `CSN`: Exports the data again only when the `csn-attribute` value on the `base-dn` entry changes. The default attribute is `contextCSN`.
- `CHANGELOG`: Reads the changelog, using `cn=changelog` by default, and filters entries by target DN. Falls back to a full export when the gap exceeds `changelog-max-gap`.

### LDAP source with polling

The following example connects to an LDAP server over SSL and performs a full export every 120 seconds:

```
data-dock:
  scheduling:
    polling-interval-seconds: 120
  retry:
    max-attempts: 5
    initial-delay-seconds: 10
  delta-sync:
    source:
      type: LDAP
      ldap:
        host: "ad.corp.example.com"
        port: 636
        use-ssl: true
        bind-dn: "cn=svc-sync,ou=ServiceAccounts,dc=corp,dc=example,dc=com"
        bind-password-file: "/secrets/ad-bind-password"
        base-dn: "ou=Groups,dc=corp,dc=example,dc=com"
        object-class: group
        page-size: 500
        change-detection: POLLING
```

### LDAP source with changelog detection

The following example checks the changelog every 120 seconds and falls back to a full export when the gap exceeds 50,000 entries:

```
data-dock:
  scheduling:
    polling-interval-seconds: 120
  delta-sync:
    source:
      type: LDAP
      ldap:
        host: "dsee.corp.example.com"
        port: 389
        bind-dn: "cn=Directory Manager"
        bind-password-file: "/secrets/dsee-bind-password"
        base-dn: "ou=People,dc=corp,dc=example,dc=com"
        object-class: inetOrgPerson
        change-detection: CHANGELOG
        changelog-base-dn: "cn=changelog"
        changelog-max-gap: 50000
```

### LDAP credentials

Supply the bind password through `bind-password-file`, which specifies the path to a mounted secret file. Inline passwords specified with `bind-password` are rejected.

If you specify `bind-dn`, you must also specify `bind-password-file`. To bind anonymously, omit `bind-dn`.

## Data Sync Config source

To read snapshots from Data Sync Config, set `type` to `IDDM` and define the search settings under `delta-sync.source.iddm`.

The source inherits its connection settings and credentials from the deployment. To override the credentials for a specific source, specify `bind-dn` and `bind-password-file` in its `iddm` block.

The following example checks for changes every 60 seconds using changelog detection:

```
data-dock:
  scheduling:
    polling-interval-seconds: 60
  delta-sync:
    source:
      type: IDDM
      iddm:
        base-dn: "ou=People,o=vds"
        object-class: inetOrgPerson
        change-detection: CHANGELOG
```

## Scheduling and retry

Use `scheduling` to control how often the source is checked for changes. Use `retry` to configure how failed attempts are retried. Declare both blocks at the `data-dock` level so that they apply to every stage.

```
data-dock:
  scheduling:
    polling-interval-seconds: 900
  retry:
    max-attempts: 5
    initial-delay-seconds: 60
    max-delay-seconds: 600
    backoff-multiplier: 2.0
    on-connection-error: true
    on-data-error: false
  delta-sync:
    source:
      type: IDDM
      iddm:
        base-dn: "ou=People,o=vds"
        object-class: inetOrgPerson
        change-detection: CSN
```

### Polling interval

`polling-interval-seconds` specifies how often the source is checked for changes. The default value, `-1`, uses the deployment's default polling interval. Any value less than or equal to `0` has the same effect.

The default polling interval is 30 seconds unless overridden by the deployment.

### Retry behavior

`max-attempts` specifies the total number of attempts, including the initial attempt.

Retries use exponential backoff. The delay before retry `n`, where `n = 1` is the first retry, is:

```
initial-delay-seconds × backoff-multiplier^(n - 1)
```

The delay does not exceed `max-delay-seconds`.

By default, connection errors are retried, but data errors, such as a malformed CSV file, are not. Data errors typically require the source data or configuration to be corrected.

## Field reference

The following fields are configured under a source object's `data-dock` block.

### `data-dock`

| Key | Description |
| --- | --- |
| `source` | Defines an object-store source using `type: OBJECT_STORE`. Cannot be used with `delta-sync.source`. |
| `scheduling` | Defines the polling interval for all stages. |
| `retry` | Defines the retry policy for all stages. |
| `transform` | Defines an optional SQL transform for an object-store source. Required when `key` is a glob pattern. |
| `delta-sync` | Required. Defines the comparison stage, including the file format or directory source. |

### `source.object-store`

| Key | Description |
| --- | --- |
| `key-prefix` | Required. Specifies the drop-zone folder in the object store. |
| `key` | Specifies a literal filename or glob pattern, such as `*.csv`. Without a transform, a literal filename is required. A glob pattern requires a transform. |

### `scheduling`

| Key | Default | Description |
| --- | --- | --- |
| `polling-interval-seconds` | `-1` | Interval, in seconds, between checks for source changes. A value less than or equal to `0` uses the deployment's default interval, which is 30 seconds unless overridden. |

### `retry`

| Key | Default | Description |
| --- | --- | --- |
| `max-attempts` | `3` | Maximum number of attempts, including the initial attempt. |
| `initial-delay-seconds` | `30` | Delay, in seconds, before the first retry. |
| `max-delay-seconds` | `600` | Maximum delay, in seconds, between retries. |
| `backoff-multiplier` | `2.0` | Multiplier applied to the delay for each subsequent retry. |
| `on-connection-error` | `true` | Whether to retry connection errors. |
| `on-data-error` | `false` | Whether to retry data errors. |

### `delta-sync.dataset`

| Key | Default | Description |
| --- | --- | --- |
| `format` | `LDIF` | File format. Supported values are `LDIF` and `CSV`. |
| `ignored-attributes` | None | Attributes to exclude from comparison. |
| `merge-queue-depth` | `0` | Number of entries buffered per sorted bucket during comparison. A value of `0` uses the default of `16`. Reduce this value when individual entries are several megabytes in size. |
| `csv` | None | CSV parsing settings. Required when `format` is `CSV`. |

### `delta-sync.dataset.csv`

| Key | Default | Description |
| --- | --- | --- |
| `separator` | `,` | Character used to separate column values. |
| `has-header` | `true` | Whether the CSV file includes a header row. |
| `columns` | None | Column names in file order. Required when `has-header` is `false`. |
| `key-columns` | None | Required. Columns whose values form the entity key, joined with underscores (`_`). These columns are not emitted under their original names. |
| `rdn-attribute` | `uid` | Attribute used for the relative distinguished name (RDN) in the entity DN. Contains the composed entity key. |
| `base-dn` | `dc=csv` | Base DN under which the entities are placed. |
| `entity-type-constant` | `inetOrgPerson` | Default `objectClass` value assigned to each row. |
| `entity-type-column` | None | Column that supplies the `objectClass` value instead of the constant. Rows without a value use `entity-type-constant`. |

### `delta-sync.source`

Use `type: LDAP` with an `ldap` block for an external LDAP directory. Use `type: IDDM` with an `iddm` block for a Data Sync Config directory.

LDAP sources specify their connection settings in the `ldap` block. Data Sync Config sources inherit their connection settings from the deployment.

The following fields belong in the corresponding `ldap` or `iddm` block. Fields marked "LDAP only" do not apply to `iddm`.

| Key | Default | Description |
| --- | --- | --- |
| `host` | None | LDAP only. Directory server hostname. |
| `port` | `389` | LDAP only. Directory server port. |
| `use-ssl` | `false` | LDAP only. Whether to use SSL for the connection. |
| `bind-dn` | None | DN used to bind to the directory. For LDAP sources, omit this field to bind anonymously. For Data Sync Config sources, use it to override the deployment's default bind DN. Requires `bind-password-file`. |
| `bind-password-file` | None | Path to a mounted secret file containing the bind password. Inline passwords are not supported. |
| `base-dn` | None | Required. Base DN for the subtree search. |
| `object-class` | `*` | Object class used to construct the filter `(objectClass=<value>)`. |
| `filter` | None | Full LDAP search filter. Overrides `object-class`. |
| `page-size` | `1000` | Number of entries requested per page. |
| `change-detection` | `POLLING` | Change detection mode. Supported values are `POLLING`, `CSN`, and `CHANGELOG`. |
| `csn-attribute` | `contextCSN` | Attribute on the `base-dn` entry used to detect changes in `CSN` mode. |
| `changelog-base-dn` | `cn=changelog` | Base DN of the changelog used in `CHANGELOG` mode. |
| `changelog-max-gap` | `10000` | Maximum changelog gap before falling back to a full export in `CHANGELOG` mode. |
