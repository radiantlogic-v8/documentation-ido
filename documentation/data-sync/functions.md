---
title: Included Functions
description: Functions available to graph pipelines as computed attributes
---

# Pipelines Functions

Computed attributes add values to a source object that are derived at ingestion time rather than read as-is from the source. They are declared per source object in the pipeline configuration and evaluated by the graph pipeline engine as each change event flows through the pipeline.

This page describes how computed attributes are configured, the behavioral settings that control how they are computed, and the functions available to them.

## Two computation engines

A computed attribute is produced by one of 2 engines, selected by the function it references:

- **Precanned functions**: Specialized functions organized by namespace (`date`, `string`, `ldap`, `recon`), each declared by name in the functions template. See [Function catalog](#function-catalog).
- **Macros engine**: The `evaluateExpression` function evaluates a JavaScript-like expression enclosed in curly braces `{}`.

```
computed-attributes:
  # precanned function
  - name: disabled
    function: getUserAccountControlAttribute
    fields:
      - useraccountcontrol
      - uacDisabled
  # macros engine
  - name: active
    function: evaluateExpression
    fields:
      - '"{!disabled}"'
```

## Configure a computed attribute

Each entry under the `computed-attributes` section of a source object supports the following keys:

| **Key** | **Required** | **Default** | **Description** |
|---------|--------------|-------------|-----------------|
| `name` | Yes | | The name of the attribute produced. |
| `function` | Yes | | The function to call: a function from the [function catalog](#function-catalog) or `evaluateExpression`. |
| `fields` | When the function takes inputs | | The ordered inputs. See [Pass input fields](#pass-input-fields). The active fields must match the function's input types one by one. `evaluateExpression` takes exactly one active field: a quoted macro string, or the name of a meta attribute whose literal is the expression. |
| `compute-strategy` | No | `OVERIDE` | How the computed value is reconciled with any existing value. See [Compute strategy](#compute-strategy). |
| `compute-phase` | No | Both phases | Restricts computation to the `INIT` or `EVENT_DRIVEN` phase. See [Compute phase](#compute-phase). |
| `event-types` | No | All | Restricts computation to specific event types (`INSERT`, `UPDATE`, `DELETE`). See [Event types](#event-types). |
| `cardinality` | No | `HIGH` | Used when the computed attribute is a `vertex-uid`. See [Advanced behavioral settings](./pipeline-configuration.md#advanced-behavioral-settings). |

> Computed attributes are evaluated in the order they are declared. A computed attribute can use the output of an attribute declared earlier in the list. Several entries can share the same `name`: each one computes the same attribute in turn, which is how `IF_EMPTY` fallbacks are built.

### Pass input fields

Each `fields` entry is one of the following:

- **An attribute name**: A backend attribute, a meta attribute, or a computed attribute of the same source object.
- **An inline literal**: A quoted string (`'"(00)"'`), an integer (`1`), a boolean (`true`, `false`), a match operator name (`NOT_EQUALS`), an Active Directory `userAccountControl` flag name (`ACCOUNTDISABLE`), or the name of a non-human account reconciliation rule.
- **An object** with `attribute-name` and the following optional keys:
  - `active` (default `true`): Set to `false` to use the attribute only as a dependency that triggers the computation. See [Dependency management](#dependency-management).
  - `auxiliary-attribute-names`: Additional attributes whose values are appended to the values of `attribute-name`, so the field is passed as one list argument. All of them must have the same type as `attribute-name`, and the field then takes the list form of that type (`STRING` becomes `STRING_ARRAY`). This is how a list parameter, such as the one of `concat`, receives several attributes. To pass a single attribute to such a parameter, use an empty list: `auxiliary-attribute-names: []`.

```
fields:
  - name_separator # meta attribute
  - attribute-name: FirstName # FirstName and LastName values …
    auxiliary-attribute-names: [LastName] # … form one list argument
```

When a backend attribute is passed to a function that expects a type other than `STRING`, declare its `type` (for example, `LONG` or `INTEGER`) in the source object so that the engine can cast it.

The configuration is checked when it is applied. An unknown function, a field that names an unknown attribute, a wrong number of active fields, or an active field whose type does not match the function's input type at that position is rejected.

### How computed attributes are evaluated

For every change event, the engine first writes the meta attributes onto the event. Then, for each computed attribute in declaration order, it:

1. Skips the attribute if the event's phase or type does not match.
2. Skips the attribute if the compute strategy says so.
3. Calls the function with the attribute's active fields.
4. Writes the result back according to the compute strategy.

An `UPDATE` event carries both the changes made to the entry and the full entry. The engine computes against both, so the changes it forwards also include the computed attribute's new values. A missing input value for a parameter that is not a string makes the function return an empty result.

> If a function throws an error (for example, bad input, an unparseable date, or an invalid argument), the whole change event is rejected and the pipeline moves on to the next event. Attach a function only to inputs that are reliably populated and well formed. A macro that calls an unknown function, method, or property, passes the wrong number of arguments, or uses a constructor is rejected earlier, when the configuration is applied or when a template is saved.

## Behavioral settings

The behavioral settings are small indicators in the configuration that control how the engine computes each computed attribute.

### Dependency management

Each `fields` entry declares an input that the computed attribute depends on. The `active` flag decides how that dependency is used:

- `active: true` (default): The field is a real input, passed as an argument to the function.
- `active: false`: The field is only a dependency. When an incoming change modifies that attribute, it triggers the computation, but its value is not passed to the function.

The auxiliary attributes of a field are dependencies too, with the field's `active` flag. For `evaluateExpression`, the dependencies are the variables the macro reads, not its `fields`.

Use `active: false` to record when something happened: the function takes no real input, and another attribute is used only to trigger it.

```
computed-attributes:
  - name: last_insertion_date
    function: getDateTimeNow # takes no real input
    compute-phase: EVENT_DRIVEN
    event-types: [INSERT]
    fields:
      - attribute-name: member # a group membership change …
        active: false # … only triggers the timestamp
  - name: last_deletion_date
    function: getDateTimeNow
    compute-phase: EVENT_DRIVEN
    event-types: [DELETE]
    fields:
      - attribute-name: member
        active: false
```

Here, `event-types` is matched against the type of change made to the trigger field: a value added to `member` is an `INSERT`, and a value removed is a `DELETE`.

**Dependency ordering**

When a computed attribute depends on another computed attribute, the engine evaluates the list from top to bottom, so a later entry uses the values produced by earlier entries. A dependency on a computed attribute is only resolved if that attribute is declared earlier in the list. A field that names a computed attribute declared later reads that attribute's value as the event carries it.

```
meta-attributes:
  - name: permission_separator
    literal: "-"
computed-attributes:
  - name: permissionId # produced first …
    function: normalizeValue
    fields:
      - permissionName
  - name: permissionId # … used here
    function: concatWithSeparator
    compute-strategy: IF_NOT_EMPTY
    fields:
      - permission_separator
      - attribute-name: permissionId
        auxiliary-attribute-names: [cn]
```

### Compute strategy

`compute-strategy` controls whether the attribute is computed, and how the result is reconciled with any existing value.

| **Strategy** | **Computes when** | **Write behavior** | **Empty result** |
|--------------|-------------------|--------------------|------------------|
| `OVERIDE` (default) | Always | Replaces the existing value with the function result. | Clears the attribute. |
| `MERGE` | Always | Combines the computed values with the existing values, without duplicates. | Nothing is merged; the existing values stay. |
| `IF_EMPTY` | The attribute is absent or empty | Writes only when the attribute is absent or empty. | Nothing is written; the attribute stays absent. |
| `IF_NOT_EMPTY` | The attribute already has a value | Recomputes and overwrites only when a value already exists. | Clears the attribute. |

An empty result is a function result that is null, an empty string, or a list holding only empty strings. Empty strings are never written as values, but a string that contains only whitespace is. Because a cleared attribute is absent, a later `IF_EMPTY` definition of the same attribute can fill it.

### Compute phase

A graph pipeline runs in 2 stages: an init stage (bulk load of the current state) and an event-driven stage (real-time change events from the connectors). `compute-phase` restricts when a computed attribute is computed:

| **compute-phase** | **Runs during** |
|-------------------|-----------------|
| (omitted) | Both phases |
| `INIT` | Only the initial bulk load |
| `EVENT_DRIVEN` | Only real-time events, after the initial load completes |

`compute-phase` is often combined with `event-types`. For example, a `last_deletion_date` declared with `compute-phase: EVENT_DRIVEN` and `event-types: [DELETE]` stays empty during the bulk load and is only populated by real-time delete events.

### Event types

`event-types` is matched against the event type determined by the pipeline, which can differ from the type the connector sent:

- A `MOVE` (rename) is an `UPDATE`.
- An `UPDATE` of an entry that the pipeline has never seen is an `INSERT`.
- A `DELETE` on a source object linked to the identity vertex is an `UPDATE`: the identity is deactivated rather than deleted, so `event-types: [DELETE]` never applies to such a source object.

When a change modifies a trigger field (`active: false`) or the computed attribute itself, `event-types` is matched against the type of change instead: an added value is an `INSERT`, a replaced value an `UPDATE`, and a removed value a `DELETE`. A change on an active field is matched as the event's own type, `UPDATE`. For `evaluateExpression`, every variable the macro reads is matched like a trigger field.

## Function catalog

The following tables summarize each function's behavior with an example. For the input and output types used to validate the configuration, see [Functions signatures](#functions-signatures), or fetch the current list from the orchestrator endpoint `GET /identitygraph/templates/functions_v1`.

Functions operate on one record at a time. Most return null for a null or empty input rather than throwing an error. The exceptions are noted below. A function that throws an error causes the whole change event to be rejected.

### Date functions

Date functions return dates in the Identity Observability graph format: `yyyyMMddHHmmss` in UTC (14 digits, no fraction, no time zone).

| **Function** | **Example (input → output)** | **Behavior** |
|--------------|------------------------------|--------------|
| `getDateTimeNow` | (no input) → `20260623203000` | Current UTC time. Takes no `fields`. |
| `convertTimestamp` | `1659302400000` → `20220731212000` | Converts epoch milliseconds (a `LONG` input). Declare the backend attribute with `type: LONG`. |
| `convertDate` | `"29/Mar/2023:15:30:00 +0200"`, `"dd/MMM/yyyy:HH:mm:ss Z"` → `20230329133000` | Parses the value with the pattern and converts it to UTC. A zone token (`X`, `x`, `Z`, `z`, `O`, `V`) applies the zone. A date-only pattern without a zone token returns `000000` for the time; a date-only pattern with a zone token throws an error. Returns null if the value or the pattern is null. An empty value, a value that doesn't match the pattern, or an invalid pattern throws an error. |
| `convertLDAPGeneralizedDateTime` | `"20230329153000.500Z"` → `20230329153000` | Parses an LDAP generalized time, removes fractional seconds, and assumes UTC if no zone is given. Null returns null. An empty value or an unsupported layout throws an error. |

> **convertDate patterns:** Use a real offset token, not a quoted literal. For example, use `yyyy-MM-dd'T'HH:mm:ss[.SSSSSSSSS]X`, not `…'Z'`: a quoted `'Z'` reads every value as UTC and ignores its real offset. The number of `S` fraction digits is fixed, so variable-width fractions don't parse with a literal pattern. Only use `convertDate` on date attributes that are reliably populated: an empty value throws an error and the event is rejected.

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

| **Function** | **Example (input → output)** | **Behavior** |
|--------------|------------------------------|--------------|
| `isBlank` | `"   "` → `true` | `true` if the value is null, empty, or only whitespace. |
| `normalizeValue` | `" Hello "` → `hello` | Trims and lowercases the value. Null returns null; `""` returns `""`. |
| `normalizeValues` | `[" Hello ", "WORLD"]` → `[hello, world]` | Trims and lowercases each element. Null returns null; fails if any element is null. |
| `filterValue` | `"test"`, `"test"`, `EQUALS` → `test` | Returns the value if it matches the operator, otherwise null. A null or empty value returns null. An unknown operator (names are case-sensitive), an invalid regular expression, or a null match string with a regular expression operator throws an error. |
| `filterValues` | `["apple","banana","cherry-a"]`, `"banana"`, `EQUALS`, `false` → `[banana]` | Returns the matching subset, with optional removal of duplicates (last argument). A null or empty list, or an empty result, returns null. |
| `filterDuplicates` | `["a","b","a"]` → `[a, b]` | Removes duplicates (case-sensitive). Null or empty returns null. |
| `concatWithSeparator` | `"."`, `["john","doe"]` → `john.doe` | Joins the values with the separator. A null separator behaves as `""`. |
| `concat` | `["john","doe"]` → `johndoe` | Concatenates the values with no separator, and skips a missing value. Use `concatWithSeparator` when 2 different combinations could produce the same text. |
| `mergeRemovingDuplicates` | `["Apple","Banana"]`, `["Banana","Cherry","Apple"]` → `Apple`, `Banana`, `Cherry` | Merges 2 lists and removes duplicates (case-sensitive). The order is not guaranteed. |
| `extractValueFromRegex` | `"BW-Main-SharePointHR-Picture"`, `"^BW-Main[^-]*-([^,]*?)-(.*)$"`, `1` → `SharePointHR` | Returns the trimmed value of the capture group at the given index, or null if there is no match. A group index beyond the regular expression's groups, or an invalid regular expression, throws an error. |
| `extractValueWithEscapeChar` | `"part1\;escaped;part2;part3"`, `";"`, `1`, `\` → `part2` | Returns the part at the index, using a literal separator and an escape character. Out of range returns null. |
| `extractValue` | `"part1;part2;part3"`, `";"`, `1` → `part2` | Same as above, but the separator is a regular expression: escape metacharacters such as `.`. Trailing empty parts are dropped. |
| `extractValuesWithEscapeChar` | `"part1\;escaped;part2;part3"`, `";"`, `[0,2]`, `\` → `[part1;escaped, part3]` | Returns the parts at the indices. Stops at the first index out of range; an empty result returns null. |
| `extractValues` | `"part1;part2;part3"`, `";"`, `[0,2]` → `[part1, part3]` | Same as above, with a regular expression separator. |
| `jsonExtract` | `{"source": {"name":"Active Directory"}}`, `"source.name"` → `Active Directory` | Returns the value at the dot path. Values that are not text are converted to strings. Invalid JSON, a missing path, or a JSON null returns null. |
| `ordinals` | `["b","a"]`, `["a","b","c"]` → `[1, 0]` | Returns the index of each value within the reference list. Not found returns `-1`. |
| `mapValue` | `"key1"`, `["key1","val1","key2","val2"]` → `val1` | Returns the value paired with the first key equal to the input in a flat key/value list. Only keys are matched, case-sensitively. No match returns null. When every value of the list comes from meta attributes, the configuration is rejected if the list is empty or has an odd number of values. |

The match operators of `filterValue` and `filterValues` are: `EQUALS`, `NOT_EQUALS`, `CONTAINS`, `NOT_CONTAINS`, `STARTS_WITH`, `NOT_STARTS_WITH`, `ENDS_WITH`, `NOT_ENDS_WITH`, `IS_BLANK`, `IS_NOT_BLANK`, `MATCHES_REGEX` (matches when the regular expression is found anywhere in the value) and `NOT_MATCHES_REGEX`.

Use `normalizeValue` on any attribute used as a vertex UID or in an edge link condition, so that values are compared with consistent casing. Use `concat` with `compute-strategy: IF_EMPTY` to build a fallback value only when the attribute is empty:

```
meta-attributes:
  - name: emailDomain
    literal: "@acme.com"
computed-attributes:
  - name: mail
    function: normalizeValue
    fields:
      - mail
  - name: mail # if mail is empty, build it from sAMAccountName and the email domain
    function: concat
    compute-strategy: IF_EMPTY
    fields:
      - attribute-name: sAMAccountName
        auxiliary-attribute-names: [emailDomain]
```

Use `extractValueFromRegex` to extract part of a value. For a group named `APP-ActiveDirectory-Read`, the following example returns `ActiveDirectory` as the resource name and `Read` as the permission name:

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

| **Function** | **Example (input → output)** | **Behavior** |
|--------------|------------------------------|--------------|
| `getUserAccountControlAttribute` | `528`, `"LOCKOUT"` → `true` | `true` if the named Active Directory `userAccountControl` flag is set. Flag names are case-insensitive; an unknown flag name throws an error. |
| `getMsDsUserAccountControlComputedAttribute` | `8388608`, `"UF_PASSWORD_EXPIRED"` → `true` | `true` if the named `ms-DS-User-Account-Control-Computed` flag is set. Flag names are case-insensitive; an unknown flag name throws an error. |
| `inferPrivilegedAccountFromUac` | `65536` → `true` | `true` if any of `PASSWD_NOTREQD`, `DONT_EXPIRE_PASSWORD` or `TRUSTED_FOR_DELEGATION` is set. |
| `getUserAccountControlAttributes` | `528`, `false`, `["LOCKOUT"]` → `true` | Combines the named flags: AND when the boolean is `true`, OR when it is `false`. With no flag names, AND returns `true` and OR returns `false`. |
| `convertBase64ObjectGuidToUuid` | `"{base64binary}AAECAwQFBgcICQoLDA0ODw=="` → `00010203-0405-0607-0809-0a0b0c0d0e0f` | Converts an Active Directory `objectGUID` to a UUID. Accepts a `{base64binary}`-marked value, a 32-character hexadecimal string, or a Base64 value. A null or empty value, invalid Base64, or a value that is not 16 bytes throws an error. |
| `convertBase64GuidToHex` | `"{base64binary}AAECAwQFBgcICQoLDA0ODw=="` → `000102030405060708090a0b0c0d0e0f` | Converts a GUID to lowercase hexadecimal. Accepts a `{base64binary}`-marked value or a Base64 value, not a hexadecimal string. |
| `normalizeDn` | `"cn=John Doe,ou=Users,dc=example,dc=com"` → `cn=john doe,ou=users,dc=example,dc=com` | Returns the canonical normalized DN. Null returns null; an invalid DN throws an error. |
| `normalizeDns` | `["cn=John Doe,ou=Users,dc=example,dc=com"]` → `[cn=john doe,ou=users,dc=example,dc=com]` | Normalizes each DN. Null returns null; a null or invalid DN in the list throws an error. |
| `filterDescendentOf` | `"cn=John Doe,ou=Users,dc=example,dc=com"`, `"ou=Users,dc=example,dc=com"` → `cn=john doe,ou=users,dc=example,dc=com` | Returns the normalized DN if it is a descendant of (or equal to) the base DN, otherwise null. |
| `filterDescendentsOf` | `["cn=John Doe,ou=Users,…", "cn=Admin,ou=Admins,…"]`, `"ou=Users,dc=example,dc=com"` → `[cn=john doe,ou=users,…]` | Returns the normalized descendant DNs, or null if none match. |
| `getParent` | `"cn=John Doe,ou=Users,dc=example,dc=com"` → `ou=Users,dc=example,dc=com` | Returns the parent DN as written in the input (not normalized), or null if there is none. |
| `getRdnValue` | `"cn=John Doe,ou=Users,dc=example,dc=com"` → `John Doe` | Returns the first attribute value of the leftmost RDN. |

Use `normalizeDns` on multi-valued DN attributes, such as the `member` attribute of a group, when they are used in an edge link condition. Use `getUserAccountControlAttribute` with flag names passed as meta attributes, such as `ACCOUNTDISABLE`, `LOCKOUT`, `PASSWORD_EXPIRED`, `PASSWD_NOTREQD`, `PASSWD_CANT_CHANGE`, `DONT_EXPIRE_PASSWORD`, and `SMARTCARD_REQUIRED`.

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

| **Function** | **Example (input → output)** | **Behavior** |
|--------------|------------------------------|--------------|
| `tagNonHumanAccountWithRuleName` | `"svc-account-rule"`, `"CN=svc01"` → `svc-account-rule` | Returns the rule name if the second value is present (not null and not empty), otherwise null. Used by the reconciliation policy to tag non-human accounts. |

## Examples

The following examples show the behavioral settings in action.

**IF_EMPTY keeps an existing value, or fills it from a dependency**

```
computed-attributes:
  # build "First, Last"
  - name: js_fullname
    function: concatWithSeparator
    fields:
      - name_separator # meta attribute literal ", "
      - attribute-name: FirstName
        auxiliary-attribute-names: [LastName]
  # use the computed full name only when DisplayName is absent
  - name: DisplayName
    function: concat
    compute-strategy: IF_EMPTY
    fields:
      - attribute-name: js_fullname
        auxiliary-attribute-names: []
```

| **Input attributes** | **js_fullname** | **DisplayName (output)** | **Why** |
|----------------------|-----------------|--------------------------|---------|
| `FirstName=Alice`, `LastName=Anderson`, `DisplayName=Alice M Anderson` | `Alice, Anderson` | `Alice M Anderson` | `IF_EMPTY` is skipped because `DisplayName` already has a value. |
| `FirstName=Bob`, `LastName=Brown` (no `DisplayName`) | `Bob, Brown` | `Bob, Brown` | `DisplayName` was empty, so it takes the computed `js_fullname`. |

**filterValue removes a placeholder value**

```
computed-attributes:
  - name: clean_normalized_mail
    function: filterValue
    fields:
      - normalized_mail
      - noemail_sentinel # meta attribute literal "noemail@example.com"
      - NOT_EQUALS
```

| **normalized_mail** | **clean_normalized_mail (output)** |
|---------------------|------------------------------------|
| `alice@example.com` | `alice@example.com` |
| `noemail@example.com` | null (not emitted) |

**Extract, map, and default**

A regular expression extracts a marker, a mapping turns it into a flag, and an `IF_EMPTY` default covers the case where there is no match.

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
      - internal_map # meta attribute "00,true" split on "," into ["00","true"]
  - name: internal
    function: concat
    compute-strategy: IF_EMPTY
    fields:
      - attribute-name: internal_default # meta attribute literal "false"
        auxiliary-attribute-names: []
```

| **EmployeeType** | **internal_match_pattern** | **internal (output)** |
|------------------|----------------------------|-----------------------|
| `100` | `00` | `true` |
| `1` | null | `false` |

## Functions signatures

The functions signatures are listed in the `functions_v1` template for the version of the configuration used (v1). They are used to validate the type of the attributes passed to each function. When a type is specified in the source object, the graph pipeline engine uses it to cast the input attribute to the expected type.

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
