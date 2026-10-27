---
title: Pipeline Configuration Examples
description: A complete graph pipelines configuration example
---

# Pipeline Configuration Examples

The following example is a complete pipeline configuration that loads Active Directory accounts and groups and HR identities and departments into the graph. It builds on the components described in [pipeline configuration](./pipeline-configuration.md):

- The `ad_users` source object derives Active Directory account flags from `userAccountControl`, converts timestamps, and builds a fallback email address.
- The `ad_groups` source object extracts resource and permission names from the group name, records membership change dates, and normalizes the `member` DNs.
- The `hr_people` and `hr_org` source objects convert HR dates with a custom format and normalize the keys used in edges.
- Repository meta-attributes (`repositoryType` and `repositoryCapability`) populate the repository type and capability flags required by the portal.
- The edges link accounts to groups, groups to groups, accounts to identities through the reconciliation engine, identities and departments to their managers, and departments to their parent departments.

```
version: v1
templates:
  - ref: functions_v1

source-objects:
  - name: ad_users
    datasource: acme
    object-name: userX
    backend-attributes:
      - name: cn
      - name: sn
      - name: sAMAccountName
      - name: description
      - name: mail
      - name: whencreated
        type: LONG
      - name: whenmodified
        type: LONG
      - name: useraccountcontrol
        type: INTEGER
      - name: objectguidstr
    meta-attributes:
      - name: repositoryName
        literal: Active Directory
      - name: repositoryDescription
        literal: This is an AD Mock server
      - name: repositoryType
        literal: Accounts
      - name: repositoryCapability
        literal: "false"
        type: BOOLEAN
      - name: emailDomain
        literal: "@acme.com"
      - name: uacLockout
        literal: "LOCKOUT"
      - name: uacPwdExpired
        literal: "PASSWORD_EXPIRED"
      - name: uacDisabled
        literal: "ACCOUNTDISABLE"
      - name: uacPwdNotRequired
        literal: "PASSWD_NOTREQD"
      - name: uacSmartCardRequired
        literal: "SMARTCARD_REQUIRED"
      - name: uacPwdCantChange
        literal: "PASSWD_CANT_CHANGE"
      - name: uacPwdDontExpire
        literal: "DONT_EXPIRE_PASSWORD"
    computed-attributes:
      - name: locked
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacLockout
      - name: disabled
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacDisabled
      - name: password_not_required
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacPwdNotRequired
      - name: password_expired
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacPwdExpired
      - name: password_cant_change
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacPwdCantChange
      - name: dont_expire_password
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacPwdDontExpire
      - name: smart_card_required
        function: getUserAccountControlAttribute
        fields:
          - useraccountcontrol
          - uacSmartCardRequired
      - name: privileged_account
        function: inferPrivilegedAccountFromUac
        fields:
          - useraccountcontrol
      - name: whencreated
        function: convertTimestamp
        fields:
          - whencreated
      - name: whenmodified
        function: convertTimestamp
        fields:
          - whenmodified
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
    object-name: groupX
    backend-attributes:
      - name: cn
      - name: description
      - name: member
      - name: objectguidstr
    meta-attributes:
      - name: repositoryName
        literal: Active Directory
      - name: repositoryDescription
        literal: This is an AD Mock server
      - name: repositoryType
        literal: Accounts
      - name: repositoryCapability
        literal: "false"
        type: BOOLEAN
      - name: regexExtractResourcesPermsFromGroupName
        literal: "^BW-Main[^-]*-([^,]*?)-(.*)$"
      - name: groupResourceIdx
        literal: 1
        type: INTEGER
      - name: groupPermissionIdx
        literal: 2
        type: INTEGER
      - name: permissionType
        literal: Group
      - name: resourceType
        literal: Application
    computed-attributes:
      - name: last_insertion_date
        function: getDateTimeNow
        compute-phase: EVENT_DRIVEN
        event-types: [INSERT]
        fields:
          - attribute-name: member
            active: false
      - name: last_deletion_date
        function: getDateTimeNow
        compute-phase: EVENT_DRIVEN
        event-types: [DELETE]
        fields:
          - attribute-name: member
            active: false
      - name: membershipCreationTime
        function: getDateTimeNow
      - name: resourceName
        cardinality: LOW
        function: extractValueFromRegex
        fields:
          - cn
          - regexExtractResourcesPermsFromGroupName
          - groupResourceIdx
      - name: permissionName
        cardinality: LOW
        function: extractValueFromRegex
        fields:
          - cn
          - regexExtractResourcesPermsFromGroupName
          - groupPermissionIdx
      - name: permissionId
        function: normalizeValue
        fields:
          - attribute-name: permissionName
      - name: permissionId
        function: concat
        compute-strategy: IF_NOT_EMPTY
        fields:
          - attribute-name: permissionId
            auxiliary-attribute-names: [cn]
      - name: resourceName
        function: normalizeValue
        fields:
          - resourceName
      - name: permissionName
        function: normalizeValue
        fields:
          - permissionName
      - name: member
        function: normalizeDns
        fields:
          - member

  - name: hr_people
    datasource: acme
    object-name: inetOrgPersonX
    backend-attributes:
      - name: fullname
      - name: cn
      - name: sn
      - name: mail
      - name: hrcode
      - name: employeeType
      - name: mobile
      - name: arrivaldate
      - name: departuredate
      - name: active
      - name: internal
      - name: organizationcode
      - name: managerhrcode
      - name: jobtitle
      - name: displayname
    meta-attributes:
      - name: repositoryName
        literal: HR Service
      - name: repositoryDescription
        literal: This is the HR Mock server
      - name: repositoryType
        literal: Identities
      - name: repositoryCapability
        literal: "false"
        type: BOOLEAN
      - name: hrDateFormat
        literal: dd/MM/yyyy
      - name: defaultJobTitle
        literal: NO_JOB_TITLE
    computed-attributes:
      - name: arrivaldate
        function: convertDate
        fields:
          - arrivaldate
          - hrDateFormat
      - name: departuredate
        function: convertDate
        fields:
          - departuredate
          - hrDateFormat
      - name: mail
        function: normalizeValue
        fields:
          - mail
      - name: organizationcode
        function: normalizeValue
        fields:
          - organizationcode
      - name: jobtitle
        function: normalizeValue
        compute-strategy: IF_EMPTY
        fields:
          - attribute-name: defaultJobTitle

  - name: hr_org
    datasource: acme
    object-name: org
    backend-attributes:
      - name: orgcode
      - name: displayname
      - name: type
      - name: parentcode
      - name: managerhrcode
    meta-attributes:
      - name: repositoryName
        literal: HR Service
      - name: repositoryDescription
        literal: This is the HR Mock server
    computed-attributes:
      - name: orgcode
        function: normalizeValue
        fields:
          - orgcode
      - name: parentcode
        function: normalizeValue
        fields:
          - parentcode

vertices:
  - name: rt_account
    linkage:
      - source-object-name: ad_users
        vertex-uid: objectguidstr
        properties:
          - name: name
            attribute-name: cn
          - name: full_name
            attribute-name: cn
          - name: displayname
            attribute-name: cn
          - name: surname
            attribute-name: sn
          - name: samaccountname
            attribute-name: sAMAccountName
          - name: identifier
            attribute-name: sAMAccountName
          - name: description
            attribute-name: description
          - name: email
            attribute-name: mail
          - name: last_modification_date
            attribute-name: whenmodified
          - name: disabled
            attribute-name: disabled
          - name: locked
            attribute-name: locked
          - name: object_guid
            attribute-name: objectguidstr
          - name: password_not_required
            attribute-name: password_not_required
          - name: password_cant_change
            attribute-name: password_cant_change
          - name: privileged_account
            attribute-name: privileged_account
          - name: dont_expire_password
            attribute-name: dont_expire_password
          - name: password_expired
            attribute-name: password_expired
          - name: smart_card_required
            attribute-name: smart_card_required

  - name: rt_group
    linkage:
      - source-object-name: ad_groups
        vertex-uid: objectguidstr
        properties:
          - name: name
            attribute-name: cn
          - name: displayname
            attribute-name: cn
          - name: description
            attribute-name: description
          - name: last_insertion_date
            attribute-name: last_insertion_date
          - name: last_deletion_date
            attribute-name: last_deletion_date

  - name: rt_identity
    linkage:
      - source-object-name: hr_people
        vertex-uid: hrcode
        properties:
          - name: external_identifier
            attribute-name: hrcode
          - name: displayname
            attribute-name: cn
          - name: surname
            attribute-name: sn
          - name: email
            attribute-name: mail
          - name: employee_number
            attribute-name: hrcode
          - name: employee_type
            attribute-name: employeeType
          - name: mobile
            attribute-name: mobile
          - name: arrival_date
            attribute-name: arrivaldate
          - name: departure_date
            attribute-name: departuredate
          - name: internal
            attribute-name: internal
          - name: active
            attribute-name: active

  - name: rt_repository
    linkage:
      - source-object-name: hr_people
        vertex-uid: repositoryName
        properties:
          - name: name
            attribute-name: repositoryName
          - name: displayname
            attribute-name: repositoryName
          - name: description
            attribute-name: repositoryDescription
          - name: type
            attribute-name: repositoryType
          - name: is_mfa_capable
            attribute-name: repositoryCapability
          - name: is_passwordless_capable
            attribute-name: repositoryCapability
          - name: is_sspr_capable
            attribute-name: repositoryCapability
      - source-object-name: ad_users
        vertex-uid: repositoryName
        properties:
          - name: name
            attribute-name: repositoryName
          - name: displayname
            attribute-name: repositoryName
          - name: description
            attribute-name: repositoryDescription
          - name: type
            attribute-name: repositoryType
          - name: is_mfa_capable
            attribute-name: repositoryCapability
          - name: is_passwordless_capable
            attribute-name: repositoryCapability
          - name: is_sspr_capable
            attribute-name: repositoryCapability
      - source-object-name: ad_groups
        vertex-uid: repositoryName
        properties:
          - name: name
            attribute-name: repositoryName
          - name: displayname
            attribute-name: repositoryName
          - name: description
            attribute-name: repositoryDescription
          - name: type
            attribute-name: repositoryType
          - name: is_mfa_capable
            attribute-name: repositoryCapability
          - name: is_passwordless_capable
            attribute-name: repositoryCapability
          - name: is_sspr_capable
            attribute-name: repositoryCapability

  - name: rt_department
    linkage:
      - source-object-name: hr_org
        vertex-uid: orgcode
        properties:
          - name: external_identifier
            attribute-name: orgcode
          - name: name
            attribute-name: orgcode
          - name: displayname
            attribute-name: displayname
          - name: type
            attribute-name: type
          - name: description
            attribute-name: type

  - name: rt_permission
    linkage:
      - source-object-name: ad_groups
        vertex-uid: permissionId
        properties:
          - name: name
            attribute-name: permissionName
          - name: displayname
            attribute-name: permissionName
          - name: type
            attribute-name: permissionType

  - name: rt_resource
    linkage:
      - source-object-name: ad_groups
        vertex-uid: resourceName
        properties:
          - name: name
            attribute-name: resourceName
          - name: displayname
            attribute-name: resourceName
          - name: type
            attribute-name: resourceType

edges:
  - name: is_in_group
    source-vertex-name: rt_account
    target-vertex-name: rt_group
    rules:
      - name: membership
        event-sources: [ad_groups]
        event-types: [INSERT, UPDATE]
        properties:
          - name: created_at
            attribute-name: membershipCreationTime
        link-condition:
          predicates:
            - field: dn
              operator: EQUALS
              value: member

  - name: is_in_group
    source-vertex-name: rt_group
    target-vertex-name: rt_group
    rules:
      - name: recursive_membership
        event-sources: [ad_groups]
        event-types: [INSERT, UPDATE]
        properties:
          - name: created_at
            attribute-name: membershipCreationTime
        link-condition:
          predicates:
            - field: dn
              operator: EQUALS
              value: member

  - name: is_owned_by
    source-vertex-name: rt_account
    target-vertex-name: rt_identity
    recon-policy:
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

  - name: is_in_repository
    source-vertex-name: rt_account
    target-vertex-name: rt_repository

  - name: is_in_repository
    source-vertex-name: rt_identity
    target-vertex-name: rt_repository

  - name: is_in_repository
    source-vertex-name: rt_group
    target-vertex-name: rt_repository

  - name: is_in_repository
    source-vertex-name: rt_resource
    target-vertex-name: rt_repository

  - name: belongs_to
    source-vertex-name: rt_permission
    target-vertex-name: rt_resource

  - name: grants_right_to
    source-vertex-name: rt_permission
    target-vertex-name: rt_group

  - name: is_working_for
    source-vertex-name: rt_identity
    target-vertex-name: rt_department
    rules:
      - name: identity2department_lookup
        event-sources: [hr_people]
        event-types: [INSERT, UPDATE]
        properties:
          - name: job_title
            attribute-name: jobtitle
        link-condition:
          predicates:
            - field: external_identifier
              operator: EQUALS
              value: organizationcode

  - name: is_managed_by
    source-vertex-name: rt_identity
    target-vertex-name: rt_identity
    rules:
      - name: identity2manager_lookup
        event-sources: [hr_people]
        event-types: [INSERT, UPDATE]
        link-condition:
          predicates:
            - field: external_identifier
              operator: EQUALS
              value: managerhrcode

  - name: is_managed_by
    source-vertex-name: rt_department
    target-vertex-name: rt_identity
    rules:
      - name: department2manager_lookup
        event-sources: [hr_org]
        event-types: [INSERT, UPDATE]
        link-condition:
          predicates:
            - field: external_identifier
              operator: EQUALS
              value: managerhrcode

  - name: is_sub_department_of
    source-vertex-name: rt_department
    target-vertex-name: rt_department
    rules:
      - name: dept2dept_lookup
        event-sources: [hr_org]
        event-types: [INSERT, UPDATE]
        link-condition:
          predicates:
            - field: external_identifier
              operator: EQUALS
              value: parentcode
```
