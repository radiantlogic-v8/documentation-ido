# Pipelines Functions

The functions section defines the signatures of the functions that can be applied to the source objects as computed attributes. The signatures are used to validate the type of the attributes used as input fields to the function. When a type is specified in the source object, the graph pipeline engine will use it to cast the input attribute to the expected type.

Methods are organized by namespace, ldap, date and string which is reflected in the java packages structure.
For the purpose of validation at configuration time, their signatures are listed in the functions template for the version of the configuration used (v1 as of now).
For an up-to-date list of functions and their signatures, you can fetch it from the orchestrator endpoint:

## Functions signatures

The functions signatures are defined as follows:

```yaml
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
