---
title: JSON Fan-Out
description: Learn how to decompose nested JSON attributes into multiple vertices and edges
---

# JSON Fan-Out Configuration

## Overview

JSON fan-out lets you decompose complex, nested JSON attributes from a single incoming event into multiple graph vertices and edges. Use it when an identity or access management system delivers rich JSON payloads, such as entitlements with nested permission arrays or resources with embedded member maps, and you need to create each inner element as its own vertex in the graph.

A regular source object maps one backend event (for example, an LDAP entry or a CSV row) to one or more vertices and their normalization edges. A **derived source object** extends this by declaring that a JSON attribute on a parent source object contains a collection of sub-elements, each of which becomes its own set of vertices and edges. The pipeline automatically:

- Parses and iterates the JSON attribute.
- Expands maps or arrays into individual entities.
- Computes unique identifiers for each entity.
- Tracks state so that additions, modifications, and removals are detected incrementally.
- Emits the appropriate `INSERT` and `DELETE` events.

For example, a CyberArk safe event whose `safeMembers` attribute lists member 25 with the `listAccounts` and `retrieveAccounts` permissions, and member 42 with the `manageSafe` permission, produces 3 `rt_permission` vertices: one per member and permission.

## Declare a derived source object

A derived source object is declared in the `derived-source-objects` section of the pipeline configuration, next to the `source-objects` section. It references a parent source object and describes how to extract entities from one of its JSON attributes.

```
derived-source-objects:
  - name: cyberark_safe_permissions # unique name for this derived source
    derived-from: cyberark_safes # parent source object
    iterate-json-attribute: safeMembers # JSON attribute to iterate
    # ... expansion, fields, inheritance ...
```

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `name` | Yes | Unique name of the derived source object. Used to reference it in vertex linkage and edge rules. |
| `derived-from` | Yes | Name of the parent source object, declared in `source-objects`. |
| `iterate-json-attribute` | Yes | Name of the parent attribute (backend, meta, or computed) that contains the JSON to iterate. |
| `json-path` | No | `$.<key>`: iterate the value of one key of a wrapper object (for example, `$.value`). Only a single key is supported. |
| `expand-map` | No | Expands a nested JSON object (map) within each iterated element. See [Expand maps](#expand-maps). |
| `expand-array` | No | Expands a nested JSON array within each iterated element. See [Expand arrays](#expand-arrays). |
| `json-fields` | No | Fields to extract from each iterated JSON element. See [JSON fields](#json-fields). |
| `inherit-attributes` | No | Parent attributes to carry into the derived entities. See [Inherited attributes](#inherited-attributes). |
| `computed-attributes` | No | Computed attributes derived from the extracted and inherited fields. See [Computed attributes](#computed-attributes). |
| `parent-filter` | No | Filter evaluated against the parent event before expansion. See [Parent filter](#parent-filter). |

> `expand-map` and `expand-array` are mutually exclusive. You can use one or neither, but not both on the same derived source object.

A derived source object takes its entries from its parent, so it cannot declare `datasource`, `object-name`, `backend-attributes`, `meta-attributes`, `filter`, `limits`, `stream-config` or `data-dock`. The attribute names it produces (JSON fields, inherited attributes, and expansion aliases) must be unique.

## Iterate over JSON

The `iterate-json-attribute` key points to the parent attribute that holds the JSON. The pipeline reads this attribute from the parent event and interprets it as a JSON array. Each element of the array can produce entities. A JSON value that is not an array is processed as a single element.

If the JSON array is wrapped in an outer object, use `json-path` to select the key holding the array:

```
iterate-json-attribute: safeMembers
json-path: "$.value"
```

Given the following attribute value, the pipeline navigates to `$.value` and iterates over the 2 member objects:

```
{
  "value": [
    { "memberId": "25", "memberName": "PVWAAppUsers" },
    { "memberId": "42", "memberName": "VaultAdmins" }
  ]
}
```

If the attribute directly contains a JSON array, omit `json-path`.

When the attribute is absent or empty, or `json-path` names a key that the JSON doesn't have, the event produces no entities. On an `UPDATE`, the entities it produced before are deleted. When the attribute is not valid JSON, the parent event's vertices are written, but not its edges or any derived entity.

**Multivalued attributes**

Data Sync Config might deliver JSON array elements as separate values of a multivalued attribute rather than a single JSON string. The pipeline reassembles the values into a JSON array before parsing, so each value must itself be valid JSON (an object, or a quoted string).

## Expand maps

Use `expand-map` when each iterated JSON element contains a nested object (map) that you want to fan out further. This is typical for permission structures where the map keys are permission names and the values indicate whether the permission is granted.

```
expand-map:
  field: permissions # key of the nested map within each iterated element
  key-alias: permissionName # attribute name assigned to each map key
  filter-value: "true" # optional: only keep entries whose value equals this
  filter-field: status # optional: filter on a specific field within the value object
  value-fields: # optional: extract additional fields from each map value
    - name: permissionType
```

Given the following iterated element and `filter-value: "true"`, the pipeline produces 2 derived entities: `manageSafe` and `manageSafeMembers`, each with `memberId` 42 and `memberName` VaultAdmins. `retrieveAccounts` is excluded because its value (`false`) doesn't match the filter.

```
{
  "memberId": "42",
  "memberName": "VaultAdmins",
  "permissions": {
    "manageSafe": true,
    "manageSafeMembers": true,
    "retrieveAccounts": false
  }
}
```

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `field` | Yes | Key of the nested map within the iterated element (for example, `permissions`). A single key, not a path. |
| `key-alias` | Yes | Attribute name that holds the map key in each derived entity. |
| `filter-value` | No | Only keep map entries whose value, as text, equals this string (case-insensitive). When the map value is a boolean, `"true"` matches `true`. |
| `filter-field` | No | When the map value is itself an object, compare this field of it with `filter-value` instead of the value as a whole. Entries whose value is not an object are skipped. Requires `filter-value`. |
| `value-fields` | No | List of JSON field declarations to extract from each map value when the value is an object. |

Only the fields declared in `json-fields` are carried from the iterated element.

## Expand arrays

Use `expand-array` when each iterated JSON element contains a nested array that you want to fan out. This is typical for SailPoint-style entitlements where each entitlement has an array of rights for a target resource.

```
expand-array:
  field: rights # key of the nested array within each iterated element
  element-alias: right # attribute name assigned to each array element
```

Given the iterated element `{ "target": "FileServer", "rights": ["read", "write", "execute"] }`, the pipeline produces 3 derived entities, with `right` set to `read`, `write`, and `execute`, and `target` set to `FileServer`.

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `field` | Yes | Key of the nested array within the iterated element. A single key, not a path. |
| `element-alias` | Yes | Attribute name that holds each array element's value. An element that is an object or an array gives an empty value. |

## JSON fields

`json-fields` declares which fields to extract from each iterated JSON element and make available as attributes on the derived entity. Only declared fields are extracted; all other JSON content is ignored. A field is read as text: an object or array field gives an empty value, and a null or missing field leaves the attribute absent.

```
json-fields:
  - name: memberId
  - name: memberName
  - name: memberType
```

If the JSON field name is not suitable as an attribute name, use `alias`. The `name` key supports dot notation to navigate into nested objects. When `alias` is not specified and `name` contains dots, the last segment is used as the attribute name (for example, `source.name` becomes `name`).

```
json-fields:
  - name: source.name
    alias: sourceName
```

**Extract from inherited attributes**

When the field you need is not inside the iterated JSON element but inside a JSON attribute inherited from the parent event, use `from-inherited` to specify which inherited attribute to parse:

```
inherit-attributes:
  - source # inherited from the parent; contains JSON like {"id":"SRC-1","name":"AD"}

json-fields:
  - name: target # extracted from the iterated element as usual
  - name: name
    alias: sourceName
    from-inherited: source # parse the "source" inherited attribute as JSON, then extract its "name" field
```

Given a parent event with `source = {"id":"SRC-1","name":"ActiveDirectory"}` and an iterated element with `target = "FileServer"`, the derived entity has `target` set to `FileServer` and `sourceName` set to `ActiveDirectory`.

> The attribute referenced by `from-inherited` must be listed in `inherit-attributes`. This is validated when the configuration is applied. When the inherited value is missing or is not valid JSON, the field is left out.

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `name` | Yes | JSON field name to extract. Supports dot notation for nested objects (for example, `source.id`). |
| `alias` | No | Attribute name to use in the derived entity. Defaults to the last segment of `name`. |
| `from-inherited` | No | Name of an inherited attribute to parse as JSON and extract from, instead of the iterated element. |

## Inherited attributes

`inherit-attributes` lists the attributes from the parent source object to carry into every derived entity. Use it to bring context from the parent event, such as identifiers, account IDs, or source metadata, into the derived entities, so that you can use them in vertex UIDs, computed attributes, or edge rules.

```
inherit-attributes:
  - safeId # the parent safe's identifier
```

Every derived entity produced from a parent event includes `safeId` with the value from that parent. For a multivalued attribute, only the first value is inherited. Any parent attribute can be inherited, including its computed attributes, and an inherited attribute keeps its cardinality from the parent.

Inherited attributes are essential to build unique composite keys. In the following example, without `safeId`, the permission ID would not be unique across different safes:

```
computed-attributes:
  - name: permissionId
    function: concat
    fields:
      - attribute-name: memberId
        auxiliary-attribute-names: [permissionName, safeId] # safeId is inherited
```

## Parent filter

A `parent-filter` applies a derived source object only to parent events whose specified attribute matches the expected value. If the filter doesn't match, the derived source object is skipped for that event.

This is useful when a single parent source object feeds multiple derived source objects, each responsible for a different subset of the data. For example, SailPoint accounts can come from different identity sources (Active Directory, Salesforce, and so on), each with its own set of attributes in the JSON. By defining separate derived source objects with parent filters, you can apply source-specific extraction logic.

```
derived-source-objects:
  - name: sp_account_attrs_ad
    derived-from: sp_accounts
    iterate-json-attribute: entitlements
    parent-filter:
      field: sourceName
      equals: "ActiveDirectory"
    # ... AD-specific json-fields, computed-attributes ...

  - name: sp_account_attrs_sf
    derived-from: sp_accounts
    iterate-json-attribute: entitlements
    parent-filter:
      field: sourceName
      equals: "Salesforce"
    # ... Salesforce-specific json-fields, computed-attributes ...
```

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `field` | Yes | The name of the parent event attribute to evaluate (its first value when multivalued). Must be a known attribute of the parent source object. |
| `equals` | Yes | The expected value. The comparison is case-insensitive. |

If a parent event matched a filter and a later `UPDATE` changes the filtered attribute so that it no longer matches, the pipeline removes all the derived entities it produced before. If the new value matches a different derived source object's filter, that derived source object produces new entities from the same event.

## Computed attributes

Computed attributes produce new values from the extracted and inherited fields. They are declared the same way as on regular source objects, including macro expressions. See [Included functions](./functions.md). The orchestrator checks them when the configuration is applied, and rejects a reference to an unknown attribute, a number of fields the function doesn't take, or a field of a type the function doesn't accept.

An inherited attribute keeps its type from the parent, and a computed attribute has the output type of its function. A JSON field or an expansion alias (`key-alias`, `element-alias`) has no declared type, so it is not type-checked.

```
computed-attributes:
  - name: permissionId
    function: concat
    fields:
      - attribute-name: memberId
        auxiliary-attribute-names: [permissionName, safeId]
```

This example concatenates `memberId`, `permissionName`, and `safeId` into a composite `permissionId` that uniquely identifies each permission across all safes and members.

**Cardinality**

Computed attributes support a `cardinality` key that controls how the graph handles a vertex whose `vertex-uid` is that attribute:

| **Cardinality** | **Behavior** |
|-----------------|--------------|
| `HIGH` (default) | Each unique value creates a distinct vertex. |
| `LOW` | Several entries can produce the same value and share the vertex. |

`LOW` cardinality is useful when many derived entities map to the same logical vertex. For example, SailPoint entitlements from the same source and target should all reference the same `rt_resource` vertex:

```
computed-attributes:
  - name: resourceId
    function: concat
    cardinality: LOW
    fields:
      - attribute-name: sourceName
        auxiliary-attribute-names: [target]
```

The pipeline counts, per vertex, the parent entries whose entities reference it:

- The vertex is created by the first parent entry that references it, and deleted when the last one stops referencing it.
- While several parent entries reference it, their updates do not change it.
- The count is kept per source object: when 2 source objects feed the same `LOW` vertex, each counts its own references, and either one can delete it.

## Vertex linkage

Derived source objects are referenced in vertex linkage exactly like regular source objects. The `vertex-uid` and `properties` reference attributes available on the derived entity (extracted fields, inherited attributes, or computed attributes).

```
vertices:
  - name: rt_permission
    linkage:
      - source-object-name: cyberark_safe_permissions # derived source object
        vertex-uid: permissionId # computed attribute
        properties:
          - name: name
            attribute-name: permissionName # extracted JSON field
```

A single vertex can have linkages from both regular and derived source objects. This is common for vertices such as `rt_resource` that can be created from different systems.

## Direct-link edges

Fan-out entities often need edges that connect them to vertices whose UIDs are known from the JSON data itself, for example linking a permission to the account that holds it. Use `direct-link` edge rules for this.

```
edges:
  - name: grants_right_to
    source-vertex-name: rt_account
    target-vertex-name: rt_permission
    rules:
      - name: sailpoint_account_right
        event-sources: [sailpoint_entitlement_rights]
        direct-link:
          vertex-uid: accountId # attribute on the derived entity whose value identifies the account at the other end
```

When the pipeline processes a derived entity from `sailpoint_entitlement_rights` with `accountId = "ACC-007"`, it creates a `grants_right_to` edge from the account whose `vertex-uid` value is `ACC-007` to the permission. The vertex at the other end must be declared in `vertices` and linked to the source object that produces it. The edge is written even when that vertex doesn't exist yet.

Direct-link rules apply to derived source objects only, and derived source objects only support direct-link rules: a rule with a `link-condition` cannot name a derived source object in its `event-sources`. A direct-link edge follows the lifecycle of the derived entity (see [Deletion and lifecycle](#deletion-and-lifecycle)), whatever the rule's `event-types`.

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `name` | Yes | Name of the rule, unique across the configuration. |
| `event-sources` | Yes | List of derived source object names that trigger the rule. |
| `event-types` | No | Not used by direct-link rules: the edge follows the lifecycle of the derived entity. |
| `direct-link.vertex-uid` | Yes | The attribute on the derived entity that holds the identifier of the vertex at the other end of the edge. |
| `filter` | No | Conditional filter. See [Edge filters](#edge-filters). |
| `event-vertex-role` | No | For recursive edges only. `SOURCE` means the vertex from the event is the edge's source and the direct-link target is the destination. `TARGET` (default) means the reverse. |
| `recursive-link-child-to-parent` | No | Deprecated. Use `event-vertex-role` instead. `true` maps to `TARGET` and `false` maps to `SOURCE`. When both are set, `event-vertex-role` takes precedence. |

**Recursive direct-link edges**

Direct-link rules also work on recursive edges, whose source and target vertices are of the same type, to model hierarchies (for example, `is_in_parent` between 2 `rt_permission` vertices). Because both ends share a vertex name, use `event-vertex-role` to indicate which end the vertex from the event represents:

| **Value** | **Description** |
|-----------|-----------------|
| `SOURCE` | The vertex from the event is the edge's source; `direct-link.vertex-uid` resolves the target. |
| `TARGET` (default) | The vertex from the event is the edge's target; `direct-link.vertex-uid` resolves the source. |

```
edges:
  - name: is_in_parent
    source-vertex-name: rt_permission
    target-vertex-name: rt_permission
    rules:
      - name: parent_ref_link
        event-sources: [entitlement_parent_refs]
        event-vertex-role: SOURCE # event = the child; direct-link target = the parent
        direct-link:
          vertex-uid: parentEntId
```

## Edge-only derived source objects

A derived source object doesn't have to map to a vertex of its own. When the only purpose of a fan-out is to produce edges between vertices that already exist, you can omit the derived source object from vertex linkage. The derived source object then only expands the parent's JSON attribute, carries the inherited attributes and extracted JSON fields, and feeds direct-link edge rules. This is useful for hierarchical relationships such as SailPoint entitlement sub-permissions or AWS IAM role-to-managed-policy references.

For an edge-only derived source object, the parent's `vertex-uid` attribute must be listed in `inherit-attributes`, so that the engine can identify the parent vertex. Otherwise, the orchestrator rejects the configuration.

```
source-objects:
  - name: sailpoint_entitlements
    datasource: sailpoint
    object-name: entitlements
    backend-attributes:
      - name: entitlementId
      - name: entitlementName
      - name: parentRefs # JSON array: [{"id": "ENT-1"}, {"id": "ENT-2"}]

derived-source-objects:
  - name: entitlement_parent_refs
    derived-from: sailpoint_entitlements
    iterate-json-attribute: parentRefs
    json-fields:
      - name: id
        alias: parentEntId
    inherit-attributes:
      - entitlementId # required so that the engine can identify the child rt_permission

vertices:
  - name: rt_permission
    linkage:
      - source-object-name: sailpoint_entitlements
        vertex-uid: entitlementId
        properties:
          - name: name
            attribute-name: entitlementName
        # entitlement_parent_refs has no linkage entry

edges:
  - name: is_in_parent
    source-vertex-name: rt_permission
    target-vertex-name: rt_permission
    rules:
      - name: parent_ref_link
        event-sources: [entitlement_parent_refs]
        event-vertex-role: SOURCE
        direct-link:
          vertex-uid: parentEntId
```

For every parent reference, the engine creates an `is_in_parent` edge from the current `rt_permission` (child) to the referenced parent `rt_permission`. Adding, removing, or modifying entries in the parent JSON array creates or deletes the corresponding edges, and deleting the parent entry removes all the edges produced by the derived source object.

A derived source object that maps to its own vertex can also drive direct-link rules on edges whose ends are not its own vertex. For those edges, the same rule applies: the parent's `vertex-uid` attribute must be listed in `inherit-attributes`.

## Edge filters

Edge rules can include a `filter` to create edges based on an attribute value of the derived entity. In the following example, the same derived entity creates an edge to either an `rt_account` or an `rt_group` depending on its `memberType` field.

```
rules:
  - name: cyberark_account_permission
    event-sources: [cyberark_safe_permissions]
    filter:
      field: memberType
      equals: "User"
    direct-link:
      vertex-uid: memberId

  - name: cyberark_group_permission
    event-sources: [cyberark_safe_permissions]
    filter:
      field: memberType
      equals: "Group"
    direct-link:
      vertex-uid: memberId
```

| **Key** | **Required** | **Description** |
|---------|--------------|-----------------|
| `field` | Yes | Attribute name on the derived entity to evaluate. |
| `equals` | Yes | Expected value, matched case-insensitively. A filter without `equals` matches nothing. |

## Normalization edges

Edges declared without `rules` between a vertex produced by a derived source object and another vertex of the same derived entity, or a vertex of its parent, are created as normalization edges.

```
edges:
  - name: belongs_to
    source-vertex-name: rt_permission
    target-vertex-name: rt_resource
```

If `rt_permission` and `rt_resource` are both produced from the same derived entity, or one from the derived entity and the other from its parent, a `belongs_to` edge is created between them. No edge is created between the vertices of 2 different entities. The edge follows the lifecycle of the derived entity, and its properties must be attributes of the parent source object.

## Deletion and lifecycle

The pipeline automatically handles the full lifecycle of derived entities:

- **INSERT**: When a parent event is first received, the JSON is expanded and all derived entities are emitted as `INSERT` events.
- **UPDATE**: When the parent event is updated, the pipeline expands the JSON again and compares the new set of entities with the previous set for that parent entry. Only the differences are emitted: new entities as `INSERT` and removed entities as `DELETE`.
- **DELETE**: When the parent event is deleted, all previously derived entities are emitted as `DELETE` events.

An entity is identified by all its attributes (inherited, extracted, expanded, and computed). A change to any of them, including an inherited value, is a `DELETE` of the old entity followed by an `INSERT` of the new one: derived vertices are never updated in place.

For example, if a member is removed from a CyberArk safe's `safeMembers` JSON, the pipeline detects the removal and deletes the corresponding permission vertices and edges, without requiring an explicit delete for each permission.

## Example: CyberArk safes

A CyberArk safe event with `safeId` `SAFE-1` carries the following `safeMembers` attribute:

```
{
  "value": [
    {
      "memberId": "25",
      "memberName": "PVWAAppUsers",
      "memberType": "Group",
      "permissions": { "retrieveAccounts": true, "listAccounts": true, "manageSafe": false }
    },
    {
      "memberId": "42",
      "memberName": "VaultAdmins",
      "memberType": "User",
      "permissions": { "manageSafe": true, "manageSafeMembers": true }
    }
  ]
}
```

```
source-objects:
  - name: cyberark_accounts
    datasource: cyberark
    object-name: accounts
    backend-attributes:
      - name: accountId
      - name: accountName

  - name: cyberark_groups
    datasource: cyberark
    object-name: groups
    backend-attributes:
      - name: groupId
      - name: groupName

  - name: cyberark_safes
    datasource: cyberark
    object-name: safes
    backend-attributes:
      - name: safeId
      - name: safeName
      - name: safeMembers

derived-source-objects:
  - name: cyberark_safe_permissions
    derived-from: cyberark_safes
    iterate-json-attribute: safeMembers
    json-path: "$.value"
    expand-map:
      field: permissions
      key-alias: permissionName
      filter-value: "true"
    json-fields:
      - name: memberId
      - name: memberName
      - name: memberType
    inherit-attributes:
      - safeId
    computed-attributes:
      - name: permissionId
        function: concat
        fields:
          - attribute-name: memberId
            auxiliary-attribute-names: [permissionName, safeId]

vertices:
  - name: rt_account
    linkage:
      - source-object-name: cyberark_accounts
        vertex-uid: accountId
        properties:
          - name: name
            attribute-name: accountName

  - name: rt_group
    linkage:
      - source-object-name: cyberark_groups
        vertex-uid: groupId
        properties:
          - name: name
            attribute-name: groupName

  - name: rt_resource
    linkage:
      - source-object-name: cyberark_safes
        vertex-uid: safeId
        properties:
          - name: name
            attribute-name: safeName

  - name: rt_permission
    linkage:
      - source-object-name: cyberark_safe_permissions
        vertex-uid: permissionId
        properties:
          - name: name
            attribute-name: permissionName

edges:
  - name: belongs_to
    source-vertex-name: rt_permission
    target-vertex-name: rt_resource

  - name: grants_right_to
    source-vertex-name: rt_account
    target-vertex-name: rt_permission
    rules:
      - name: cyberark_account_permission
        event-sources: [cyberark_safe_permissions]
        filter:
          field: memberType
          equals: "User"
        direct-link:
          vertex-uid: memberId

  - name: grants_right_to
    source-vertex-name: rt_group
    target-vertex-name: rt_permission
    rules:
      - name: cyberark_group_permission
        event-sources: [cyberark_safe_permissions]
        filter:
          field: memberType
          equals: "Group"
        direct-link:
          vertex-uid: memberId
```

The pipeline creates the following permission vertices. Each one also gets a `belongs_to` edge to the parent `rt_resource` (the safe).

| **permissionId** | **permissionName** | **Linked to** |
|------------------|--------------------|---------------|
| `25retrieveAccountsSAFE-1` | `retrieveAccounts` | The `rt_group` whose `groupId` is 25 |
| `25listAccountsSAFE-1` | `listAccounts` | The `rt_group` whose `groupId` is 25 |
| `42manageSafeSAFE-1` | `manageSafe` | The `rt_account` whose `accountId` is 42 |
| `42manageSafeMembersSAFE-1` | `manageSafeMembers` | The `rt_account` whose `accountId` is 42 |

## Example: SailPoint entitlements

A SailPoint entitlement event carries a `source` attribute (the JSON of an object that identifies the originating system) and a `directPermissions` attribute (the JSON of an array of permissions, each with a `target` and an inner `rights` array):

```
{
  "entitlementId": "ENT-001",
  "accountId": "ACC-007",
  "source": { "id": "SRC-AD", "name": "ActiveDirectory" },
  "directPermissions": [
    { "target": "FileServer", "rights": ["read", "write"] },
    { "target": "Database", "rights": ["query"] }
  ]
}
```

```
source-objects:
  - name: sailpoint_accounts
    datasource: sailpoint
    object-name: accounts
    backend-attributes:
      - name: accountId
      - name: accountName

  - name: sailpoint_entitlements
    datasource: sailpoint
    object-name: entitlements
    backend-attributes:
      - name: entitlementId
      - name: accountId
      - name: source
      - name: directPermissions

derived-source-objects:
  - name: sailpoint_entitlement_rights
    derived-from: sailpoint_entitlements
    iterate-json-attribute: directPermissions
    expand-array:
      field: rights
      element-alias: right
    json-fields:
      - name: target
      - name: name
        alias: sourceName
        from-inherited: source
    inherit-attributes:
      - entitlementId
      - accountId
      - source
    computed-attributes:
      - name: resourceId
        function: concat
        cardinality: LOW
        fields:
          - attribute-name: sourceName
            auxiliary-attribute-names: [target]
      - name: permissionId
        function: concat
        fields:
          - attribute-name: entitlementId
            auxiliary-attribute-names: [right, target]

vertices:
  - name: rt_account
    linkage:
      - source-object-name: sailpoint_accounts
        vertex-uid: accountId
        properties:
          - name: name
            attribute-name: accountName

  - name: rt_resource
    linkage:
      - source-object-name: sailpoint_entitlement_rights
        vertex-uid: resourceId
        properties:
          - name: name
            attribute-name: target

  - name: rt_permission
    linkage:
      - source-object-name: sailpoint_entitlement_rights
        vertex-uid: permissionId
        properties:
          - name: name
            attribute-name: right

edges:
  - name: belongs_to
    source-vertex-name: rt_permission
    target-vertex-name: rt_resource

  - name: grants_right_to
    source-vertex-name: rt_account
    target-vertex-name: rt_permission
    rules:
      - name: sailpoint_account_right
        event-sources: [sailpoint_entitlement_rights]
        direct-link:
          vertex-uid: accountId
```

The expansion works as follows:

1. **Iterate**: The pipeline reads `directPermissions` and iterates the 2 array elements.
2. **Expand the array**: For the first element (`target: FileServer`, `rights: [read, write]`), the `rights` array is expanded into 2 entities.
3. **Extract JSON fields**: `target` is extracted from the iterated element, and `sourceName` is extracted from the inherited `source` attribute (`ActiveDirectory`).
4. **Inherit**: `entitlementId`, `accountId`, and `source` are carried from the parent.
5. **Compute**: `resourceId` = `ActiveDirectoryFileServer` and `permissionId` = `ENT-001readFileServer`.

| **permissionId** | **right** | **target** | **resourceId** | **Linked to** |
|------------------|-----------|------------|----------------|---------------|
| `ENT-001readFileServer` | `read` | `FileServer` | `ActiveDirectoryFileServer` | The `rt_account` whose `accountId` is `ACC-007` |
| `ENT-001writeFileServer` | `write` | `FileServer` | `ActiveDirectoryFileServer` | The `rt_account` whose `accountId` is `ACC-007` |
| `ENT-001queryDatabase` | `query` | `Database` | `ActiveDirectoryDatabase` | The `rt_account` whose `accountId` is `ACC-007` |

2 `rt_resource` vertices are created with `LOW` cardinality: `ActiveDirectoryFileServer` and `ActiveDirectoryDatabase`. All 3 permissions get `belongs_to` edges to their resource. If another entitlement entry also targets `FileServer` from `ActiveDirectory`, no new resource vertex is created: the entry is counted as one more reference to it.
