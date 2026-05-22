# Administrator and Integrator Guide 

The MCP (Model Context Protocol) Server lets an AI assistant such as Cursor or n8n answer questions about people and accounts in your Identity Observability catalog without code or queries. This guide covers all the steps needed to provision credentials in Keycloak, and reference for the four tools the MCP exposes.

For the end user tutroal (configuring Cursor or n8n with a token), see the [Quickstart tutorial](url).

## How It Works

1. **Provision credentials (admin).** Create an OIDC client per user or automation in Keycloak, grant it the `mcp-access` role, and set a token lifespan appropriate to the use case.
2. **Generate a token or share the secret (admin).** For chat users, mint a long lived access token via `client_credentials` and hand it over. For automation, share the `client_id` + `client_secret` so the workflow can fetch fresh tokens itself.
3. **Integrate the four tools (integrator).** Point your client at the MCP endpoint and call `fetch_account_id`, `fetch_identity_id`, `get_account_context`, or `get_identity_context` as documented in the [Tools and Data Reference](#tools-and-data-reference).


## Admin Setup in RadiantLogic Identity Observability

These steps require **realm-admin** privileges on the tenant realm in Keycloak. On hardened production deployments, your IT team may have removed this capability from line of business administrators. Verify your permissions before attempting to provide MCP access to a user.

Replace these placeholders with your actual platform values before running any command.

| Placeholder | What it means | Example |
| --- | --- | --- |
| `<tenant>` | Identity Observability deployment identifier. Used as both the APISIX URL path segment and the Keycloak realm name. | `acme`, `acme-prod` |
| `<app-external-dns>` | Public hostname of the Identity Observability application gateway (APISIX). | `ido.example.com` |
| `<auth-external-dns>` | Public hostname of Keycloak. | `auth.example.com` |
| `<client_id>` / `<client_secret>` | OIDC client credentials provisioned in Keycloak. One pair per end user or automation. | `mcp-john-doe` |
| `<your-long-lived-token>` | Access token minted from a client pair, carrying the `mcp-access` role. | JWT (~1.5 kB) |

### 1. Open the Keycloak Admin Console

All authentication and access related operations in this guide are performed in the Keycloak Admin Console within the tenant realm. There is no separate Identity Observability UI for this workflow/

1. Go to `https://<auth-external-dns>/auth/admin`.
2. Sign in with a realm-admin account.
3. Select the **tenant realm** in the top left dropdown.

![Keycloak Admin Console with the tenant realm selector highlighted](./Media/screenshot-024.png)

### 2. Create the OIDC Client

Each end user or automation gets its own client. One client per user is what makes revocation, auditing, and TTL tuning workable.

1. In the left menu, go to **Clients → Create client**.

   ![Clients list with the Create client button highlighted](./Media/screenshot-025.png)

2. Set **General Settings**:

   * **Client type:** `OpenID Connect`
   * **Client ID:** a descriptive name reflecting who the client is for, for example `mcp-john-doe` (chat) or `mcp-n8n-finance` (automation).
   * **Name / Description:** optional.

3. Click **Next**.

4. Set **Capability config**:

   * **Client authentication:** `ON` (confidential client; the secret stays server side).
   * **Direct access grants:** `ON`
   * **Service accounts roles:** `ON`

   ![Capability config default state with toggles off](./Media/screenshot-026.png)

   The result should match:

   ![Capability config with Client authentication on and both grants enabled](./Media/screenshot-027.jpg)

5. Click **Next**, leave **Login settings** empty, click **Save**.

6. Open the **Credentials** tab and copy the **Client Secret**. Share this with the end user (for n8n), or use it to mint a long lived token (for chat).

   ![Credentials tab with the Client Secret copy button highlighted](./Media/screenshot-028.png)

### 3. Grant the `mcp-access` Role

The MCP middleware checks that incoming tokens carry the `mcp-access` role. Without it, any request is rejected with HTTP 403.

1. From the client detail page, open **Service accounts roles**.
2. Click **Assign role**.

   ![Service accounts roles tab with Assign role highlighted](./Media/screenshot-029.png)

3. Search for `mcp-access`, tick it, click **Assign**.

   ![Assign Client roles dialog with mcp-access selected](./Media/screenshot-030.png)

> `mcp-access` is a composite role that already includes `user`, so the service account automatically gets the basic Identity Observability read permissions it needs. No second role is required.

### 5. Set the Access Token Lifespan (TTL)

TTL is key setting where you can distinguish **chat mode** usage from **automation mode**.

1. Open the client's **Advanced** tab.

   ![Client detail page with the Advanced tab selected](./Media/screenshot-031.png)

2. Go to **Advanced Settings → Access Token Lifespan**.

   ![Advanced settings sidebar with Access Token Lifespan highlighted](./Media/screenshot-032.png)

3. Set the value:

 | Use case                           | Recommended TTL                                          | Reason                                                                                                                                 |
| ---------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Chat mode** (Cursor)             | 30 days (or the maximum allowed by your security policy) | The token is manually copied into a configuration file and cannot be automatically refreshed.                                             |
| **Automation mode** (n8n, scripts) | 15 minutes (short-lived)                                 | The workflow retrieves a new token each run, so short TTLs have no operational cost and significantly reduce the impact of token leakage. |


4. Click **Save**.

   ![Access Token Lifespan field set to 30 days](./Media/screenshot-033.png)

5. **For chat usage only.** Set **Realm settings → Sessions → SSO Session Max** to the same value, then **Save**.

   ![SSO Session Max field set to 30 days](./Media/screenshot-034.png)

> **One client per user, always.** Granular revocation (disable one client at offboarding without breaking everyone else), per client `client_id` in audit logs, and per client TTLs (30 day chat tokens vs 15 minute automation tokens) all depend on it.

### 6. Generate a Token (Chat Mode) and Deliver Credentials

In chat mode the user does not run OAuth themselves; they paste a token you mint for them.

```bash
curl -X POST \
  "https://<auth-external-dns>/auth/realms/<tenant>/protocol/openid-connect/token" \
  -d "grant_type=client_credentials" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>" \
  | jq -r '.access_token'
```

Deliver the resulting token to the user along with the MCP URL (`https://<app-external-dns>/<tenant>/mcp/`).

### 7. Revoke Access

Use the option that matches the situation:

| Option | What it does | Use when |
| --- | --- | --- |
| **Disable the client** (toggle *Enabled* OFF) | All tokens for this client stop working immediately. | Standard offboarding. |
| **Remove the `mcp-access` role** | Token stays cryptographically valid, but the MCP rejects it with 403. | You want to keep the client for other roles but block MCP. |
| **Not Before policy** (Advanced → *Not Before* = now) | Every token issued before now becomes invalid; new ones still work. | Suspected leak: invalidate in flight tokens without disabling the client. |

For fast single token revocation:

```bash
curl -X POST \
  "https://<auth-external-dns>/auth/realms/<tenant>/protocol/openid-connect/revoke" \
  -d "token=$TOKEN" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>" \
  -d "token_type_hint=access_token"
```

## Tools and Data Reference

This section is for integrators who need to know exactly what the MCP returns, field by field.

### 1. What Information the MCP Can Return

The MCP exposes two complementary views: **accounts** (the technical objects in your repositories) and **identities** (the people, aggregated across all their accounts).

**Accounts** are populated by `get_account_context`.

| Group | Fields |
| --- | --- |
| Identifiers | `id`, `account_id`, `login`, `samaccountname`, `dn`, `email`, `full_name`, `given_name`, `surname` |
| Lifecycle | `creation_date`, `last_modification_date`, `last_login`, `login_count`, `disabled`, `locked` |
| Password policy | `password_expired`, `password_not_required`, `password_cant_change`, `password_last_set_date`, `bad_password_date`, `dont_expire_password` |
| MFA | `mfa_active`, `mfa_allowed`, `mfa_required`, `mfa_registered`, `mfa_properly_configured`, `mfa_properly_configured_reason`, `default_mfa_method_type`, `default_mfa_method_strength`, `latest_mfa_method_type_used`, `latest_mfa_method_strength_used`, `secondary_mfa_method_types`, `secondary_mfa_method_strength` |
| SSPR / Passwordless | `is_sspr_allowed`, `is_sspr_registered`, `is_passwordless_active`, `is_passwordless_required` |
| Reconciliation | `reconciliation_type`, `reconciliation_rule`, `reconciliation_reliability` |
| Privilege | `privileged_account` |
| Relationships | `repository` (source system), `owner` (the identity the account is reconciled to, with HR data, manager, departments, full account list) |
| Authorization | `groups[]`, `permissions[]` (entitlements with linked `resource`) |
| Risk and quality | `risks[]` (aggregated and intrinsic levels and scores, sensitivity), `control_defects[]` (each defect carries the full control definition and audit trail) |

**Identities** are populated by `get_identity_context`.

| Group | Fields |
| --- | --- |
| Identifiers | `id`, `hr_employee_id`, `full_name`, `given_name`, `surname`, `email` |
| Lifecycle | `arrival_date`, `departure_date`, `status`, `internal` |
| Org | `departments[]` (each with `department_short_name`, `identity_job_title`, `managers[]`) |
| Accounts | `accounts[]`; per account: `id`, `groups[]`, `permissions[]`, `control_defects[]` |
| Risk | `risks[]`; same dual axis structure (`agg_`, `int_`, `identity_nb_defects`, `sensitivity_level`) |
| Defects | `control_defects[]`; identity level defects (e.g. *contractor with past ending date*) |

**Reading the risk block.** Identity Observability computes risk on two axes:

* `int_risk_*` (intrinsic): risk from the account's own attributes (privileged, no MFA, dormant).
* `agg_risk_*` (aggregated): propagated from resources and permissions the account holds.

`*_risk_level` is a bucket (1..4, higher = riskier); `*_risk_score` is the numeric breakdown. Use `*_risk_level` for everyday questions, `*_risk_score` only for sorting.

### 2. Tools

The MCP Server exposes **four tools** in GA. The schemas below are exactly what the LLM sees through `tools/list`.

**Common conventions**

* All responses wrap into `{ "results": ..., "result_count": <int>, "status": "success" | "error", "error": "<optional>" }`.
* No match → call succeeds with `result_count: 0` and `results: []`. There is no dedicated *not found* error.
* Date fields not exposed by the source repository return a human readable sentinel string (e.g. `"Last login date not available in <repository_name>"`); always check value type before parsing.

#### 2.1 `fetch_account_id`

 Find a unique account by login or email **inside a specific repository**. Call first to obtain the `account_id` that `get_account_context` needs.

| Input | Type | Required | Description |
| --- | --- | --- | --- |
| `account_name` | string | yes | Account Name OR Login OR Email Address |
| `repository_name` | string | yes | Repository / System Name (e.g. `AD_CORP`, `HR`) |

**Output schema (top level)**

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

**Sample request**

```json
{
  "name": "fetch_account_id",
  "arguments": {
    "account_name": "evelyn.estrada@example.com",
    "repository_name": "AD_CORP"
  }
}
```

**Sample response**

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

A login search (`"account_name": "eestrada"`) on the same repository returns the same record. A non existing repository returns `result_count: 0` (not an error).

#### 2.2 `fetch_identity_id`

 Find a unique identity by HR ID, full name, or corporate email. Call first to obtain the `identity_id` that `get_identity_context` needs.

| Input | Type | Required | Description |
| --- | --- | --- | --- |
| `identity_name` | string | yes | HR Employee ID OR Full Name OR Email Address |

**Output schema (top level)**

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

**Sample request and response (lookup by email)**

```json
{
  "name": "fetch_identity_id",
  "arguments": { "identity_name": "lawrence.brown@example.com" }
}
```

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

The same result comes back for `"Lawrence Brown"` (full name) and `"E000065"` (HR ID). A partial name (`"Brown"`) or unknown identity returns `result_count: 0`.

#### 2.3 `get_account_context`

Returns the comprehensive context of an account: attributes, owner, repository, groups, permissions, risks, control defects. Use this to answer *"what does this account do, who owns it, is it risky?"*.

| Input | Type | Required | Description |
| --- | --- | --- | --- |
| `account_id` | string | yes | Unique account identifier (from `fetch_account_id`) |

**Output structure.** See section [1. What Information the MCP Can Return](#1-what-information-the-mcp-can-return) for the field by field listing of `results.account[0]`.

**Sample request**

```json
{
  "name": "get_account_context",
  "arguments": { "account_id": "029c789f3480fae1249ddaeed314f335" }
}
```

**Sample response (excerpt; full payload is several kB)**

```json
{
  "results": {
    "account": [
      {
        "account_id": "9178d0c8-7868-624e-a477-d008d6773091",
        "id": "029c789f3480fae1249ddaeed314f335",
        "login": "eestrada",
        "full_name": "Evelyn Estrada",
        "email": "evelyn.estrada@example.com",
        "dn": "cn=evelyn estrada,ou=group 2,ou=optometry,ou=root department,dc=corp,dc=example,dc=com",
        "disabled": false,
        "locked": false,
        "privileged_account": true,
        "mfa_active": "MFA not supported by AD_CORP",
        "repository": { "repository_name": "AD_CORP", "repository_family": "AD", "repository_type": "Accounts" },
        "owner": {
          "hr_employee_id": "E000155",
          "full_name": "Evelyn Estrada",
          "managers": [ { "full_name": "William Bryant", "email": "william.bryant@example.com" } ],
          "departments": [ { "department_short_name": "OPT_GRP2", "identity_job_title": "Optometrist" } ]
        },
        "risks": [ { "agg_risk_level": 4, "agg_risk_score": 1000101, "int_risk_level": 4, "int_risk_score": 1000101, "account_nb_defects": 3 } ],
        "groups": [ /* ... */ ],
        "permissions": [
          {
            "permission_displayname": "Nagios_Database Administrator",
            "resource": [ { "resource_displayname": "Nagios", "resource_family": "AD", "resource_type": "Profile" } ]
          }
          /* ... more permissions ... */
        ],
        "control_defects": [
          {
            "control_defect_status": "new",
            "control": [
              {
                "control_name": "Identity Observability_ACC01",
                "control_displayname": "Dormant Accounts",
                "control_category": "Hygiene",
                "control_risk_level": 1,
                "control_description": "Enabled accounts whose Last Login Date attribute is more than 60 days ago",
                "control_suggested_action": "Monitor account last login date..."
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

#### 2.4 `get_identity_context`

 Returns the comprehensive context of an identity (the *person* view): HR attributes, manager and department, every account the person owns with their groups, permissions, and defects, plus identity level risks and defects.

| Input | Type | Required | Description |
| --- | --- | --- | --- |
| `identity_id` | string | yes | Unique identity identifier (from `fetch_identity_id`) |

**Output structure.** See section [1. What Information the MCP Can Return](#1-what-information-the-mcp-can-return) for a detailed breakdown of the fields in results.identity[0]. `results.identity[0]`.

**Sample request**

```json
{
  "name": "get_identity_context",
  "arguments": { "identity_id": "1daf7bca9fe71c076b3778d2d0b29659" }
}
```

**Sample response (excerpt; full payload is ~100 kB)**

```json
{
  "results": {
    "identity": [
      {
        "id": "1daf7bca9fe71c076b3778d2d0b29659",
        "hr_employee_id": "E000065",
        "full_name": "Lawrence Brown",
        "email": "lawrence.brown@example.com",
        "internal": false,
        "status": false,
        "arrival_date": "US",
        "departure_date": "Departure date not available in HR",
        "departments": [
          {
            "department_short_name": "R&D_OPT4",
            "identity_job_title": "Dispensing optician",
            "managers": [ { "hr_employee_id": "E000096", "full_name": "Barbara Mcgrath", "email": "barbara.mcgrath@example.com" } ]
          }
        ],
        "risks": [ { "agg_risk_level": 4, "agg_risk_score": 2000101, "int_risk_level": 4, "int_risk_score": 1000000, "identity_nb_defects": 1 } ],
        "accounts": [
          {
            "id": "acc949ad1b38be8a8847cb6296804d5b",
            "groups": [ /* ... */ ],
            "permissions": [
              {
                "permission_displayname": "CPSI_Medical Director",
                "resource": [ { "resource_displayname": "CPSI", "resource_family": "AD", "resource_type": "Profile" } ]
              }
            ],
            "control_defects": [ /* ... */ ]
          }
        ],
        "control_defects": [
          {
            "control_defect_status": "new",
            "control": [
              {
                "control_name": "Identity Observability_HR10",
                "control_displayname": "Contractor with past ending date and active accounts",
                "control_category": "Lifecycle",
                "control_risk_level": 4,
                "control_description": "Identities who are contractors set as inactive or whose departure date has passed and owning active accounts",
                "control_suggested_action": "Double check with the contractor manager."
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

> **Tip for the LLM consumer.** Each context can be ~100 kB. When asking *"compare two identities"*, focus the model on a single dimension (risk profile, or permissions) so both responses fit in context.

### 3. Manual `curl` Validation

For troubleshooting outside any client, the MCP protocol requires three calls:

```bash
TOKEN="<your access token>"
MCP_URL="https://<app-external-dns>/<tenant>/mcp/"

# 1) Initialize: capture the Mcp-Session-Id header from the response
curl -i -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"curl-test","version":"1.0"}}}'

# 2) Confirm initialization
SESSION_ID="<paste Mcp-Session-Id value>"
curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","method":"notifications/initialized"}'

# 3) List tools
curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'

# 4) Call a tool, for example fetch_identity_id
curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"fetch_identity_id","arguments":{"identity_name":"lawrence.brown@example.com"}}}'
```

For local or dev environments with self signed certificates, add `-k` to every `curl` call. **Never use `-k` in production.**

