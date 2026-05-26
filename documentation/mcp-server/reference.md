# MCP tools and data reference

The Identity Observability MCP server exposes tools that can be invoked by language models.  Each tool is uniquely identified by a name and includes metadata describing its schema.

This document servea as a tools and data reference guide for integrators or anyone who needs a detailed view of what the MCP Server returns, field by field. 

## 1. Data returned by the MCP Server

The MCP Server exposes two complementary views of your identity landscape: accounts and identities. Accounts represent technical objects in connected repositories. Identities represent people, aggregated across all of their accounts. 

### 1.1 Account data (get_account_context)

The `get_account_context` tool returns the full context for an account. Fields are grouped below by theme.

| Group | Fields |
| --- | --- |
| Identifiers | `id`, `account_id`, `login`, `samaccountname`, `dn`, `email`, `full_name`, `given_name`, `surname` |
| Lifecycle | `creation_date`, `last_modification_date`, `last_login`, `login_count`, `disabled`, `locked` |
| Password policy | `password_expired`, `password_not_required`, `password_cant_change`, `password_last_set_date`, `bad_password_date`, `dont_expire_password` |
| MFA | `mfa_active`, `mfa_allowed`, `mfa_required`, `mfa_registered`, `mfa_properly_configured`, `mfa_properly_configured_reason`, `default_mfa_method_type`, `default_mfa_method_strength`, `latest_mfa_method_type_used`, `latest_mfa_method_strength_used`, `secondary_mfa_method_types`, `secondary_mfa_method_strength` |
| SSPR / passwordless | `is_sspr_allowed`, `is_sspr_registered`, `is_passwordless_active`, `is_passwordless_required` |
| Reconciliation | `reconciliation_type`, `reconciliation_rule`, `reconciliation_reliability` |
| Privilege | `privileged_account` |
| Relationships | `repository` (repository where the account lives), `owner` (reconciled identity, including HR attributes, manager, departments, and full account list) |
| Authorization | `groups[]` (group membership), `permissions[]` (entitlements), each with a linked `resource` |
| Risk and quality | `risks[]` (aggregated and intrinsic risk levels and scores, sensitivity), `control_defects[]` (each defect includes the full control definition and audit trail) |

### 1.2 Identity data (get_identity_context)

The `get_identity_context` tool returns the full context for an identity (the person view). Fields are grouped below by theme.

| Group | Fields |
| --- | --- |
| Identifiers | `id`, `hr_employee_id`, `full_name`, `given_name`, `surname`, `email` |
| Lifecycle | `arrival_date`, `departure_date`, `status`, `internal` |
| Organization | `departments[]` (each department has `department_short_name`, `identity_job_title`, and `managers[]`) |
| Accounts | `accounts[]` – for each account: `id`, `groups[]`, `permissions[]`, `control_defects[]` |
| Risk | `risks[]` – same dual-axis structure as accounts (`agg_*`, `int_*`, `identity_nb_defects`, `sensitivity_level`) |
| Defects | `control_defects[]` – identity-level defects, such as “contractor with past ending date and active accounts” |

### 1.3 Interpreting the risk model

Identity Observability computes risk along two axes:

- `int_risk_*` (intrinsic risk): risk introduced by the account or identity itself, for example privileged access, missing MFA, or dormant accounts.  
- `agg_risk_*` (aggregated risk): risk propagated from the resources and permissions associated with the account or identity. 

For each axis:

- `*_risk_level` is a bucket from 1 to 4, where higher values indicate higher risk.  
- `*_risk_score` is a numeric value designed for ranking and sorting.  

Use `*_risk_level` for everyday questions and user-facing explanations. Use `*_risk_score` when precise sorting or prioritization is required. 

## 2. Tools

The MCP Server exposes four tools in general availability. Tool descriptions, input parameters, and output schemas below match what an MCP client sees via the `tools/list` method described in the MCP specification. 

### 2.1 Common conventions

All tool responses follow a common wrapper structure:

```json
{
  "results": ...,
  "result_count": 0,
  "status": "success",
  "error": "string (optional, only on failure)"
}
```

Additional conventions:

- Lookups that do not find a match return `status: "success"`, `result_count: 0`, and `results: []`. There is no dedicated “not found” error.  
- Date fields that are not exposed by the source repository return a human-readable sentinel string, such as `"Last login date not available in <repository_name>"`. Clients should always check the value type before parsing dates. [web:35][web:38]

### 2.2 `fetch_account_id`

Finds a unique account by login or email within a specific repository. Typically called first to obtain the `account_id` required by `get_account_context`.

#### Input parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `account_name` | string | yes | Account name, login, or email address. |
| `repository_name` | string | yes | Repository or system name to search, such as `AD_CORP` or `HR`. |

#### Output schema (top level)

```json
{
  "results": [
    {
      "account_id":      "string",
      "email":           "string | null",
      "employee_number": "string | null",
      "full_name":       "string | null",
      "login":           "string | null",
      "repository":      "string"
    }
  ],
  "result_count": 0,
  "status": "success",
  "error": "string (optional, only on failure)"
}
```

#### Sample request

```json
{
  "name": "fetch_account_id",
  "arguments": {
    "account_name": "evelyn.estrada@example.com",
    "repository_name": "AD_CORP"
  }
}
```

#### Sample response

```json
{
  "results": [
    {
      "account_id": "029c789f3480fae1249ddaeed314f335",
      "email": "evelyn.estrada@example.com",
      "employee_number": null,
      "full_name": "Evelyn Estrada",
      "login": "eestrada",
      "repository": "AD_CORP"
    }
  ],
  "result_count": 1,
  "status": "success"
}
```

A login-based search such as `"account_name": "eestrada"` on the same repository returns the same record. A lookup using a non-existing repository name returns `result_count: 0` with no error.

### 2.3 `fetch_identity_id`

Finds a unique identity by HR identifier, full name, or corporate email. Typically called first to obtain the `identity_id` required by `get_identity_context`.

#### Input parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `identity_name` | string | yes | HR employee ID, full name, or email address. |

#### Output schema (top level)

```json
{
  "results": [
    {
      "identity_id":    "string",
      "email":          "string",
      "full_name":      "string",
      "hr_employee_id": "string"
    }
  ],
  "result_count": 0,
  "status": "success",
  "error": "string (optional)"
}
```

#### Sample: lookup by email

**Request**

```json
{
  "name": "fetch_identity_id",
  "arguments": {
    "identity_name": "lawrence.brown@example.com"
  }
}
```

**Response**

```json
{
  "results": [
    {
      "email": "lawrence.brown@example.com",
      "full_name": "Lawrence Brown",
      "hr_employee_id": "E000065",
      "identity_id": "1daf7bca9fe71c076b3778d2d0b29659"
    }
  ],
  "result_count": 1,
  "status": "success"
}
```

The same result is returned for `"identity_name": "Lawrence Brown"` (full name) and `"identity_name": "E000065"` (HR ID). Partial names such as `"Brown"` or non-existing identities return `result_count: 0`.

### 2.4 `get_account_context`

Returns comprehensive context for a single account, including attributes, owner, repository, group membership, permissions, risks, and control defects.

#### Input parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `account_id` | string | yes | Unique account identifier obtained from `fetch_account_id`. |

#### Output structure

The account payload is returned under `results.account[0]`. See the account field listing in section **1.1 Account data**.  

#### Sample request

```json
{
  "name": "get_account_context",
  "arguments": {
    "account_id": "029c789f3480fae1249ddaeed314f335"
  }
}
```

#### Sample response (excerpt)

```json
{
  "results": {
    "account": [
      {
        "account_id": "9178d0c8-7868-624e-a477-d008d6773091",
        "id": "029c789f3480fae1249ddaeed314f335",
        "login": "eestrada",
        "samaccountname": "eestrada",
        "full_name": "Evelyn Estrada",
        "email": "evelyn.estrada@example.com",
        "given_name": "Evelyn",
        "surname": "Estrada",
        "dn": "cn=evelyn estrada,ou=group 2,ou=optometry,ou=root department,dc=corp,dc=example,dc=com",
        "disabled": false,
        "locked": false,
        "privileged_account": true,
        "password_expired": false,
        "dont_expire_password": true,
        "mfa_active": "MFA not supported by AD_CORP",

        "repository": {
          "id": "902d138bd2eb570305bc1019007f4922",
          "repository_name": "AD_CORP",
          "repository_family": "AD",
          "repository_type": "Accounts",
          "description": "Account repository for the corporate Active Directory tenant"
        },

        "owner": {
          "id": "ee00974f9d5d6e9d5c02438a471a2c90",
          "hr_employee_id": "E000155",
          "full_name": "Evelyn Estrada",
          "email": "evelyn.estrada@example.com",
          "internal": false,
          "status": false,
          "managers": [
            {
              "id": "21cffff238af312a6412e5f87c1a88b4",
              "full_name": "William Bryant",
              "email": "william.bryant@example.com"
            }
          ],
          "departments": [
            {
              "department_short_name": "OPT_GRP2",
              "identity_job_title": "Optometrist"
            }
          ]
        },

        "risks": [
          {
            "agg_risk_level": 4,
            "agg_risk_score": 1000101,
            "int_risk_level": 4,
            "int_risk_score": 1000101,
            "account_nb_defects": 3,
            "sensitivity_level": 0
          }
        ],

        "groups": [
          {
            "id": "96938894a6a6df022a9d47ca986aea79",
            "group_name": "18a03f18-d9f9-c343-8332-6449c4674fa6",
            "dn": "cn=exchange server_database administrator,ou=exchange server,...",
            "sensitivity_level": 0
          }
          /* ... more groups ... */
        ],

        "permissions": [
          {
            "id": "198b2e14cebb9802cb60715e9a38b784",
            "permission_displayname": "Nagios_Database Administrator",
            "permission_comment": "cn=nagios_database administrator,ou=nagios,...",
            "resource": [
              {
                "id": "a4d0dcd495a92c8c65bddcc1cfca8c80",
                "resource_displayname": "Nagios",
                "resource_family": "AD",
                "resource_type": "Profile",
                "resource_risk/isens": 0
              }
            ]
          }
          /* ... more permissions ... */
        ],

        "control_defects": [
          {
            "id": "cdft_O8qQfA8Q7s9sojVa67uQ5A",
            "control_defect_status": "new",
            "control_defect_is_closed": false,
            "control_defect_created_at": "20260506125607",
            "control_defect_last_reason": "Appeared in observation 'IDO_ACC01'",
            "control": [
              {
                "id": "ctrl_2RMghb776oIADFelQMmJjC",
                "control_name": "IDO_ACC01",
                "control_displayname": "Dormant Accounts",
                "control_category": "Hygiene",
                "control_risk_level": 1,
                "control_description": "Enabled accounts whose Last Login Date attribute is more than 60 days ago",
                "control_suggested_action": "Monitor account last login date. If the last login date is longer than 60 days..."
              }
            ]
          }
          /* ... more defects ... */
        ]
      }
    ]
  },
  "result_count": 1,
  "status": "success"
}
```

### 2.5 `get_identity_context`

Returns comprehensive context for a single identity (person), including HR details, manager and department, all owned accounts (with groups, permissions, and defects), and identity-level risks and defects.

#### Input parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `identity_id` | string | yes | Unique identity identifier obtained from `fetch_identity_id`. |

#### Output structure

The identity payload is returned under `results.identity[0]`. See the identity field listing in section **1.2 Identity data**.

#### Sample request

```json
{
  "name": "get_identity_context",
  "arguments": {
    "identity_id": "1daf7bca9fe71c076b3778d2d0b29659"
  }
}
```

#### Sample response (excerpt)

```json
{
  "results": {
    "identity": [
      {
        "id": "1daf7bca9fe71c076b3778d2d0b29659",
        "hr_employee_id": "E000065",
        "full_name": "Lawrence Brown",
        "given_name": "Lawrence",
        "surname": "Brown",
        "email": "lawrence.brown@example.com",
        "internal": false,
        "status": false,
        "arrival_date": "US",
        "departure_date": "Departure date not available in HR",

        "departments": [
          {
            "id": "465433d3f25a28cfa507d642aa394658",
            "department_short_name": "R&D_OPT4",
            "identity_job_title": "Dispensing optician",
            "managers": [
              {
                "id": "7a7ae2188e8049f09873aa1400327cec",
                "hr_employee_id": "E000096",
                "full_name": "Barbara Mcgrath",
                "email": "barbara.mcgrath@example.com",
                "internal": false,
                "status": true
              }
            ]
          }
        ],

        "risks": [
          {
            "agg_risk_level": 4,
            "agg_risk_score": 2000101,
            "int_risk_level": 4,
            "int_risk_score": 1000000,
            "identity_nb_defects": 1,
            "sensitivity_level": 0
          }
        ],

        "accounts": [
          {
            "id": "acc949ad1b38be8a8847cb6296804d5b",
            "groups": [
              {
                "id": "0cb909a6738052015d3b0bc85b9a6f29",
                "group_name": "acef6ed2-07a9-be41-a477-f3d625bcddc3",
                "dn": "cn=active directory_cloud administrator,ou=active directory,...",
                "sensitivity_level": 0
              }
              /* ... more groups ... */
            ],
            "permissions": [
              {
                "id": "061fca9bad5b59d0397fc0d78d3d97ac",
                "permission_displayname": "CPSI_Medical Director",
                "permission_comment": "cn=cpsi_medical director,...",
                "resource": [
                  {
                    "id": "aed449cef0ce23617086b522511b4c07",
                    "resource_displayname": "CPSI",
                    "resource_family": "AD",
                    "resource_type": "Profile"
                  }
                ]
              }
              /* ... more permissions ... */
            ],
            "control_defects": [
              {
                "id": "cdft_O8qQfA8Q7s9sojVa67uQ5A",
                "control_defect_status": "new",
                "control_defect_is_closed": false,
                "control_defect_created_at": "20260506125607",
                "control": [
                  {
                    "control_name": "IDO_ACC01",
                    "control_displayname": "Dormant Accounts",
                    "control_category": "Hygiene",
                    "control_risk_level": 1
                  }
                ]
              }
              /* ... more account-level defects ... */
            ]
          }
        ],

        "control_defects": [
          {
            "id": "cdft_4zXgr2qFKFql5uFWOOhUIB",
            "control_defect_status": "new",
            "control_defect_is_closed": false,
            "control_defect_created_at": "20260506125607",
            "control_defect_last_reason": "Appeared in observation 'IDO_HR10'",
            "control": [
              {
                "control_name": "IDO_HR10",
                "control_displayname": "Contractor with past ending date and active accounts",
                "control_category": "Lifecycle",
                "control_risk_level": 4,
                "control_description": "Identities who are contractors set as inactive or whose departure date has passed and owning active accounts",
                "control_suggested_action": "Double-check with the contractor manager."
              }
            ]
          }
        ]
      }
    ]
  },
  "result_count": 1,
  "status": "success"
}
```

**Note for clients:** comparing two identities at once requires loading both contexts into the model. Each identity context can be around 100 KB. If reasoning appears truncated, constrain the request to a specific dimension such as “risk profile only” or “permissions only” so both responses fit within the model’s context window. [web:36][web:39]

## 3. Manual curl validation

The MCP HTTP transport uses a JSON-RPC lifecycle with initialization, operation, and tool calls, as described in the MCP specification. [web:39][web:42] The following sequence allows you to validate the MCP Server with `curl`.

```bash
TOKEN="<your access token>"
MCP_URL="https://<app-external-dns>/<tenant>/mcp/"
```

### 3.1 Initialize

Capture the `Mcp-Session-Id` header from the response.

```bash
curl -i -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"curl-test","version":"1.0"}}}'
```

### 3.2 Confirm initialization

Set `SESSION_ID` to the `Mcp-Session-Id` value from the previous response.

```bash
SESSION_ID="<paste Mcp-Session-Id value>"

curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","method":"notifications/initialized"}'
```

### 3.3 List tools

```bash
curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

### 3.4 Call a tool (example: `fetch_identity_id`)

```bash
curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"fetch_identity_id","arguments":{"identity_name":"lawrence.brown@example.com"}}}'
```

For local or development environments that use self-signed certificates, add `-k` to each `curl` command to skip TLS verification. Do not use `-k` in production.