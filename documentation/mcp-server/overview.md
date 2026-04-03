---
title: Overview
description: Overview of Identity Observability MCP Server
---

# Identity Observability MCP Server

The Identity Observability MCP server lets you connect AI agents to your identity observability data and access context in a simple, predictable way so they can look up information about identities, accounts, understand their risk, and safely automate remediation steps.

The MCP service implementation exposes four core tools that provide agents with identity and contextual information.

## Available Services

- **Identity Unique Identifier Lookup:** Locate identities in the identity platform.
- **Identity Context Retrieval:** Retrieve full identity context.
- **Access Account Unique Identifier Lookup:** Locate accounts in specific repositories.
- **Access Account Context Retrieval:** Retrieve complete access account context.

These services allow agents to:

- Locate identities in the identity platform
- Locate accounts in specific repositories
- Retrieve complete access account context

Each service returns data in JSON format, making it easy for automation platforms, agents, or other systems to process the results.

## Identity Unique Identifier Lookup

Use this service when you need to find the unique identifier of an identity.

It searches for a person/identity using information such as their name, employee ID, or email address and returns their unique identity identifier. This identifier is required for retrieving the person's full identity context in later queries.

### Input Parameters

You can search using one or more of the following:

- HR Employee ID (matricule RH)
- Full Name (the person's complete name)
- Corporate Email Address

These parameters are evaluated using a logical OR. This means the service will return any identity that matches any of the provided values.

### Possible Results

#### No Result

No identity matches the search criteria.

This usually means:

- the person does not exist in the system, or
- the provided information is incorrect.

#### One Result

Exactly one identity matches the search.

The response includes key details about the identity and the unique identity identifier, which can be used in the Identity Context Retrieval service.

#### Multiple Results

More than one identity matches the search criteria.

When this happens, the response includes identifying details (such as HR ID, name, and email) so the requester can determine which identity is correct.

### Response Data (per identity)

Each identity record includes:

- **Unique Identity Identifier** – the internal primary key for the identity
- **HR Employee ID** – the employee's HR identifier
- **Full Name** – the person's recorded name
- **Email Address** – the primary corporate email

### JSON Response Example

```
{
  "result_count": 1,
  "results": [
    {
      "account_id": "CLAUDIA_NEAL_1730380876537_3683",
      "email": null,
      "employee_number": null,
      "full_name": "ACME.COM:claudia.neal@acme.com",
      "login": "claudia.neal@acme.com",
      "repository": "ACME.COM"
    }
  ],
  "status": "success"
}
```

## Identity Context Retrieval

Use this service when you already know the Identity ID and want to retrieve complete information about that person.

This includes information about:

- their manager
- their organizational assignments
- their associated accounts
- their risk profile
- any issues detected for the identity or its accounts

This information helps with analysis, investigations, and operational decisions.

### Input Parameters

Required parameter:

- **Unique Identity Identifier**
  (returned by the Identity Lookup service)

### Response Data: Complete Identity Context

The response provides structured information about the identity in several categories.

#### Core Identity Attributes

All attributes stored for the identity within the platform.

These may include HR attributes and other identity metadata maintained by the system.

#### Direct Manager Information

Information about the identity's direct manager, including:

- Manager HR Employee ID
- Manager Full Name
- Manager Email Address

#### Risk Profile

A risk evaluation associated with the identity.

This includes:

- **Risk Level** – Low, Medium, High, or Critical
- **Risk Score** – a numeric score representing the risk level

#### Organizational Attachment Information

Details about where the identity sits within the organization.

For each assignment, the response includes:

- Department or business function name

If available, it also includes the department manager's information:

- Manager HR Employee ID
- Manager Full Name
- Manager Email Address

#### Associated Access Accounts

A list of accounts that belong to this identity.

For each account:

- Repository or system name (for example: Active Directory, SAP, Salesforce)
- Account login identifier
- Account status (Active or Inactive)
- Last successful login date and time

#### Identity Issues and Detected Problems

Any issues detected for the identity or its associated accounts.

Each issue includes:

**Attachment Details**

Indicates whether the issue is associated with:

- the identity itself, or
- a specific account

If attached to an account, the response includes:

- Account login
- Account repository name

**Control Information**

Information about the control that detected the issue.

- Control name
- Control description

**Risk Details**

Details about the risk identified.

- Risk description
- Risk level

**Remediation Guidance**

Information about how the issue can be resolved.

- Recommended remediation action
- Current resolution status
- Issue detection date

### JSON Response Example

```
{
  "identity": {
    "identity_id": "unique_identifier",
    "hr_employee_id": "EMP12345",
    "full_name": "John Doe",
    "email": "john.doe@company.com",
    "all_attributes": {},
    "direct_manager": {
      "hr_employee_id": "MGR001",
      "full_name": "Jane Smith",
      "email": "jane.smith@company.com"
    },
    "risk_profile": {
      "risk_level": "Medium",
      "risk_score": 65
    },
    "organizational_assignments": [
      {
        "department": "Engineering",
        "business_function": "Software Development",
        "department_manager": {
          "hr_employee_id": "MGRDEPT001",
          "full_name": "Robert Johnson",
          "email": "robert.johnson@company.com"
        }
      }
    ],
    "associated_accounts": [
      {
        "repository": "Active Directory",
        "login": "jdoe",
        "status": "Active",
        "last_login": "2025-11-28T09:15:00Z"
      }
    ],
    "issues": [
      {
        "attached_to": "account",
        "account_login": "jdoe",
        "account_repository": "Active Directory",
        "control_name": "Privileged Account Usage Monitor",
        "control_description": "Ensures privileged accounts are moni",
        "risk_description": "Potential unauthorized access detected",
        "risk_level": "High",
        "recommended_remediation": "Review access logs and disable i",
        "resolution_status": "In Progress",
        "detection_date": "2025-11-25T08:00:00Z"
      }
    ]
  },
  "status": "success"
}
```

## Access Account Unique Identifier Lookup

Use this service to find an account ID in a specific system.

It searches for the account and returns a unique account identifier, which can then be used to retrieve the account's full context.

### Input Parameters

The query must include:

- **Account Login OR Email Address**

AND

- **Repository/System Name**

Combined logic:

```
(Login OR Email) AND Repository
```

This ensures that the correct system is searched when looking for the account.

### Possible Results

#### No Result

No account matches the provided criteria.

#### One Result

Exactly one account matches.

The response includes the unique account identifier, which can be used to retrieve the full account context.

#### Multiple Results

More than one account matches the criteria.

Additional information is returned so the requester can determine which account is correct.

#### JSON Response Example

```json
{
  "result_count": 1,
  "results": [
    {
      "account_id": "CLAUDIA_NEAL_1730380876537_3683",
      "email": null,
      "employee_number": null,
      "full_name": "ACME.COM:claudia.neal@acme.com",
      "login": "claudia.neal@acme.com",
      "repository": "ACME.COM"
    }
  ],
  "status": "success"
}
```

## Access Account Context Retrieval

Use this service to retrieve detailed information about a specific account.

This includes information about:

- the system where the account exists
- who owns or manages the account
- the groups and resources the account can access
- the permissions granted to the account
- any detected issues or risks

### Input Parameters

Required parameter:

- **Unique Account Identifier**
  (obtained from the Account Lookup service)

### Response Data: Complete Account Context

The response provides several types of information about the account.

#### Core Account Attributes

All attributes stored for the account in the platform.

#### Repository or System Information

Details about the system that hosts the account.

Includes:

- Repository ID
- Repository Name
- Repository Description
- Repository Type (for example: Directory Service, Database, Cloud Application)

#### Account Risk Profile

Risk indicators associated with the account.

- **Risk Level** (Low / Medium / High / Critical)
- **Risk Score** (numeric)

#### Account Owner or Manager Information

The information returned depends on the account type.

**User Accounts**

Includes information about the account owner:

- Owner HR Employee ID
- Owner Full Name
- Owner Email Address
- Owner Status (Active / Inactive)
- Owner Departure Date (if applicable)

Also includes the owner's manager:

- Manager HR Employee ID
- Manager Full Name
- Manager Email Address

**Technical or Service Accounts**

Instead of a single owner, these accounts may have one or more managers.

For each manager:

- HR Employee ID
- Full Name
- Email Address
- Status
- Departure Date (if applicable)

#### Associated Groups

Lists the groups the account belongs to.

Each group entry includes:

- Group Name
- Group Description
- Association Type (Direct or Indirect)
- Repository information

#### Associated Resources and Permissions

Lists the resources that the account can access.

For each resource:

- Resource Identifier
- Resource Name
- Resource Description
- Resource Type

Permissions granted to the account include:

- Permission Identifier
- Permission Name
- Permission Description
- Permission Type

#### Account Issues and Detected Problems

Any issues detected for the account.

Each issue includes:

**Control Information**

- Control Name
- Control Description

**Risk Details**

- Risk Description
- Risk Level

**Remediation Guidance**

- Recommended remediation action
- Current resolution status
- Issue detection date

### JSON Response Example

```
{
	"result_count": 1,
	"results": {
	  "account": [
		{
		  "account_id": "claudia.neal",
		  "control_defects": [
			{
			  "control": [
				{
				  "control_category": "test",
				  "control_creation_date": "30/03/26",
				  "control_description": "test control",
				  "control_displayname": "CONTROL 1 HELLO",
				  "control_entity": "account",
				  "control_modification_date": "30/03/26",
				  "control_name": "CONTROL 0001 YES",
				  "control_nb_defects_1": 1,
				  "control_nb_defects_2": 1,
				  "control_nb_defects_3": 1,
				  "control_nb_defects_4": 1,
				  "control_nb_defects_week_1": 1,
				  "control_nb_defects_week_2": 1,
				  "control_nb_defects_week_3": 1,
				  "control_nb_defects_week_4": 1,
				  "control_remediation_policy": "Remediation policy",
				  "control_remove_policy": "None",
				  "control_risk_description": "This is a risk description",
				  "control_risk_level": 10,
				  "control_suggested_action": "Suggested action",
				  "control_updated_date": "30/03/26",
				  "id": "CONTROL-0001-ACCOUNT"
				},
				{
				  "control_category": "test",
				  "control_creation_date": "30/03/26",
				  "control_description": "test control",
				  "control_displayname": "CONTROL 1 HELLO",
				  "control_entity": "identity",
				  "control_modification_date": "30/03/26",
				  "control_name": "CONTROL 0001 YES",
				  "control_nb_defects_1": 1,
				  "control_nb_defects_2": 1,
				  "control_nb_defects_3": 1,
				  "control_nb_defects_4": 1,
				  "control_nb_defects_week_1": 1,
				  "control_nb_defects_week_2": 1,
				  "control_nb_defects_week_3": 1,
				  "control_nb_defects_week_4": 1,
				  "control_remediation_policy": "Remediation policy",
				  "control_remove_policy": "None",
				  "control_risk_description": "This is a risk description",
				  "control_risk_level": 10,
				  "control_suggested_action": "Suggested action",
				  "control_updated_date": "30/03/26",
				  "id": "CONTROL-0001-IDENTITY"
				}
			  ],
			  "control_defect_audit_trail": "audit_trail_test",
			  "control_defect_created_at": "30/03/2026",
			  "control_defect_displayname": "Control Defect 01",
			  "control_defect_is_closed": false,
			  "control_defect_last_update": "31/03/2026",
			  "control_defect_last_who_name": "name",
			  "control_defect_remediation_uuid": "0001",
			  "control_defect_risk_level": 23,
			  "control_defect_status": "TEST",
			  "control_defect_third_party_status": "TESTING",
			  "id": "CTRL-DEFECT-001"
			}
		  ],
		  "creation_date": "Creation date not available in ACME.COM",
		  "default_mfa_method_strength": "False",
		  "default_mfa_method_type": "none",
		  "disabled": true,
		  "dn": "claudia.neal",
		  "dont_expire_password": false,
		  "full_name": "ACME.COM:claudia.neal@acme.com",
		  "groups": [
			{
			  "dn": "EVERYONE_EXCEPT_EXTERNAL_USERS",
			  "group_name": "EVERYONE_EXCEPT_EXTERNAL_USERS",
			  "id": "EVERYONE_EXCEPT_1730380876194_3519",
			  "sensitivity_level": 0
			}
		  ],
		  "id": "CLAUDIA_NEAL_1730380876537_3683",
		  "is_passwordless_active": "False",
		  "is_passwordless_required": "True",
		  "is_sspr_allowed": "True",
		  "is_sspr_registered": "False",
		  "last_login": "Last login date not available in ACME.COM",
		  "locked": false,
		  "login": "claudia.neal@acme.com",
		  "mfa_active": "False",
		  "mfa_allowed": "False",
		  "mfa_properly_configured": "False",
		  "mfa_properly_configured_reason": "Mfa is disabled",
		  "mfa_registered": "False",
		  "mfa_required": "True",
		  "owner": [
			{
			  "arrival_date": "US",
			  "departments": [
				{
				  "department": "DMARPR",
				  "department_short_name": "DMARPR",
				  "id": "DMARPR_1730380867151_45",
				  "identity_job_title": "Marketing Advisor",
				  "managers": [
					{
					  "arrival_date": "US",
					  "email": "tbenet@radiantlogic.com",
					  "employee_type": "Internal",
					  "full_name": "Thibeau BENET",
					  "given_name": "Thibeau",
					  "hr_employee_id": "ID0000101",
					  "id": "ID0000101_1730380868867_527",
					  "internal": true,
					  "status": true,
					  "surname": "BENET",
					  "title": "Mr"
					}
				  ],
				  "team_members": [
					{
					  "arrival_date": "US",
					  "email": "tbenet@radiantlogic.com",
					  "employee_type": "Internal",
					  "full_name": "Thibeau BENET",
					  "given_name": "Thibeau",
					  "hr_employee_id": "ID0000101",
					  "id": "member_id",
					  "internal": true,
					  "resources": [
						{
						  "description": "Company's Enterprise Resource Planning system (ERP)",
						  "id": "resource_id",
						  "resource_family": "SAP",
						  "resource_name": "Main-FS2-Graphics Solution",
						  "resource_type": "Share",
						  "sensitivity_level": 0
						}
					  ],
					  "status": true,
					  "surname": "BENET",
					  "title": "Mr"
					}
				  ]
				}
			  ],
			  "departure_date": "Departure date not available in HR",
			  "email": "cneal17@acme.com",
			  "employee_type": "Internal",
			  "full_name": "Claudia NEAL",
			  "given_name": "Claudia",
			  "hr_employee_id": "ID0000056",
			  "hr_repository": [
				{
				  "id": "1730380822875_402",
				  "repository_name": "HR",
				  "repository_type": "Accounts"
				}
			  ],
			  "id": "ID0000056_1730380868289_345",
			  "internal": true,
			  "managers": [
				{
				  "arrival_date": "US",
				  "email": "tbenet@radiantlogic.com",
				  "employee_type": "Internal",
				  "full_name": "Thibeau BENET",
				  "given_name": "Thibeau",
				  "hr_employee_id": "ID0000101",
				  "id": "ID0000101_1730380868867_527",
				  "internal": true,
				  "status": true,
				  "surname": "BENET",
				  "title": "Mr"
				}
			  ],
			  "status": true,
			  "surname": "NEAL",
			  "title": "Ms"
			}
		  ],
		  "password_cant_change": false,
		  "password_expired": false,
		  "password_not_required": false,
		  "privileged_account": false,
		  "reconciliation_comment": "fullname with username",
		  "reconciliation_date": "20241031142209",
		  "reconciliation_description": "fullname with username",
		  "reconciliation_rule": "FULLNAME_USERNAME",
		  "repository": [
			{
			  "id": "ACME_COM_1730380876181_3517",
			  "is_mfa_capable": true,
			  "is_passwordless_capable": true,
			  "is_sspr_capable": true,
			  "repository_family": "Azure",
			  "repository_name": "ACME.COM",
			  "repository_type": "Accounts"
			}
		  ],
		  "risks": [
			{
			  "account_nb_defects": 0,
			  "agg_risk_1": 0,
			  "agg_risk_2": 0,
			  "agg_risk_3": 0,
			  "agg_risk_4": 0,
			  "agg_risk_level": 0,
			  "agg_risk_score": 0,
			  "int_risk_1": 0,
			  "int_risk_2": 0,
			  "int_risk_3": 0,
			  "int_risk_4": 0,
			  "int_risk_level": 0,
			  "int_risk_score": 0,
			  "sensitivity_level": 0
			}
		  ],
		  "samaccountname": "claudia.neal@acme.com",
		  "secondary_mfa_method_types": "none"
		}
	  ]
	},
	"status": "success"
  }
```
