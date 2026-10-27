---
title: Data-Dock
description: Learn how to read source objects from snapshots with Data-Dock
---

# Data-Dock Configuration

## Overview

Add a `data-dock` block to a source object of your pipeline configuration to read it from a snapshot instead of the real-time Data Sync Config path. A snapshot can be a CSV or LDIF file in the object store, or an LDAP or Data Sync Config directory polled on a schedule. The rest of the configuration (vertices, edges, computed attributes, and reconciliation) is written as usual.

On a Data-Dock source object, `datasource` and `object-name` are free names. They name the event stream `idg_graph_<datasource>_<object-name>` (lowercased, with `-`, `.`, `/` and spaces replaced with `_`), which the orchestrator creates.

## Structure of a data-dock block

```
data-dock:
  source: # object-store origin (top level)
    type: OBJECT_STORE
    object-store:
      key-prefix: "hr" # the drop-zone folder
      key: "people.csv" # literal filename, or a glob ("*.csv") when a transform is present
  scheduling:
    polling-interval-seconds: 300
  retry:
    max-attempts: 3
  transform: # object-store origins only; required by a glob key
    sql: [ ... ]
    output: { from: result }
  delta-sync: # the comparison stage (required)
    dataset: # CSV or LDIF format (object-store origins only)
      format: CSV
      csv: { ... }
    source: # LDAP or Data Sync Config origin (not at the top level)
      type: IDDM
      iddm: { ... }
```

The following rules are checked when the configuration is applied:

- `delta-sync` is required.
- Declare the origin in exactly one place: an object store (`type: OBJECT_STORE` with a non-blank `key-prefix`) in the top-level `source`, or an LDAP or Data Sync Config directory in `delta-sync.source`.
- A glob `key` (containing `*`, `?`, `[` or `{`) requires a `transform`. Without one, `key` must be a literal filename.
- A `transform` is only allowed on an object-store origin.
- An object-store origin requires `delta-sync.dataset`, and `format: CSV` requires its `csv` block.

A Data-Dock source object cannot declare the source object `filter` or `limits`. To scope an LDAP or Data Sync Config snapshot, use the `filter` key of its `ldap` or `iddm` block instead.

The source object's `backend-attributes` select what Data-Dock emits: the CSV columns kept (matched by name), the LDIF attributes kept, or the attributes requested from the LDAP or Data Sync Config server.

`scheduling` and `retry` are declared once at the `data-dock` level and apply to every stage.

## Mix Data-Dock and Data Sync Config sources

A single configuration can mix Data Sync Config and Data-Dock source objects. The following example uses one source object of each Data-Dock type (object-store CSV, object-store LDIF, and Data Sync Config snapshot) together with a regular Data Sync Config source object, and builds the identity graph from all of them.

```
version: v1
templates:
  - ref: functions_v1

source-objects:
  # CSV in the object store via Data-Dock (single file, no transform)
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

  # LDIF in the object store via Data-Dock
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

  # Data Sync Config snapshot via Data-Dock
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

  # Regular Data Sync Config source object (no data-dock block)
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

The following example reads one CSV file, with no transform. Each row becomes an entity with the DN `{rdn-attribute}={key},{base-dn}`, where the key is the `key-columns` values joined with `_`. Key columns are emitted only as the `rdn-attribute`, so a single key column is usually also the `rdn-attribute` (as below), and is what a vertex `vertex-uid` refers to.

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

For a CSV file without a header, declare the column names in file order with `columns`:

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

When the export is already in LDIF format, point to the file. Optionally, exclude attributes that always change so that they don't produce unnecessary update events.

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

When the drop-zone folder contains many CSV fragments rather than one file, use a glob `key` and add a `transform` block. The transform combines the fragments into the single CSV that `delta-sync.dataset.csv` describes, using SQL.

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

Declare an external LDAP server (or a virtual view) under `delta-sync.source`. `change-detection` selects how Data-Dock decides whether to export the data again:

- `POLLING` (default): Full export at every cycle. Required for virtual naming contexts that have no changelog and no CSN.
- `CSN`: Exports again only when the `csn-attribute` (`contextCSN` by default) of the `base-dn` entry changes.
- `CHANGELOG`: Reads `cn=changelog`, filtered by target DN. Falls back to a full export when the gap exceeds `changelog-max-gap`.

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

> Supply the bind password with `bind-password-file`, the path of a mounted secret file. An inline `bind-password` is rejected. A `bind-dn` requires a `bind-password-file`. Omit `bind-dn` to bind anonymously.

## Data Sync Config source

`type: IDDM` takes a snapshot of Data Sync Config. The connection and credentials come from the deployment, so the block only contains the search. Use `bind-dn` and `bind-password-file` to override the credentials for a specific source.

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

`scheduling` and `retry` are declared at the `data-dock` level and apply to every stage:

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

`polling-interval-seconds` defines how often the source is checked for changes. When it is left at the default (`-1`), or set to any value less than or equal to 0, the Data-Dock default interval is used (30 seconds unless the deployment changes it). `max-attempts` includes the first attempt.

Retries use exponential backoff: the nth retry waits `initial-delay-seconds` × `backoff-multiplier`^(n-1), up to `max-delay-seconds`. Data errors, such as a malformed CSV file, are not retried by default because they rarely resolve on their own.

## Field reference

All keys are declared under the `data-dock` block of a source object.

**data-dock**

| **Key** | **Description** |
|---------|-----------------|
| `source` | Object-store origin only (`type: OBJECT_STORE`). Mutually exclusive with `delta-sync.source`. |
| `scheduling` | Polling interval for all stages. |
| `retry` | Retry policy for all stages. |
| `transform` | Optional SQL transform on an object-store origin. Required when `key` is a glob. |
| `delta-sync` | Required. The comparison stage (dataset format, or LDAP or Data Sync Config source). |

**source.object-store**

| **Key** | **Description** |
|---------|-----------------|
| `key-prefix` | Drop-zone folder (required). |
| `key` | Literal filename (required without `transform`) or a glob such as `*.csv` (requires `transform`). |

**scheduling**

| **Key** | **Default** | **Description** |
|---------|-------------|-----------------|
| `polling-interval-seconds` | `-1` | A value less than or equal to 0 uses the Data-Dock default interval. |

**retry**

| **Key** | **Default** |
|---------|-------------|
| `max-attempts` | `3` |
| `initial-delay-seconds` | `30` |
| `max-delay-seconds` | `600` |
| `backoff-multiplier` | `2.0` |
| `on-connection-error` | `true` |
| `on-data-error` | `false` |

**delta-sync.dataset**

| **Key** | **Default** | **Description** |
|---------|-------------|-----------------|
| `format` | `LDIF` | `LDIF` or `CSV`. |
| `ignored-attributes` | None | Attributes excluded from the comparison. |
| `merge-queue-depth` | `0` | Entries buffered per sorted bucket during the comparison. `0` uses the default (16). Lower it for entries of several MB. |
| `csv` | None | CSV parser block (required when `format: CSV`). |

**delta-sync.dataset.csv**

| **Key** | **Default** | **Description** |
|---------|-------------|-----------------|
| `separator` | `,` | |
| `has-header` | `true` | |
| `columns` | None | Required when `has-header: false`. |
| `key-columns` | None | Required. Columns forming the entity key, joined with `_`. Not emitted under their own name. |
| `rdn-attribute` | `uid` | RDN attribute for the entity DN. Holds the composed key. |
| `base-dn` | `dc=csv` | Base DN under which entities are placed. |
| `entity-type-constant` | `inetOrgPerson` | Constant objectClass for every row. |
| `entity-type-column` | None | Derives the objectClass from a column instead. Rows without it use `entity-type-constant`. |

**delta-sync.source**

`type: LDAP` uses an `ldap` block (with `host`, `port`, and `use-ssl`). `type: IDDM` uses an `iddm` block, with the connection inherited from the Data Sync Config deployment. Both share the search keys below.

| **Key** | **Default** | **Description** |
|---------|-------------|-----------------|
| `host` | None | LDAP only. |
| `port` | `389` | LDAP only. |
| `use-ssl` | `false` | LDAP only. |
| `bind-dn` | None | LDAP: omit to bind anonymously. Data Sync Config: optional override of the default bind DN. Requires `bind-password-file`. |
| `bind-password-file` | None | Path of a mounted secret file. The only accepted form. |
| `base-dn` | None | Required. Base DN of the subtree search. |
| `object-class` | `*` | Object class filter shorthand: `(objectClass=<value>)`. |
| `filter` | None | Full LDAP filter (overrides `object-class`). |
| `page-size` | `1000` | |
| `change-detection` | `POLLING` | `POLLING`, `CSN`, or `CHANGELOG`. |
| `csn-attribute` | `contextCSN` | `CSN` mode. |
| `changelog-base-dn` | `cn=changelog` | `CHANGELOG` mode. |
| `changelog-max-gap` | `10000` | `CHANGELOG` mode: the gap (in change numbers) before falling back to a full export. |
