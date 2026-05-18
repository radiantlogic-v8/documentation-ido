---
title: Quickstart tutorial
description: A quickstart guide on how to use the Identity Observability MCP Server
---


## Overview

The MCP (Model Context Protocol) Server lets your AI assistant (Cursor, n8n, and similar tools) answer questions about people and accounts in your IDO catalog, in plain English, without writing any code or query.

### Usage

You can ask about any person (an identity) and about any account they own in the systems. For each, the assistant can tell you:

- **Who they are:** name, email, employee id, job title, manager, department, arrival and departure dates.
- **What they can access:** which groups they belong to, which permissions and resources they hold, whether the account is privileged.
- **Their associated risks:** risk level and score, whether MFA is set up, account status (active / disabled / locked), how the account was reconciled to the person.

A question needs **at least one identity, account or resource named in it** (a person's name, an email, a login, a repository). The assistant can only answer when you point it at a precise target. Open-ended queries like *"show me everything risky"* will not work.

####  Sample questions

**Simple lookups**

- *"What resources does Lawrence Brown have access to?"*
- *"What are the risks associated with lawrence.brown@example.com?"*
- *"Is Evelyn Estrada in the Active Directory_Cloud Administrator group?"*
- *"Who is the manager of Lawrence Brown?"*
- *"Is the account eestrada in repository AD_CORP a privileged account?"*
- *"When was Lawrence Brown's account last used?"*

**Analysis questions** 

With these type of questions, the assistant reads several pieces of data and reasons across them. Example:

- *"Compare the risk profiles of Lawrence Brown and Evelyn Estrada and tell me who is the most exposed, and why."*
- *"List the resources that Lawrence Brown has access to but that none of his teammates in the same department have."*
- *"Is Lawrence Brown a contractor whose departure date has passed? If yes, what active accounts does he still have?"*

### Getting started

This section walks you through plugging the MCP server into your tool of choice (examples: Cursor or n8n). Follow the instructions based on your tool:

- **In your chat app (Cursor):** easiest, single user, conversational.
- **In an automation tool (n8n):** for workflows that run in an automated manner.

### In your chat app (Cursor)

Connecting the MCP to your chat app gives you a way to query IDO without leaving the environment you're already working in. Common use cases:

- **Ad-hoc investigation from your IDE:** paste an email, a login or an HR id encountered in code, in a ticket or in another tool, and ask the assistant for the full context (manager, groups, risks).
- **Conversational drilling:** start with a broad question (*"What risks are associated with this user?"*) and refine step by step (*"Focus on the sensitive groups, which other accounts hold them?"*).
- **Alert triage:** when an identity or an account has a defect, retrieve its context (risk level, owner, suggested remediation) before opening a ticket.
- **One-off comparisons:** request a side-by-side comparison of two identities in a single prompt (*"compare Alice's and Bob's access"*).


#### 1.1 Requirements from your administrator

Before configuring anything on your machine, request the following from the team that operates Identity Observability:

| Item | Looks like |
|---|---|
| **MCP URL** | An HTTPS link ending with `/mcp/`, e.g. `https://ido.example.com/acme/mcp/` |
| **Access token** | A long string with letters and numbers. Treat it like a password. |

Store your token securely and do not share it with others. You will need to paste the URL and the token into the chat app configuration as detailed below.


#### 1.3 Configure Cursor

Follow this guide step by step to configure the MCP Server on Cursor.


**Step 1.** Click on File, hover over Preferences, then click "Cursor Settings."

![Cursor File menu showing Preferences and Cursor Settings](images/cursor-01-file-menu.png)

**Step 2.** Click "Tools & MCPs."

![Cursor settings sidebar with Tools & MCPs highlighted](images/cursor-02-tools-mcps.png)

**Step 3.** Click on "Add Custom MCP" to open the `mcp.json` file.

![No MCP Tools panel with Add Custom MCP button](images/cursor-03-add-custom-mcp.png)

The `mcp.json` file opens.

![Empty mcp.json file in editor](images/cursor-04-empty-mcp-json.png)

**Step 4.** Replace the contents of the open file with the JSON below.

```json
{
  "mcpServers": {
    "IDO": {
      "url": "<MCP URL from your admin>",
      "headers": {
        "Authorization": "Bearer <access token from your admin>"
      }
    }
  }
}
```

Here is the result you should see in Cursor.

![Populated mcp.json file with URL and Bearer token](images/cursor-05-filled-mcp-json.png)

**Step 5.** Restart Cursor.

![Installed MCP Servers list showing IDO connected with available tools](images/cursor-06-connected.png)

The IDO server now appears as "Connected." The installation is complete. You can proceed directly to step 1.4 to test it.

#### 1.4 Try it out

Type a question about your own name or email. For example:

- *"Give me a summary of <your.email@example.com>."* to fetch an identity.
- *"Who is the owner of <your.email@example.com> on service ACME?"* to fetch an account.

If everything is configured correctly, the assistant returns a readable answer with real data about that person (name, manager, groups, risks). If you see *"I don't have access to that information,"* an error code, or hallucinated content, jump to Troubleshooting.

#### 1.5 Troubleshooting

The table below describes problems by **what you see in the chat**. For most issues you'll need an error code. See "How to read the error code" underneath the table.

| What you see | What's going on | What to do |
|---|---|---|
| No MCP server appears in the app's settings | The config file is in the wrong place, has a typo, or the app didn't pick it up. | Re-open the config file. Check that it's at the exact path described above. Make sure the URL and token have been copied without any extra spaces. Restart the app. |
| The assistant replies *"I don't have access to that information"* or shows error `401` | The token is missing, expired or invalid. | Check that the token has been pasted in the JSON configuration file. Check that the token has not been truncated or altered during the paste operation. If everything seems correct, contact your administrator with this error code "401". |
| The assistant shows error `403` | The token is valid but doesn't have permission to query the MCP. | Contact your administrator with this error code "403". |
| The assistant shows error `429` | You're querying the MCP faster than the platform allows. | Wait one minute, then retry. If this happens often, contact your administrator with this error code "429". |
| The assistant says it cannot find the person you mentioned | The name or email isn't in IDO under that spelling. | Double-check the name or email you typed. The name or email is case-insensitive. If your question is about an account, be sure to specify the target system containing the account. If your question is about an identity, try the HR employee id as an alternative. |
| The assistant returns names or values that look invented | The connection is fine but the model is guessing instead of calling the MCP. | Re-ask with a more precise reference (full email, full name, login). |

##### How to read the error code

The error code is a three-digit number (`401`, `403`, `429`, `503`, and similar) usually shown in the assistant's reply. If it's not visible, open the Cursor settings and select "Tools & MCPs." You will see a red dot with a button to "Show output" where you will find the error code.

![Cursor Installed MCP Servers panel with red dot on IDO and Show Output output panel](images/cursor-07-error-output.png)

If you cannot resolve the issue yourself, send the error code (and one example of the question you asked) to your administrator.

### In an automation tool (n8n)

#### 2.1 What this lets you do

With n8n you can build automated workflows around the MCP. The most common patterns are:

- **An internal chat panel** where colleagues ask questions about access risks without leaving your intranet.
- **A scheduled report:** for example, *"every Monday morning, produce a risk summary for the 10 new joiners of the week and post it to Slack."*
- **An enrichment step** plugged into your existing IGA pipelines: for example, *"after a new account is created, look up its risk profile and notify the owner if it lights up as privileged."*

The rest of this section walks you through the **simplest end-to-end flow**: an n8n chat interface backed by an AI agent that has the MCP tools at its disposal. Once you have that running, the more advanced patterns above are just additional nodes around the same agent.

#### 2.2 Requirements

Before opening n8n, request the following from your administrator:

| Item | Looks like | What it's for |
|---|---|---|
| **MCP URL** | `https://ido.example.com/acme/mcp/` | The endpoint that n8n will call to query IDO. |
| **Token endpoint** | `https://auth.example.com/auth/realms/acme/protocol/openid-connect/token` | Where n8n asks for a fresh access token before each batch of calls. |
| **Client ID** | A short string, e.g. `mcp-n8n-finance` | n8n identity when asking for tokens. |
| **Client Secret** | A long random string | n8n password when asking for tokens. Treat as sensitive. |

You will also need an **API key with an LLM provider** (OpenAI, Anthropic, Google Vertex AI, and similar). The MCP supplies the *tools*; the LLM provides the *reasoning*.

#### 2.3 Creating credentials on n8n

**Step 1.** Access your n8n instance, then click on the "Credentials" tab. Click the "Create credential" button at the top right.

![n8n Credentials tab with Create credential button highlighted](images/n8n-01-credentials-tab.png)

This will open a dialog box asking for the kind of token.

![Add new credential dialog with search field](images/n8n-02-add-credential-dialog.png)

**Step 2.** Enter "Bearer Auth" and select it. The result should look like the screenshot below. Click "Continue."

![Add new credential dialog with Bearer Auth selected](images/n8n-03-bearer-auth-selected.png)

**Step 3.** Put the Credential in "Expression" mode and paste `{{ $json.raw_token }}` in the Bearer token field. Then click "Save."

![Bearer Auth credential with expression mode and raw_token expression](images/n8n-04-bearer-expression.png)

**Step 4.** Go back to the "Credentials" tab. Click the "Create credential" button at the top right.

![Credentials tab again](images/n8n-05-credentials-tab-again.png)

**Step 5.** Enter the name of your LLM provider. In this example, we'll use "Mistral." Click "Continue."

![Add new credential dialog with provider search](images/n8n-06-add-llm-credential.png)

The result should look like the screenshot below:

![Add new credential with Mistral Cloud API selected](images/n8n-07-mistral-selected.png)

**Step 6.** Login with the account or the API Key from your LLM Provider and click "Save."

![Mistral Cloud credential configuration with API Key field](images/n8n-08-mistral-api-key.png)

You now have both credentials configured. You can continue.

![Credentials list showing Mistral Cloud and Bearer Auth accounts](images/n8n-09-both-credentials.png)

#### 2.4 Create a workflow

**Step 1.** Copy the JSON below.

```json
{
  "nodes": [
    {
      "parameters": {
        "method": "POST",
        "url": "<your token open id request url>",
        "sendBody": true,
        "contentType": "form-urlencoded",
        "bodyParameters": {
          "parameters": [
            {
              "name": "grant_type",
              "value": "client_credentials"
            },
            {
              "name": "client_id",
              "value": "<your-client-id>"
            },
            {
              "name": "client_secret",
              "value": "<your-client-secret>"
            }
          ]
        },
        "options": {
          "allowUnauthorizedCerts": true,
          "timeout": 10000
        }
      },
      "id": "f5a4343b-0fc5-413f-9ef8-0f02630dabea",
      "name": "Get Token",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [
        320,
        272
      ]
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "={{ $('When chat message received').item.json.chatInput }}",
        "hasOutputParser": true,
        "options": {
          "systemMessage": "Use your available MCP Tools. Start by fetching identities id, with identity id, go to identity context and retrieve them, then fetch account id, then account context.",
          "maxIterations": 10
        }
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3,
      "position": [
        864,
        272
      ],
      "id": "65997ef4-eb1d-4eb9-b5b8-1855743233fd",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "endpointUrl": "<mcp request url>",
        "authentication": "bearerAuth",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.mcpClientTool",
      "typeVersion": 1.2,
      "position": [
        1056,
        528
      ],
      "id": "44acd691-1a22-47d1-a8d1-c09ba846b380",
      "name": "MCP Client",
      "rewireOutputLogTo": "ai_tool",
      "credentials": {
        "httpBearerAuth": {
          "id": "jYyQAa7Jk2cNOvGK",
          "name": "Bearer Auth account"
        }
      }
    },
    {
      "parameters": {
        "options": {
          "allowFileUploads": false,
          "responseMode": "responseNodes"
        }
      },
      "type": "@n8n/n8n-nodes-langchain.chatTrigger",
      "typeVersion": 1.4,
      "position": [
        64,
        272
      ],
      "id": "c077d57e-12b6-4e14-b510-acb0131c10cc",
      "name": "When chat message received",
      "webhookId": "73ac790f-94d1-4dee-8e7c-6e3721ed5544"
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "f1e2d3c4-0001-4000-8000-aaaaaaaaaaaa",
              "name": "bearer_token",
              "value": "=Bearer {{ $json.access_token }}",
              "type": "string"
            },
            {
              "id": "f1e2d3c4-0002-4000-8000-bbbbbbbbbbbb",
              "name": "expires_in",
              "value": "={{ $json.expires_in }}",
              "type": "number"
            },
            {
              "id": "f1e2d3c4-0003-4000-8000-cccccccccccc",
              "name": "raw_token",
              "value": "={{ $json.access_token }}",
              "type": "string"
            }
          ]
        },
        "options": {}
      },
      "id": "eed66a37-9aa7-4a13-a1ff-884828670eff",
      "name": "Format Bearer Token",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [
        544,
        272
      ]
    },
    {
      "parameters": {
        "message": "={{ $json.output }}",
        "waitUserReply": false,
        "options": {
          "memoryConnection": false
        }
      },
      "type": "@n8n/n8n-nodes-langchain.chat",
      "typeVersion": 1,
      "position": [
        1376,
        272
      ],
      "id": "882ba150-efef-4b35-acd5-692647b6fe07",
      "name": "Respond to Chat"
    }
  ],
  "connections": {
    "Get Token": {
      "main": [
        [
          {
            "node": "Format Bearer Token",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Respond to Chat",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "MCP Client": {
      "ai_tool": [
        [
          {
            "node": "AI Agent",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "When chat message received": {
      "main": [
        [
          {
            "node": "Get Token",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Format Bearer Token": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "cd3acb52c7bad51f685dccfc81efd411e6db265d5ed0655b62470689e0850912"
  }
}
```

**Step 2.** In n8n, open a workflow and click Paste (Ctrl + V). The default workflow appears.

![n8n workflow canvas with pasted nodes (When chat message received, Get Token, Format Bearer Token, AI Agent, MCP Client, Respond to Chat)](images/n8n-10-workflow-canvas.png)

**Step 3.** Open node "Get Token" and replace:

- `<your token open id request url>` with the Token Endpoint you previously requested from your administrator.
- `<your-client-id>` with the Client ID given by your administrator.
- `<your-client-secret>` with the secret linked to the Client ID given by your administrator.

![Get Token node parameters with URL, client_id, client_secret fields highlighted](images/n8n-11-get-token-node.png)

**Step 4.** Open node "MCP Client" and replace:

- `<mcp request url>` with the MCP URL given by your administrator.

![MCP Client node parameters with Endpoint field highlighted](images/n8n-12-mcp-client-node.png)

The starter workflow ships without a chat model. The right node depends on which LLM provider you picked at Step 1.

**Step 5.** Click the "+" sign at the bottom of the AI Agent.

**Step 6.** Search for the right node. Pick the LLM credential you created at Step 1. In this example, it's "Mistral."

![Language Models picker with various chat model options](images/n8n-13-language-models-picker.png)

**Step 7.** A configuration menu opens. In the first dropdown, select the credential you created earlier. In the second dropdown, select the model you want to use.

![Chat model node configuration with credential and model dropdowns](images/n8n-14-chat-model-config.png)

**Step 8.** Save the workflow and click Activate (top right).

![Workflow Active toggle switched on](images/n8n-15-workflow-active.png)

#### 2.5 Try it out

**Step 1.** Click "Open Chat" next to the When chat message received node.

![When chat message received node with Open chat button](images/n8n-16-open-chat.png)

**Step 2.** Type a question about a person you know exists in IDO. For example: *"Give me a quick summary of <a.colleague@example.com>: manager, department, risk level."*

![n8n chat panel with example summary question](images/n8n-17-chat-question.png)

If everything is set up correctly, the agent replies with real IDO data about that person. If the assistant errors out or invents data, see Troubleshooting.

#### 2.6 Troubleshooting

| What you see | What's going on | What to do |
|---|---|---|
| The chat returns error `401` from the MCP | The token is not authorized to access the MCP Server, or it is invalid. | Contact your administrator with this error code "401". |
| The chat returns *"Permission denied"* or error `403` | The token is valid, but the underlying client doesn't have permission to query the MCP. | Contact your administrator with this error code "403". |
| No green dot appears around the MCP Client when executing | The agent answers without ever calling IDO. The MCP Client is not wired to the agent as a tool. | Open the **MCP Client** node. The connector to the AI Agent must be the `ai_tool` port (the bottom one, dotted). If not, delete and re-add the node directly under the AI Agent. n8n wires it as a tool automatically. |
| The chat panel stays empty after you send a message | The agent finished but **Respond to Chat** is not connected. | Make sure the **AI Agent's** main output is connected to the **Respond to Chat** node. |
| The agent invents data | The MCP tools are reachable, but the model isn't picking them. | Rephrase with a precise reference (full email or HR id). If it keeps happening, edit the *System Message* in the **AI Agent** node to insist on using the MCP tools. |
| Error `429` appears randomly | The workflow runs too fast for the rate limit. | Add a small delay between calls, or ask your administrator to raise the rate limit. |

For deeper inspection, every n8n node has an **Output** panel that shows the raw JSON exchanged with the MCP. That's the place to look for error details.

---

## Section 3: Technical setup in RadiantLogic IDO

This section is for **administrators** provisioning MCP access to end users.

### 3.1 Conventions and terminology

The placeholders below appear throughout sections 3 and 4. Replace them with the actual values of your platform before running any command.

| Placeholder | What it means | Example value |
|---|---|---|
| `<tenant>` | The logical isolation boundary that identifies your IDO deployment. In practice it is **both** the URL path segment that APISIX uses to route requests **and** the matching Keycloak realm name. By convention, the two share the same name. One IDO platform can host several tenants side by side (for example, one per customer, or one per environment). | `acme`, `acme-prod`, `acme-preprod` |
| `<app-external-dns>` | Public hostname of the IDO application gateway (APISIX). | `ido.example.com` |
| `<auth-external-dns>` | Public hostname of Keycloak (the identity provider used by IDO). | `auth.example.com` |
| `<client_id>` / `<client_secret>` | OIDC client credentials provisioned in Keycloak (steps below). One pair per end user or automation. | `mcp-john-doe`, `mcp-n8n-prod` |
| `<your-long-lived-token>` | Access token generated from a `<client_id>` / `<client_secret>` pair, carrying the `mcp-access` role. | A JWT string (~1.5 kB) |

### 3.2 Architecture in two lines

The MCP Server is a service deployed in the same cluster as the rest of IDO.

This section is a step-by-step tutorial for you to provision the OIDC client that generates those tokens, with the right role and lifespan.

> **Permissions warning.** These steps require **realm-admin** privileges on the tenant realm in Keycloak. On hardened production deployments, the customer's IT team may have removed this capability from line-of-business administrators. Verify your admin permissions before promising MCP access to a user. Our dev and preprod platforms are intentionally more permissive than locked-down production environments.

### 3.3 Open the Keycloak Admin Console

**Why:** every operation below happens in the Keycloak Admin Console, scoped to the tenant realm. There is no IDO-specific UI for this. Keycloak is the source of truth for who can call the MCP.

**Step 1.** Go to `https://<auth-external-dns>/auth/admin`.

**Step 2.** Sign in with a realm-admin account.

**Step 3.** Select the **tenant realm** in the top-left dropdown (this is the `<tenant>` value used in your URLs).

![Keycloak Admin Console with tenant realm selected in top-left dropdown](images/keycloak-01-realm-selector.png)

### 3.4 Create the OIDC client

**Why:** each end user (or each automation) gets its own client. The client is what generates tokens. One client per user is what makes revocation, auditing and TTL tuning workable later on.

**When:** do this once per user or automation, at onboarding time.

**Step 1.** Left menu → **Clients** → **Create client**.

![Clients list with Create client button](images/keycloak-02-clients-list.png)

**Step 2.** General Settings:

- **Client type:** `OpenID Connect`
- **Client ID:** a descriptive name reflecting **who** the client is for, for example `mcp-john-doe` (chat) or `mcp-n8n-finance` (automation).
- **Name / Description:** optional.

**Step 3.** Click **Next**.

**Step 4.** Capability config:

- **Client authentication:** `ON` (confidential client; the secret stays server-side).
- **Authentication flow:**
  - Direct access grants: `ON`
  - Service accounts roles: `ON`

![Capability config with Client authentication off and authentication flow defaults](images/keycloak-03-capability-off.png)

The result should look like this:

![Capability config with Client authentication ON and Direct access grants + Service accounts roles checked](images/keycloak-04-capability-on.png)

**Step 5.** Click **Next**, leave **Login settings** empty, then click **Save**.

**Step 6.** Open the **Credentials** tab and copy the **Client Secret**. This is what you will share with the end user (for n8n) or use yourself to generate a long-lived token (for chat).

![Credentials tab with Client Secret copy button highlighted](images/keycloak-05-client-secret.png)

### 3.5 Grant the `mcp-access` role

**Why:** the MCP middleware checks that incoming tokens carry the `mcp-access` role. Without it, any request is rejected with `403`, no matter how valid the token is otherwise.

**Step 1.** From the client detail page, open the **Service accounts roles** tab.

**Step 2.** Click **Assign role**.

![Service accounts roles tab with Assign role button highlighted](images/keycloak-06-service-roles-tab.png)

**Step 3.** Search for `mcp-access`, tick it, then click **Assign**.

![Assign Client roles dialog with mcp-access selected](images/keycloak-07-assign-mcp-access.png)

> **Note.** `mcp-access` is a composite role that already includes `user`, so the service account automatically gains the basic IDO read permissions it needs. There is no second role to assign.

### 3.6 Set the access token lifespan (TTL)

**Why:** this step determines how often the user has to refresh credentials. It is the single technical knob that distinguishes **chat mode** from **automation mode**. Different use cases need very different defaults.

**When:** change the default whenever you create a chat-mode client. Usually, keep the default for automation clients.

**Step 1.** Open the client's **Advanced** tab.

![Client detail with Advanced tab selected](images/keycloak-08-advanced-tab.png)

**Step 2.** Locate **Advanced Settings** → **Access Token Lifespan**.

![Advanced tab Jump to section with Advanced settings highlighted](images/keycloak-09-jump-to-advanced.png)

**Step 3.** Set the value based on the use case:

| Use case | Recommended TTL | Why |
|---|---|---|
| **Chat mode** (Cursor) | 30 days (or more, up to your security policy) | The end user pastes the raw access token into a config file. They cannot refresh it. A short TTL would force them to re-paste the token several times a day, which is unusable. |
| **Automation mode** (n8n, scripts) | 15 minutes (the realm default is usually 5 min; keep it short) | The automation tool re-requests a fresh token from the token endpoint every cycle. There is no usability cost to a short TTL, and it dramatically reduces the blast radius if a token leaks. |

**Step 4.** Click **Save**.

![Advanced settings with Access Token Lifespan set to 30 Days](images/keycloak-10-access-token-lifespan.png)

**Step 5.** For chat usage **ONLY**, you will also need to change the SSO Session Max timing to the same value via: Realm settings → **Sessions** → **SSO Session Max**.

**Step 6.** Click **Save**.

![Realm settings Sessions tab with SSO Session Max set to 30 Days](images/keycloak-11-sso-session-max.png)

> **Why one client per user?** Always create **one Keycloak client per end user** (not one shared client). Reasons:
>
> - **Revocation is granular:** when an employee leaves, you disable their client and the rest of your users keep working.
> - **Auditability:** Keycloak logs include the `client_id`, so you know which user or workflow made which call.
> - **TTL is per-client:** you can give a 30-day token to John (an analyst on chat) and a 15-minute token to the n8n service account, without compromising one for the other.

### 3.7 Generate a token (chat mode) and deliver the credentials

**Why:** in chat mode the user does not run OAuth themselves. They paste a token that you give them. You generate that token once, with the long TTL you set at step 4.

**Step 1.** Generate the token via `client_credentials`:

```bash
curl -X POST \
  "https://<auth-external-dns>/auth/realms/<tenant>/protocol/openid-connect/token" \
  -d "grant_type=client_credentials" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>" \
  | jq -r '.access_token'
```

You can now deliver this token to the user.

### 3.8 Revoke access

**Why:** people leave teams, credentials leak. You need a way to invalidate access without disrupting other users.

**When:** at offboarding, after a security incident, or when rotating credentials.

| Option | What it does | Use when |
|---|---|---|
| **Disable the client** (Clients → name → toggle *Enabled* OFF) | All existing tokens for this client stop working immediately. | Standard offboarding, one user. |
| **Remove the `mcp-access` role** (Service accounts roles → unassign) | Existing token remains cryptographically valid, but the MCP middleware rejects it with `403`. | You want to keep the client for other roles but block MCP. |
| **Not-Before policy** (Advanced → Set *Not Before* = now) | Every token issued before *now* becomes invalid; subsequent ones still work. | Suspected leak. You want to invalidate all in-flight tokens but keep the client running. |

For a fast token-level revocation (for example, one leaked token, with everything else still running):

```bash
curl -X POST \
  "https://<auth-external-dns>/auth/realms/<tenant>/protocol/openid-connect/revoke" \
  -d "token=$TOKEN" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>" \
  -d "token_type_hint=access_token"
```

---

## Section 4: Reference (tools and data)

This section is for **integrators** and anyone who needs to know exactly what the MCP returns, field by field.

### 4.1 What information the MCP can return

The MCP exposes two complementary views of your identity landscape: **accounts** (the technical objects living in your repositories) and **identities** (the people, aggregated across all their accounts).

**Accounts** — populated by `get_account_context`.

| Group of fields | What's in it |
|---|---|
| Identifiers | `id`, `account_id`, `login`, `samaccountname`, `dn`, `email`, `full_name`, `given_name`, `surname` |
| Lifecycle | `creation_date`, `last_modification_date`, `last_login`, `login_count`, `disabled`, `locked` |
| Password policy | `password_expired`, `password_not_required`, `password_cant_change`, `password_last_set_date`, `bad_password_date`, `dont_expire_password` |
| MFA | `mfa_active`, `mfa_allowed`, `mfa_required`, `mfa_registered`, `mfa_properly_configured`, `mfa_properly_configured_reason`, `default_mfa_method_type`, `default_mfa_method_strength`, `latest_mfa_method_type_used`, `latest_mfa_method_strength_used`, `secondary_mfa_method_types`, `secondary_mfa_method_strength` |
| SSPR / Passwordless | `is_sspr_allowed`, `is_sspr_registered`, `is_passwordless_active`, `is_passwordless_required` |
| Reconciliation | `reconciliation_type`, `reconciliation_rule`, `reconciliation_reliability` |
| Privilege | `privileged_account` |
| Relationships | `repository` (the system the account lives in), `owner` (the identity the account is reconciled to, with its HR data, manager, departments, and full list of accounts) |
| Authorization | `groups[]` (group membership), `permissions[]` (entitlements with linked resource) |
| Risk & quality | `risks[]` (aggregated and intrinsic risk levels and scores, sensitivity), `control_defects[]` (each defect carries the full control definition and an audit trail) |

**Identities** — populated by `get_identity_context`.

| Group of fields | What's in it |
|---|---|
| Identifiers | `id`, `hr_employee_id`, `full_name`, `given_name`, `surname`, `email` |
| Lifecycle | `arrival_date`, `departure_date`, `status`, `internal` |
| Org | `departments[]` (each with `department_short_name`, `identity_job_title`, `managers[]`) |
| Accounts | `accounts[]`: for each account, `id`, `groups[]`, `permissions[]`, `control_defects[]` |
| Risk | `risks[]`: same dual-axis structure as accounts (`agg_`, `int_`, `identity_nb_defects`, `sensitivity_level`) |
| Defects | `control_defects[]`: identity-level defects (for example, contractor with past ending date) |

**Reading the risk block.** IDO computes risk on two axes:

- `int_risk_*` (intrinsic): risk introduced by the account's own attributes (privileged, no MFA, dormant, and similar).
- `agg_risk_*` (aggregated): propagated from the resources and permissions the account holds.

`*_risk_level` is a bucket (1–4, higher = riskier). `*_risk_score` is a numeric breakdown. Use `*_risk_level` for everyday questions; use `*_risk_score` only when you need to sort.

### 4.2 Tools

The MCP Server exposes **4 tools** in GA. The descriptions, input parameters and output schemas below are exactly what the LLM sees through `tools/list`.

**Common conventions**

- All responses are wrapped in `{ "results": ..., "result_count": <int>, "status": "success" | "error", "error": "<optional>" }`.
- When a lookup finds no match, the call succeeds with `result_count: 0` and `results: []`. There is no dedicated *not found* error.
- Date fields that are not exposed by the source repository return a human-readable sentinel string (for example, `"Last login date not available in <repository_name>"`). Always check the value type before parsing.

#### 4.2.1 `fetch_account_id`

**Purpose:** find a unique account by login or email **inside a specific repository**. Designed to be called first, to obtain the `account_id` that `get_account_context` needs.

**Input parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `account_name` | string | yes | Account Name OR Account Login OR Account Email Address |
| `repository_name` | string | yes | Repository or System Name. Specifies which system to search (for example, `AD_CORP`, `HR`). |

**Output schema (top level)**

```json
{
  "results": [
    {
      "account_id": "string",
      "email": "string | null",
      "employee_number": "string | null",
      "full_name": "string | null",
      "login": "string | null",
      "repository": "string"
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

A login search (`"account_name": "eestrada"`) on the same repository returns the **same** record. Searching with a non-existing repository name returns `result_count: 0` (not an error).

#### 4.2.2 `fetch_identity_id`

**Purpose:** find a unique identity by HR id, full name, or corporate email. Designed to be called first, to obtain the `identity_id` that `get_identity_context` needs.

**Input parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `identity_name` | string | yes | HR Employee ID (matricule RH) OR Full Name (complete person name) OR Email Address |

**Output schema (top level)**

```json
{
  "results": [
    {
      "identity_id": "string",
      "email": "string",
      "full_name": "string",
      "hr_employee_id": "string"
    }
  ],
  "result_count": 0,
  "status": "success",
  "error": "string (optional)"
}
```

**Sample requests and responses**

Lookup by **email**:

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

The same result is returned by `"identity_name": "Lawrence Brown"` (full name) and `"identity_name": "E000065"` (HR id). A partial name (`"Brown"`) or a non-existing identity returns `result_count: 0`.

#### 4.2.3 `get_account_context`

**Purpose:** return the comprehensive context of an account: attributes, owner, repository, groups, permissions, risks, control defects. The main building block to answer questions like *"what does this account do, who owns it, is it risky?"*

**Input parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `account_id` | string | yes | Unique account identifier (obtained via `fetch_account_id`) |

**Output structure:** see Section 4.1 for the field-by-field listing of `results.account[0]`.

**Sample request**

```json
{
  "name": "get_account_context",
  "arguments": { "account_id": "029c789f3480fae1249ddaeed314f335" }
}
```

**Sample response** (excerpt; full payload is several kB)

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

#### 4.2.4 `get_identity_context`

**Purpose:** return the comprehensive context of an identity (the *person* view): HR attributes, manager and department, **every account the person owns** with their groups, permissions and defects, plus identity-level risks and defects.

**Input parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `identity_id` | string | yes | Unique identity identifier (obtained via `fetch_identity_id`) |

**Output structure:** see Section 4.1 for the field-by-field listing of `results.identity[0]`.

**Sample request**

```json
{
  "name": "get_identity_context",
  "arguments": { "identity_id": "1daf7bca9fe71c076b3778d2d0b29659" }
}
```

**Sample response** (excerpt; full payload is ~100 kB)

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

> **Tip for the LLM consumer.** When you ask *"compare two identities,"* the model needs both contexts in its window. Each context can be ~100 kB. If the model returns truncated reasoning, ask it to focus on a specific subject (risk profile only, or permissions only) so it can fit both responses in context.

### 4.3 Appendix: Manual curl validation

For troubleshooting outside of any client, the MCP protocol requires three calls:

```bash
TOKEN="<your access token>"
MCP_URL="https://<app-external-dns>/<tenant>/mcp/"

# 1) Initialize. Capture the Mcp-Session-Id header from the response.
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

# 4) Call a tool. Example: fetch_identity_id
curl -X POST "$MCP_URL" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"fetch_identity_id","arguments":{"identity_name":"lawrence.brown@example.com"}}}'
```

> For local or dev environments with self-signed certificates, add `-k` to every `curl` call. **Never use `-k` in production.**
