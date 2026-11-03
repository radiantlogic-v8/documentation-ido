---
title: Included Functions
description: Learn how to configure computed attributes and use the functions available to graph pipelines.
---

# Pipeline Functions

**Computed attributes** add values to a source object by deriving them at ingestion time rather than reading them directly from the source. Define them for each source object in the pipeline configuration. The graph pipeline engine evaluates them as each change event passes through the pipeline.

This page explains how to configure computed attributes, control their evaluation, and use the available functions.

## Two computation engines

Each computed attribute uses one of two engines, determined by the function it references:

- Predefined functions: Specialized functions organized by namespace (`date`, `string`, `ldap`, and `recon`) and declared by name in the functions template. See [Function catalog](#function-catalog).
- Macros engine: The `evaluateExpression` function evaluates a JavaScript-like expression enclosed in curly braces (`{}`).

```
computed-attributes:
  # Predefined function
  - name: disabled
    function: getUserAccountControlAttribute
    fields:
      - useraccountcontrol
      - uacDisabled
  # Macros engine
  - name: active
    function: evaluateExpression
    fields:
      - '"{!disabled}"'
```

## Configure a computed attribute

Each entry in a source object's `computed-attributes` section supports the following keys:

| Key | Required | Default | Description |
| --- | --- | --- | --- |
| `name` | Yes | | Name of the attribute to produce. |
| `function` | Yes | | Function to call: a function from the [function catalog](#function-catalog) or `evaluateExpression`. |
| `fields` | When the function takes inputs | | Ordered function inputs. See [Pass input fields](#pass-input-fields). Each active field must match the function's input type at the corresponding position. `evaluateExpression` requires exactly one active field: a quoted macro string or the name of a meta attribute whose literal contains the expression. |
| `compute-strategy` | No | `OVERIDE` | Controls how the computed value is reconciled with any existing value. See [Compute strategy](#compute-strategy). |
| `compute-phase` | No | Both phases | Restricts computation to the `INIT` or `EVENT_DRIVEN` phase. See [Compute phase](#compute-phase). |
| `event-types` | No | All | Restricts computation to specific event types: `INSERT`, `UPDATE`, or `DELETE`. See [Event types](#event-types). |
| `cardinality` | No | `HIGH` | Used when the computed attribute is a `vertex-uid`. See [Advanced behavioral settings](./pipeline-configuration.md#advanced-behavioral-settings). |

> Computed attributes are evaluated in declaration order. An attribute can use the output of another computed attribute declared earlier in the list. Multiple entries can use the same `name`; each entry computes that attribute in turn. Use this pattern to define `IF_EMPTY` fallbacks.

### Pass input fields

Each entry in `fields` can be one of the following:

- An attribute name: A backend attribute, meta attribute, or computed attribute from the same source object.
- An inline literal: A quoted string (`'"(00)"'`), integer (`1`), boolean (`true` or `false`), match operator name (`NOT_EQUALS`), Active Directory `userAccountControl` flag name (`ACCOUNTDISABLE`), or non-human account reconciliation rule name.
- An object: An object containing `attribute-name` and either of the following optional keys:
  - `active` (default: `true`): Set to `false` to use the attribute only as a dependency that triggers computation. See [Dependency management](#dependency-management).
  - `auxiliary-attribute-names`: Additional attributes whose values are appended to the values of `attribute-name`, allowing the field to be passed as a single list argument. All listed attributes must have the same type as `attribute-name`. The resulting field uses the list form of that type; for example, `STRING` becomes `STRING_ARRAY`. Use this setting to pass several attributes to a list parameter, such as the input to `concat`. To pass a single attribute as a list argument, specify `auxiliary-attribute-names: []`.

```
fields:
  - name_separator # Meta attribute
  - attribute-name: FirstName
    auxiliary-attribute-names: [LastName] # FirstName and LastName form one list argument
```

When passing a backend attribute to a function that expects a type other than `STRING`, declare the attribute's `type`, such as `LONG` or `INTEGER`, in the source object. The engine uses this type to cast the input value.

The configuration is validated when it is applied. Validation rejects:

- Unknown functions.
- Fields that reference unknown attributes.
- An incorrect number of active fields.
- Active fields whose types do not match the function's input types at the corresponding positions.

### How computed attributes are evaluated

For each change event, the engine first adds the meta attributes to the event. It then processes computed attributes in declaration order:

1. Skips the attribute if the event's phase or type does not match the configured settings.
2. Skips the attribute if required by the compute strategy.
3. Calls the function with the attribute's active fields.
4. Writes the result according to the compute strategy.

An `UPDATE` event contains both the changes to the entry and the full entry. The engine evaluates computed attributes against both, so the forwarded changes also include the computed attributes' new values. If an input value is missing for a non-string parameter, the function returns an empty result.

> If a function throws an error, such as an invalid argument, malformed input, or an unparseable date, the entire change event is rejected. The pipeline then continues with the next event. Use functions only with inputs that are reliably populated and well formed. A macro that references an unknown function, method, or property, passes an incorrect number of arguments, or uses a constructor is rejected earlier, when the configuration is applied or a template is saved.

## Behavioral settings

Behavioral settings control when each computed attribute is evaluated and how its result is written.

### Dependency management

Each `fields` entry defines an attribute on which the computation depends. The `active` flag controls how the dependency is used:

- `active: true` (default): Passes the field's value to the function as an input argument.
- `active: false`: Uses the field only as a dependency. A change to the attribute triggers computation, but its value is not passed to the function.

A field's auxiliary attributes are also dependencies and use the same `active` setting as the field. For `evaluateExpression`, dependencies are determined by the variables the macro reads, not by its `fields` entries.

Use `active: false` to record when a change occurs. In this pattern, the function takes no active inputs, and another attribute serves only as the trigger.

```
computed-attributes:
  - name: last_insertion_date
    function: getDateTimeNow # Takes no active inputs
    compute-phase: EVENT_DRIVEN
    event-types: [INSERT]
    fields:
      - attribute-name: member # A membership change triggers computation
        active: false # The member value is not passed to the function
  - name: last_deletion_date
    function: getDateTimeNow
    compute-phase: EVENT_DRIVEN
    event-types: [DELETE]
    fields:
      - attribute-name: member
        active: false
```

In this example, `event-types` matches the type of change to the trigger field. Adding a value to `member` is an `INSERT`; removing a value is a `DELETE`.

#### Dependency ordering

The engine evaluates computed attributes from top to bottom. A computed attribute can therefore use values produced by earlier entries.

A dependency on another computed attribute is resolved only if that attribute is declared earlier in the list. If a field references a computed attribute declared later, it reads the value already carried by the event.

```
meta-attributes:
  - name: permission_separator
    literal: "-"
computed-attributes:
  - name: permissionId # Computed first
    function: normalizeValue
    fields:
      - permissionName
  - name: permissionId # Uses the previously computed value
    function: concatWithSeparator
    compute-strategy: IF_NOT_EMPTY
    fields:
      - permission_separator
      - attribute-name: permissionId
        auxiliary-attribute-names: [cn]
```

### Compute strategy

`compute-strategy` determines whether an attribute is computed and how the result is reconciled with any existing value.

| Strategy | Computes when | Write behavior | Empty result |
| --- | --- | --- | --- |
| `OVERIDE` (default) | Always | Replaces the existing value with the function result. | Clears the attribute. |
| `MERGE` | Always | Combines computed and existing values without duplicates. | Leaves existing values unchanged. |
| `IF_EMPTY` | The attribute is absent or empty | Writes the result only when the attribute is absent or empty. | Writes nothing; the attribute remains absent. |
| `IF_NOT_EMPTY` | The attribute already has a value | Recomputes and replaces the value only when one already exists. | Clears the attribute. |

A result is considered empty if it is null, an empty string, or a list containing only empty strings. Empty strings are never written as attribute values. Strings containing only whitespace are written.

A cleared attribute is absent, so a later entry with the same name and `compute-strategy: IF_EMPTY` can populate it.

### Compute phase

A graph pipeline runs in two phases:

- Initial phase: Bulk-loads the current state.
- Event-driven phase: Processes real-time change events from connectors.

Use `compute-phase` to restrict computation to one phase.

| `compute-phase` | Runs during |
| --- | --- |
| Omitted | Both phases |
| `INIT` | Initial bulk load only |
| `EVENT_DRIVEN` | Real-time event processing after the initial load completes |

Combine `compute-phase` with `event-types` to further restrict computation. For example, a `last_deletion_date` attribute configured with `compute-phase: EVENT_DRIVEN` and `event-types: [DELETE]` remains empty during the bulk load and is populated only by real-time delete events.

### Event types

`event-types` matches the event type determined by the pipeline, which may differ from the type sent by the connector:

- A `MOVE` (rename) is treated as an `UPDATE`.
- An `UPDATE` for an entry the pipeline has not previously seen is treated as an `INSERT`.
- A `DELETE` for a source object linked to the identity vertex is treated as an `UPDATE`. The identity is deactivated rather than deleted, so `event-types: [DELETE]` never applies to that source object.

When a change affects a trigger field (`active: false`) or the computed attribute itself, `event-types` instead matches the type of value change:

- Added value: `INSERT`.
- Replaced value: `UPDATE`.
- Removed value: `DELETE`.

A change to an active field matches the event's own type, `UPDATE`. For `evaluateExpression`, each variable read by the macro is matched in the same way as a trigger field.

## Function catalog

The following tables describe each function and provide an example. For the input and output types used during configuration validation, see [Function signatures](#function-signatures). You can also retrieve the current list from the orchestrator endpoint `GET /identitygraph/templates/functions_v1`.

Functions process one record at a time. Most return null for null or empty input rather than throwing an error. Exceptions are noted below. If a function throws an error, the entire change event is rejected.

### Date functions

Date functions return values in the Identity Observability graph format: `yyyyMMddHHmmss` in UTC, with 14 digits, no fractional seconds, and no time zone.

| Function | Example (input → output) | Behavior |
| --- | --- | --- |
| `getDateTimeNow` | (no input) → `20260623203000` | Returns the current UTC time. Takes no `fields`. |
| `convertTimestamp` | `1659302400000` → `20220731212000` | Converts epoch milliseconds from a `LONG` input. Declare the backend attribute with `type: LONG`. |
| `convertDate` | `"29/Mar/2023:15:30:00 +0200"`, `"dd/MMM/yyyy:HH:mm:ss Z"` → `20230329133000` | Parses a value using the specified pattern and converts it to UTC. A zone token (`X`, `x`, `Z`, `z`, `O`, or `V`) applies the time zone. A date-only pattern without a zone token returns `000000` for the time. A date-only pattern with a zone token throws an error. Returns null if the value or pattern is null. Throws an error for an empty value, a value that does not match the pattern, or an invalid pattern. |
| `convertLDAPGeneralizedDateTime` | `"20230329153000.500Z"` → `20230329153000` | Parses an LDAP generalized time and removes fractional seconds. Assumes UTC when no time zone is specified. Returns null for null input. Throws an error for an empty value or unsupported layout. |

> For `convertDate` patterns, use an offset token rather than a quoted literal. For example, use `yyyy-MM-dd'T'HH:mm:ss[.SSSSSSSSS]X`, not `…'Z'`. A quoted `'Z'` treats every value as UTC and ignores its actual offset. The number of fractional-second digits represented by `S` is fixed, so variable-width fractions do not parse with a literal pattern. Use `convertDate` only with reliably populated date attributes: an empty value throws an error and causes the event to be rejected.

```
backend-attributes:
  - name: whenmodified
    type: LONG
  - name: arrivaldate
  - name: whenCreated
meta-attributes:
  - name: hrDateFormat
    literal: "dd/MM/yyyy"
computed-attributes:
  - name: membershipCreationTime
    function: getDateTimeNow
  - name: last_modification_date
    function: convertTimestamp
    fields:
      - whenmodified
  - name: arrival_date # "15/01/2024" -> 20240115000000
    function: convertDate
    fields:
      - arrivaldate
      - hrDateFormat
  - name: creation_date # "20230329153000.500Z" -> 20230329153000
    function: convertLDAPGeneralizedDateTime
    fields:
      - whenCreated
```

### String functions

| Function | Example (input → output) | Behavior |
| --- | --- | --- |
| `isBlank` | `"   "` → `true` | Returns `true` if the value is null, empty, or contains only whitespace. |
| `normalizeValue` | `" Hello "` → `hello` | Trims the value and converts it to lowercase. Returns null for null input and `""` for `""`. |
| `normalizeValues` | `[" Hello ", "WORLD"]` → `[hello, world]` | Trims each element and converts it to lowercase. Returns null for null input. Fails if any element is null. |
| `filterValue` | `"test"`, `"test"`, `EQUALS` → `test` | Returns the value if it matches the operator; otherwise returns null. Returns null for a null or empty value. Throws an error for an unknown operator, an invalid regular expression, or a null match string used with a regular expression operator. Operator names are case-sensitive. |
| `filterValues` | `["apple","banana","cherry-a"]`, `"banana"`, `EQUALS`, `false` → `[banana]` | Returns the matching values. The last argument controls whether duplicates are removed. Returns null for a null or empty list, or when no values match. |
| `filterDuplicates` | `["a","b","a"]` → `[a, b]` | Removes duplicates using case-sensitive comparison. Returns null for null or empty input. |
| `concatWithSeparator` | `"."`, `["john","doe"]` → `john.doe` | Joins values using the specified separator. Treats a null separator as `""`. |
| `concat` | `["john","doe"]` → `johndoe` | Joins values without a separator and skips missing values. Use `concatWithSeparator` when different input combinations could produce the same text. |
| `mergeRemovingDuplicates` | `["Apple","Banana"]`, `["Banana","Cherry","Apple"]` → `Apple`, `Banana`, `Cherry` | Merges two lists and removes duplicates using case-sensitive comparison. Output order is not guaranteed. |
| `extractValueFromRegex` | `"BW-Main-SharePointHR-Picture"`, `"^BW-Main[^-]*-([^,]*?)-(.*)$"`, `1` → `SharePointHR` | Returns the trimmed value of the capture group at the specified index, or null if there is no match. Throws an error if the group index exceeds the available groups or the regular expression is invalid. |
| `extractValueWithEscapeChar` | `"part1\;escaped;part2;part3"`, `";"`, `1`, `\` → `part2` | Returns the part at the specified index, using a literal separator and an escape character. Returns null if the index is out of range. |
| `extractValue` | `"part1;part2;part3"`, `";"`, `1` → `part2` | Works like `extractValueWithEscapeChar`, but uses a regular expression separator. Escape metacharacters such as `.`. Trailing empty parts are dropped. |
| `extractValuesWithEscapeChar` | `"part1\;escaped;part2;part3"`, `";"`, `[0,2]`, `\` → `[part1;escaped, part3]` | Returns the parts at the specified indices. Stops at the first out-of-range index. Returns null if the result is empty. |
| `extractValues` | `"part1;part2;part3"`, `";"`, `[0,2]` → `[part1, part3]` | Works like `extractValuesWithEscapeChar`, but uses a regular expression separator. |
| `jsonExtract` | `{"source": {"name":"Active Directory"}}`, `"source.name"` → `Active Directory` | Returns the value at the specified dot path. Converts non-text values to strings. Returns null for invalid JSON, a missing path, or a JSON null value. |
| `ordinals` | `["b","a"]`, `["a","b","c"]` → `[1, 0]` | Returns the index of each value in the reference list. Returns `-1` for a value that is not found. |
| `mapValue` | `"key1"`, `["key1","val1","key2","val2"]` → `val1` | Returns the value paired with the first matching key in a flat key/value list. Matches only keys, using case-sensitive comparison. Returns null if no key matches. When all list values come from meta attributes, configuration validation rejects an empty list or a list with an odd number of values. |

`filterValue` and `filterValues` support the following match operators:

- `EQUALS`
- `NOT_EQUALS`
- `CONTAINS`
- `NOT_CONTAINS`
- `STARTS_WITH`
- `NOT_STARTS_WITH`
- `ENDS_WITH`
- `NOT_ENDS_WITH`
- `IS_BLANK`
- `IS_NOT_BLANK`
- `MATCHES_REGEX`: Matches when the regular expression is found anywhere in the value.
- `NOT_MATCHES_REGEX`

Use `normalizeValue` for attributes used as vertex UIDs or in edge link conditions to ensure consistent casing during comparison.

To build a fallback value only when an attribute is empty, use `concat` with `compute-strategy: IF_EMPTY`:

```
meta-attributes:
  - name: emailDomain
    literal: "@acme.com"
computed-attributes:
  - name: mail
    function: normalizeValue
    fields:
      - mail
  - name: mail # Build from sAMAccountName and the domain only when mail is empty
    function: concat
    compute-strategy: IF_EMPTY
    fields:
      - attribute-name: sAMAccountName
        auxiliary-attribute-names: [emailDomain]
```

Use `extractValueFromRegex` to extract part of a value. For a group named `APP-ActiveDirectory-Read`, the following example produces `ActiveDirectory` as the resource name and `Read` as the permission name:

```
meta-attributes:
  - name: groupNameRegex
    literal: "^APP-([^-]+)-(.+)$"
  - name: resourceIdx
    literal: 1
    type: INTEGER
  - name: permissionIdx
    literal: 2
    type: INTEGER
computed-attributes:
  - name: resourceName
    cardinality: LOW
    function: extractValueFromRegex
    fields:
      - cn
      - groupNameRegex
      - resourceIdx
  - name: permissionName
    cardinality: LOW
    function: extractValueFromRegex
    fields:
      - cn
      - groupNameRegex
      - permissionIdx
```

### LDAP functions

| Function | Example (input → output) | Behavior |
| --- | --- | --- |
| `getUserAccountControlAttribute` | `528`, `"LOCKOUT"` → `true` | Returns `true` if the specified Active Directory `userAccountControl` flag is set. Flag names are case-insensitive. Throws an error for an unknown flag name. |
| `getMsDsUserAccountControlComputedAttribute` | `8388608`, `"UF_PASSWORD_EXPIRED"` → `true` | Returns `true` if the specified `ms-DS-User-Account-Control-Computed` flag is set. Flag names are case-insensitive. Throws an error for an unknown flag name. |
| `inferPrivilegedAccountFromUac` | `65536` → `true` | Returns `true` if any of these flags is set: `PASSWD_NOTREQD`, `DONT_EXPIRE_PASSWORD`, or `TRUSTED_FOR_DELEGATION`. |
| `getUserAccountControlAttributes` | `528`, `false`, `["LOCKOUT"]` → `true` | Combines the specified flags using AND when the boolean argument is `true`, or OR when it is `false`. With no flag names, AND returns `true` and OR returns `false`. |
| `convertBase64ObjectGuidToUuid` | `"{base64binary}AAECAwQFBgcICQoLDA0ODw=="` → `00010203-0405-0607-0809-0a0b0c0d0e0f` | Converts an Active Directory `objectGUID` to a UUID. Accepts a value marked with `{base64binary}`, a 32-character hexadecimal string, or a Base64 value. Throws an error for a null or empty value, invalid Base64, or a value that is not 16 bytes. |
| `convertBase64GuidToHex` | `"{base64binary}AAECAwQFBgcICQoLDA0ODw=="` → `000102030405060708090a0b0c0d0e0f` | Converts a GUID to lowercase hexadecimal. Accepts a value marked with `{base64binary}` or a Base64 value, but not a hexadecimal string. |
| `normalizeDn` | `"cn=John Doe,ou=Users,dc=example,dc=com"` → `cn=john doe,ou=users,dc=example,dc=com` | Returns the canonical normalized DN. Returns null for null input. Throws an error for an invalid DN. |
| `normalizeDns` | `["cn=John Doe,ou=Users,dc=example,dc=com"]` → `[cn=john doe,ou=users,dc=example,dc=com]` | Normalizes each DN. Returns null for null input. Throws an error if the list contains a null or invalid DN. |
| `filterDescendentOf` | `"cn=John Doe,ou=Users,dc=example,dc=com"`, `"ou=Users,dc=example,dc=com"` → `cn=john doe,ou=users,dc=example,dc=com` | Returns the normalized DN if it is a descendant of, or equal to, the base DN. Otherwise returns null. |
| `filterDescendentsOf` | `["cn=John Doe,ou=Users,…", "cn=Admin,ou=Admins,…"]`, `"ou=Users,dc=example,dc=com"` → `[cn=john doe,ou=users,…]` | Returns the normalized descendant DNs, or null if none match. |
| `getParent` | `"cn=John Doe,ou=Users,dc=example,dc=com"` → `ou=Users,dc=example,dc=com` | Returns the parent DN as written in the input, without normalization. Returns null if there is no parent. |
| `getRdnValue` | `"cn=John Doe,ou=Users,dc=example,dc=com"` → `John Doe` | Returns the first attribute value in the leftmost RDN. |

Use `normalizeDns` for multi-valued DN attributes, such as a group's `member` attribute, when they are used in an edge link condition.

Pass flag names to `getUserAccountControlAttribute` through meta attributes. Examples include `ACCOUNTDISABLE`, `LOCKOUT`, `PASSWORD_EXPIRED`, `PASSWD_NOTREQD`, `PASSWD_CANT_CHANGE`, `DONT_EXPIRE_PASSWORD`, and `SMARTCARD_REQUIRED`.

```
backend-attributes:
  - name: useraccountcontrol
    type: INTEGER
  - name: member
meta-attributes:
  - name: uacLockout
    literal: "LOCKOUT"
computed-attributes:
  - name: locked
    function: getUserAccountControlAttribute
    fields:
      - useraccountcontrol
      - uacLockout
  - name: privileged_account
    function: inferPrivilegedAccountFromUac
    fields:
      - useraccountcontrol
  - name: member
    function: normalizeDns
    fields:
      - member
```

### Reconciliation functions

| Function | Example (input → output) | Behavior |
| --- | --- | --- |
| `tagNonHumanAccountWithRuleName` | `"svc-account-rule"`, `"CN=svc01"` → `svc-account-rule` | Returns the rule name if the second value is neither null nor empty. Otherwise returns null. The reconciliation policy uses this function to tag non-human accounts. |

## Examples

The following examples demonstrate how behavioral settings affect computed attributes.

### Preserve an existing value or populate a fallback

Use `IF_EMPTY` to preserve an existing `DisplayName` or populate it from a computed full name when it is absent.

```
computed-attributes:
  # Build "First, Last"
  - name: js_fullname
    function: concatWithSeparator
    fields:
      - name_separator # Meta attribute literal ", "
      - attribute-name: FirstName
        auxiliary-attribute-names: [LastName]
  # Use the computed full name only when DisplayName is absent
  - name: DisplayName
    function: concat
    compute-strategy: IF_EMPTY
    fields:
      - attribute-name: js_fullname
        auxiliary-attribute-names: []
```

| Input attributes | `js_fullname` | `DisplayName` output | Explanation |
| --- | --- | --- | --- |
| `FirstName=Alice`, `LastName=Anderson`, `DisplayName=Alice M Anderson` | `Alice, Anderson` | `Alice M Anderson` | Computation is skipped because `DisplayName` already has a value. |
| `FirstName=Bob`, `LastName=Brown` (no `DisplayName`) | `Bob, Brown` | `Bob, Brown` | `DisplayName` is empty, so it is populated with `js_fullname`. |

### Remove a placeholder value

Use `filterValue` with `NOT_EQUALS` to exclude a placeholder email address.

```
computed-attributes:
  - name: clean_normalized_mail
    function: filterValue
    fields:
      - normalized_mail
      - noemail_sentinel # Meta attribute literal "noemail@example.com"
      - NOT_EQUALS
```

| `normalized_mail` | `clean_normalized_mail` output |
| --- | --- |
| `alice@example.com` | `alice@example.com` |
| `noemail@example.com` | null (not emitted) |

### Extract, map, and apply a default

Use a regular expression to extract a marker, map the marker to a flag, and apply an `IF_EMPTY` default when no match is found.

```
computed-attributes:
  - name: internal_match_pattern
    function: extractValueFromRegex
    fields:
      - EmployeeType
      - '"(00)"'
      - 1
  - name: internal
    function: mapValue
    fields:
      - internal_match_pattern
      - internal_map # Meta attribute "00,true" split on "," into ["00","true"]
  - name: internal
    function: concat
    compute-strategy: IF_EMPTY
    fields:
      - attribute-name: internal_default # Meta attribute literal "false"
        auxiliary-attribute-names: []
```

| `EmployeeType` | `internal_match_pattern` | `internal` output |
| --- | --- | --- |
| `100` | `00` | `true` |
| `1` | null | `false` |

## Function signatures

The `functions_v1` template lists the function signatures for configuration version `v1`. These signatures are used to validate the types of attributes passed to each function.

When an attribute's type is specified in the source object, the graph pipeline engine uses it to cast the input attribute to the expected type.

```
version: v1

functions:
  - name: tagNonHumanAccountWithRuleName
    method: tagNonHumanAccountWithRuleName
    namespace: recon
    input-types:
      - type: STRING
      - type: STRING
    output-type:
      type: STRING
  - name: getUserAccountControlAttribute
    method: getUserAccountControlAttribute
    namespace: ldap
    input-types:
      - type: INTEGER
      - type: STRING
    output-type:
      type: BOOLEAN
  - name: getMsDsUserAccountControlComputedAttribute
    method: getMsDsUserAccountControlComputedAttribute
    namespace: ldap
    input-types:
      - type: INTEGER
      - type: STRING
    output-type:
      type: BOOLEAN
  - name: inferPrivilegedAccountFromUac
    method: inferPrivilegedAccountFromUac
    namespace: ldap
    input-types:
      - type: INTEGER
    output-type:
      type: BOOLEAN
  - name: getUserAccountControlAttributes
    method: getUserAccountControlAttributes
    namespace: ldap
    input-types:
      - type: INTEGER
      - type: BOOLEAN
      - type: STRING_ARRAY
    output-type:
      type: BOOLEAN
  - name: convertBase64ObjectGuidToUuid
    method: convertBase64ObjectGuidToUuid
    namespace: ldap
    input-types:
      - type: STRING
    output-type:
      type: STRING
  - name: convertBase64GuidToHex
    method: convertBase64GuidToHex
    namespace: ldap
    input-types:
      - type: STRING
    output-type:
      type: STRING
  - name: normalizeDn
    method: normalizeDn
    namespace: ldap
    input-types:
      - type: STRING
    output-type:
      type: STRING
  - name: normalizeDns
    method: normalizeDns
    namespace: ldap
    input-types:
      - type: STRING
        list: true
    output-type:
      type: STRING
      list: true
  - name: filterDescendentOf
    method: filterDescendentOf
    namespace: ldap
    input-types:
      - type: STRING
      - type: STRING
    output-type:
      type: STRING
  - name: filterDescendentsOf
    method: filterDescendentsOf
    namespace: ldap
    input-types:
      - type: STRING
        list: true
      - type: STRING
    output-type:
      type: STRING
      list: true
  - name: getParent
    method: getParent
    namespace: ldap
    input-types:
      - type: STRING
    output-type:
      type: STRING
  - name: getRdnValue
    method: getRdnValue
    namespace: ldap
    input-types:
      - type: STRING
    output-type:
      type: STRING

  - name: getDateTimeNow
    method: now
    namespace: date
    output-type:
      type: STRING
  - name: convertTimestamp
    method: convertTimestamp
    namespace: date
    input-types:
      - type: LONG
    output-type:
      type: STRING
  - name: convertDate
    method: convertDate
    namespace: date
    input-types:
      - type: STRING
      - type: STRING
    output-type:
      type: STRING
  - name: convertLDAPGeneralizedDateTime
    method: convertDate
    namespace: date
    input-types:
      - type: STRING
    output-type:
      type: STRING

  - name: isBlank
    method: isBlank
    namespace: string
    input-types:
      - type: STRING
    output-type:
      type: BOOLEAN
  - name: normalizeValue
    method: normalizeValue
    namespace: string
    input-types:
      - type: STRING
    output-type:
      type: STRING
  - name: normalizeValues
    method: normalizeValues
    namespace: string
    input-types:
      - type: STRING
        list: true
    output-type:
      type: STRING
      list: true
  - name: filterValue
    method: filterValue
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
      - type: STRING
    output-type:
      type: STRING
  - name: filterValues
    method: filterValues
    namespace: string
    input-types:
      - type: STRING
        list: true
      - type: STRING
      - type: STRING
      - type: BOOLEAN
    output-type:
      type: STRING
      list: true
  - name: filterDuplicates
    method: filterDuplicates
    namespace: string
    input-types:
      - type: STRING
        list: true
    output-type:
      type: STRING
      list: true
  - name: concatWithSeparator
    method: concatWithSeparator
    namespace: string
    input-types:
      - type: STRING
      - type: STRING_ARRAY
    output-type:
      type: STRING
  - name: concat
    method: concat
    namespace: string
    input-types:
      - type: STRING_ARRAY
    output-type:
      type: STRING
  - name: mergeRemovingDuplicates
    method: mergeRemovingDuplicates
    namespace: string
    input-types:
      - type: STRING
        list: true
      - type: STRING
        list: true
    output-type:
      type: STRING
      list: true
  - name: extractValueFromRegex
    method: extractValueFromRegex
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
      - type: INTEGER
    output-type:
      type: STRING
  - name: extractValueWithEscapeChar
    method: extractValue
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
      - type: INTEGER
      - type: CHAR
    output-type:
      type: STRING
  - name: extractValue
    method: extractValue
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
      - type: INTEGER
    output-type:
      type: STRING
  - name: extractValuesWithEscapeChar
    method: extractValues
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
      - type: INTEGER_ARRAY
      - type: CHAR
    output-type:
      type: STRING
      list: true
  - name: extractValues
    method: extractValues
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
      - type: INTEGER_ARRAY
    output-type:
      type: STRING
      list: true
  - name: ordinals
    method: ordinals
    namespace: string
    input-types:
      - type: STRING
        list: true
      - type: STRING
        list: true
    output-type:
      type: INTEGER_ARRAY
  - name: mapValue
    method: mapValue
    namespace: string
    input-types:
      - type: STRING
      - type: STRING
        list: true
    output-type:
      type: STRING
```
