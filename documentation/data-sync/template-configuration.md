---
title: Template Configuration
description: Learn how to use configuration templates for graph pipelines
---

# Configuration Templates

The [pipeline configuration](./pipeline-configuration.md) document lists the core configuration options available for defining graph pipelines. A template is a piece of reusable configuration that can be referenced by the main templated configuration.
The ultimate purpose of a template is to ship predefined graph pipeline configurations for known and supported out-of-the-box backends like SailPoint, Entra ID, etc.

## System templates

System templates ship with the orchestrator. As of today, the orchestrator exposes 1 system template called [**functions_v1**](./functions.md), which contains the signatures of all computed attribute functions supported by the system.

Template names starting with `functions_` are reserved for system templates. Such a template cannot be added, updated, or deleted.

## Manage templates

Templates are listed in **Template Management**. You can also list, get, add, update, and delete templates using the orchestrator API.

Do not edit the predefined templates. Instead, add organization-specific changes in the **template-overload** section of your pipeline configuration.

A template name can contain only letters, digits, `_` and `-`. When you submit a template, the orchestrator validates it as a configuration of the version it declares:

- `version` is required, and the matching functions template (`functions_v1` for `v1`) must exist.
- Every computed attribute must call a function of that functions template, with arguments of the right number and types.
- Every vertex, property, and edge must exist in the graph schema.

A template cannot reference other templates. We currently support version v1 only: templates and templated configurations must declare `version: v1`.

Create one template per application or repository. If an application exposes accounts, groups, and permissions, the template defines a source object for each of these object classes and maps them to the corresponding vertices.

> If several applications share a similar schema, normalize their attribute names in Data Sync Config. This way, a single template can be reused for all of them.

## Template structure

A template is a YAML file built around 2 main sections: `source-objects` and `vertices`. Edges can also be defined in a template.

The `source-objects` section defines the data to extract from Data Sync Config, with 1 entry per object class:

| **Field** | **Description** |
|-----------|-----------------|
| `name` | Unique name of the source object. |
| `datasource` | Name of the data source in Data Sync Config. |
| `object-name` | The objectclass or table name to query. |
| `backend-attributes` | Attributes fetched from the data source as-is. Declare a `type` when an attribute is passed to a function that expects a type other than `STRING`. |
| `meta-attributes` | Static literal values, such as the repository name, constants, or regular expressions. |
| `computed-attributes` | Attributes derived from backend attributes and meta-attributes using [functions](./functions.md). |

The `vertices` section maps the source objects to the graph, with 1 entry per vertex:

| **Field** | **Description** |
|-----------|-----------------|
| `name` | The vertex name, for example `rt_account`, `rt_identity`, or `rt_group`. |
| `linkage` | Binds 1 or more source objects to the vertex. |
| `vertex-uid` | The source object attribute used as the unique identifier of the vertex. |
| `properties` | Maps vertex property names to source object attributes. |

See [pipeline configuration](./pipeline-configuration.md) for the full list of options.

## Write a templated configuration

The templated configuration is a YAML file that references templates, overloads them with environment-specific settings, and can add its own source objects, vertices, and edges.

| **Key** | **Description** |
|---------|-----------------|
| `version` | The configuration version, `v1`. |
| `templates` | The list of templates used, in order. Each entry holds the keys below. |
| `templates[].ref` | The name of the template. `functions_v1` is added automatically when it is not listed. |
| `templates[].template-transformations` | Optional. Renames applied to the template before it is used. See [Apply transformations to a template](#apply-transformations-to-a-template). |
| `templates[].template-overload` | Optional. Settings merged into the template. See [How an overload is merged](#how-an-overload-is-merged). |
| `source-objects`, `vertices`, `edges` | Optional. Configuration added next to the templates, for example edges linking the vertices of 2 templates. |

> A key of a `templates` entry other than `ref`, `template-overload` and `template-transformations` is ignored without an error. For example, a misspelled `template-overload` silently overloads nothing.

In this example, 2 templates are referenced to reuse predefined configurations for 2 types of identity backends, called **hr_db** and **ad_db**.
The **template-overload** sets the data source names of this environment and adds computed attributes to a source object of **ad_db**. Other common overrides include repository meta-attributes such as `repositoryName` and `repositoryDescription`.
An additional edge linking the vertices of the 2 templates is also defined.

Edges can be defined either in a template or in the templated configuration. Keep generic edges in the template, and define edges that depend on the deployment, such as reconciliation rules and conflict resolution, in the templated configuration.

```
version: v1
templates:
  # if not listed, the function template is automatically added to the configuration based on its version
  - ref: hr_db
    template-overload: # the template overload section is optional and can be used to overload the template configuration
      source-objects:
        - name: hr_people
          datasource: acme # the data source name of this environment, since the template uses a default one
        - name: hr_org
          datasource: acme
  - ref: ad_db
    template-overload:
      source-objects:
        - name: ad_users
          datasource: acme
          meta-attributes:
            - name: emailDomain
              literal: "@acme.com"
          computed-attributes: # adding an extra layer of logic on top of the template
            - name: mail
              function: concat
              compute-strategy: IF_EMPTY
              fields:
                - attribute-name: sAMAccountName
                  auxiliary-attribute-names: [emailDomain]
            - name: mail
              function: normalizeValue
              fields:
                - mail
        - name: ad_groups
          datasource: acme

edges: # an extra edge linking the hr vertices to the ad vertices
  - name: is_owned_by
    source-vertex-name: rt_account
    target-vertex-name: rt_identity
    recon-policy:
      conflict-resolution:
        name: arrival_date_based_conflict_resolution
        date-priorities:
          - date-property-name: arrival_date
            date-selector: EARLIER_WINS
      ownership-rules-sets:
        - name: ad_ruleset
          rules:
            - name: identity2account_email_lookup
              event-sources: [hr_people]
              event-types: [INSERT, UPDATE]
              confidence-level: 5
              link-condition:
                logical-operator: AND
                predicates:
                  - field: email
                    operator: EQUALS
                    value: mail
            - name: identity2account_name_lookup
              event-sources: [hr_people]
              event-types: [INSERT, UPDATE]
              confidence-level: 3
              link-condition:
                logical-operator: AND
                predicates:
                  - field: name
                    operator: EQUALS
                    value: cn
            - name: account2identity_email_lookup
              event-sources: [ad_users]
              event-types: [INSERT, UPDATE]
              confidence-level: 5
              link-condition:
                logical-operator: AND
                predicates:
                  - field: email
                    operator: EQUALS
                    value: mail
            - name: account2identity_name_lookup
              event-sources: [ad_users]
              event-types: [INSERT, UPDATE]
              confidence-level: 3
              link-condition:
                logical-operator: AND
                predicates:
                  - field: displayname
                    operator: EQUALS
                    value: cn
```

### How an overload is merged

A `template-overload` is merged into its template, after the template's transformations:

- **It modifies, it does not add.** Each entry of its `source-objects`, `vertices`, and `edges` must match an entry of the template, by the name it has after the transformations. Otherwise, the configuration is rejected, and the error lists the available names. Declare a new source object, vertex, or edge at the top level of the templated configuration instead.
- **Entries are matched by identity.** A source object, vertex, attribute, or property is matched by its `name`, a vertex linkage by its `source-object-name`, and a predicate by its `field`. A computed attribute is matched by its `name` and `function`, so a computed attribute with a new function is added next to the template's definitions. This is how the example above adds 2 definitions of `mail`. An edge is matched by its `name`, `source-vertex-name`, `target-vertex-name` and its type (with `rules`, with a `recon-policy`, or neither), so an edge overload must repeat them.
- **Values merge.** A matched entry is merged key by key: a value replaces the template's value, and an entry of a nested list that matches nothing (a new attribute, property, or rule) is added.
- **Lists of values are combined.** A list of plain values (`event-sources`, `event-types`, `fields`, and so on) receives the overload's values next to the template's values. An overload can add values to such a list, but cannot remove or replace them.

> A key-based rule takes exactly one predicate. An overload that writes a predicate with a different `field` adds a second predicate, and the configuration is rejected.

The templates are then merged into the configuration in their order. A source object, vertex, or edge that both the configuration and a template declare is merged the same way, with the template's values replacing the configuration's values.

## Apply transformations to a template

You can apply transformations to a template to reuse it with different settings. This can be useful to reuse the same template for multiple similar datasources.
Imagine you have 10 backends hosting the same database schemas, you can define 1 template and apply a transformation to configure the 10 different backends with the same graph configurations.

The transformations support 2 operations, `suffix` and `replace`. Each one lists the strings to change, and `with` gives what to append or to replace them with. Transformations are listed under `template-transformations` and are applied in their order, before the overload. The **template-overload** section must therefore reference the transformed names, for example `iddm_users_idx1` instead of `iddm_users`.

> A transformation rewrites the whole template text before it is read: every occurrence of each listed string, anywhere in the template, is changed, including inside longer names and values, and in keys. For example, `suffix: [ad]` with `_1` turns `ad_users` into `ad_1_users`, and `replace: [IDDM]` with `IDDM_1` turns `IDDM Mock server` into `IDDM_1 Mock server`. The strings are regular expressions, so a `.` matches any character. Choose strings that appear only where you want to change them.

When a template is referenced twice, each reference must rename its source objects. Otherwise, both references merge into the same objects. Objects that the transformations leave unchanged, such as the vertices and edges in the following example, are shared by both references.

In the following example, the template `template2` is used twice with transformations that suffix its source object names and replace its repository name.

```
version: v1
templates:
  - ref: template1
    template-overload:
      source-objects:
        - name: ad_users
          datasource: acme
        - name: ad_groups
          datasource: acme
  - ref: template2
    template-transformations:
      - suffix:
          - iddm_users
          - iddm_groups
        with: _idx1
      - replace:
          - IDDM
        with: IDDM_1
    template-overload:
      source-objects:
        - name: iddm_users_idx1
          datasource: fid1
        - name: iddm_groups_idx1
          datasource: fid1
  - ref: template2
    template-transformations:
      - suffix:
          - iddm_users
          - iddm_groups
        with: _idx2
      - replace:
          - IDDM
        with: IDDM_2
    template-overload:
      source-objects:
        - name: iddm_users_idx2
          datasource: fid2
        - name: iddm_groups_idx2
          datasource: fid2
```

The result holds the source objects `iddm_users_idx1`, `iddm_groups_idx1`, `iddm_users_idx2` and `iddm_groups_idx2`, with the repository names `IDDM_1` and `IDDM_2`. The `rt_account` and `rt_group` vertices each link the users and groups of all the sources, and the `is_in_group` rule takes its events from all the groups source objects.

Original template used as example:

```
version: v1

source-objects:
  - name: iddm_users
    datasource: ds
    object-name: user
    backend-attributes:
      - name: cn
      - name: uuid
      - name: description
      - name: mail
      - name: createdAt
    meta-attributes:
      - name: repositoryName
        literal: IDDM
      - name: repositoryDescription
        literal: IDDM Mock server
    computed-attributes:
      - name: createdAt
        function: convertLDAPGeneralizedDateTime
        fields:
          - createdAt

  - name: iddm_groups
    datasource: ds
    object-name: group
    backend-attributes:
      - name: cn
      - name: description
      - name: member
      - name: uuid
    meta-attributes:
      - name: repositoryName
        literal: IDDM
      - name: repositoryDescription
        literal: IDDM Mock server
    computed-attributes:
      - name: membershipCreationTime
        function: getDateTimeNow

vertices:
  - name: rt_account
    linkage:
      - source-object-name: iddm_users
        vertex-uid: uuid
        properties:
          - name: name
            attribute-name: cn
          - name: description
            attribute-name: description
          - name: email
            attribute-name: mail
          - name: creation_date
            attribute-name: createdAt

  - name: rt_group
    linkage:
      - source-object-name: iddm_groups
        vertex-uid: uuid
        properties:
          - name: name
            attribute-name: cn
          - name: description
            attribute-name: description

edges:
  - name: is_in_group
    source-vertex-name: rt_account
    target-vertex-name: rt_group
    rules:
      - name: membership_based_on_members
        event-sources: [iddm_groups]
        event-types: [INSERT, UPDATE]
        properties:
          - name: created_at
            attribute-name: membershipCreationTime
        link-condition:
          predicates:
            - field: dn
              operator: EQUALS
              value: member
```
