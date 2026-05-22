# Using MCP server with your AI tools

The MCP (Model Context Protocol) Server lets your AI assistant (Cursor, n8n, and others) answer questions about people and accounts in your IDO catalog in plain English. No code or query needed.

If you administer or integrate the platform, see the companion **MCP Server: Administrator and Integrator Guide** for setup steps and the tools reference.

## How It Works

1. **Get credentials from your administrator.** Ask for an MCP URL and either an access token (for chat apps like Cursor) or a client ID, client secret, and token endpoint (for automation tools like n8n).
2. **Configure your tool.** Paste the URL and credentials into Cursor's `mcp.json` file, or wire them into an n8n workflow with the template provided below.
3. **Ask a question.** Mention a real person or account by name, email, or login. The assistant calls the MCP tools, retrieves the data, and replies in plain English.

## What You Can Ask About

You can ask about any **person** (an identity) and any **account** they own. For each, the assistant can tell you:

* **Who they are.** Name, email, employee ID, job title, manager, department, arrival and departure dates.
* **What they can access.** Group membership, permissions, resources, and whether the account is privileged.
* **Their associated risks.** Risk level and score, MFA status, account state (active, disabled, locked), and how the account was reconciled to the person.

A question must include **at least one identity, account, or resource name**, such as a person's name, an email, a login, or a repository. Open ended queries like *"show me everything risky"* will not work; the assistant only answers when you point it at a precise target.

**Sample questions**

* *"What resources does Lawrence Brown have access to?"*
* *"What are the risks associated with `lawrence.brown@example.com`?"*
* *"Is the account `eestrada` in repository `AD_CORP` a privileged account?"*
* *"Compare the risk profiles of Lawrence Brown and Evelyn Estrada and tell me who is most exposed, and why."*
* *"Is Lawrence Brown a contractor whose departure date has passed? If yes, what active accounts does he still have?"*

## Table of Contents

* [Using MCP in Cursor](#using-mcp-in-cursor)
* [Using MCP in n8n](#using-mcp-in-n8n)

## Using MCP in Cursor

Connecting the MCP to your chat app lets you query IDO from your IDE. Common use cases: ad hoc investigation, conversational drilling on risk, alert triage, and side by side identity comparisons.

### 1. Get Two Things From Your Administrator

| Item | Looks like |
| --- | --- |
| **MCP URL** | An HTTPS link ending with `/mcp/`, for example `https://ido.example.com/acme/mcp/` |
| **Access token** | A long string of letters and numbers. Treat it like a password. |

The token is personal. Do not share it with colleagues.

### 2. Configure Cursor

1. Click **File → Preferences → Cursor Settings**.

   ![Cursor File menu path File → Preferences → Cursor Settings](./mcp-doc-screenshots/screenshot-000.png)

2. Click **Tools & MCPs**.

   ![Cursor Settings sidebar with Tools & MCPs highlighted](./mcp-doc-screenshots/screenshot-001.png)

3. Click **Add Custom MCP** to open the `mcp.json` file.

   ![Add Custom MCP button in the Tools & MCPs panel](./mcp-doc-screenshots/screenshot-002.png)

   ![Default empty mcp.json](./mcp-doc-screenshots/screenshot-003.png)

4. Replace the contents with:

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

   The result:

   ![Filled mcp.json with the IDO server URL and bearer token](./mcp-doc-screenshots/screenshot-004.png)

5. Restart Cursor. The IDO server now appears as **Connected**.

   ![Cursor settings showing the IDO MCP server connected with its tools listed](./mcp-doc-screenshots/screenshot-005.png)

### 3. Try It Out

Type a question about your own name or email. For example:

* *"Give me a summary of `<your.email@example.com>`."* fetches an identity.
* *"Who is the owner of `<your.email@example.com>` on service ACME?"* fetches an account.

If everything is configured correctly, the assistant returns a readable answer with real data (name, manager, groups, risks). If you see *"I don't have access to that information"*, an error code, or hallucinated content, jump to [Troubleshooting](#4-troubleshooting).

### 4. Troubleshooting

| What you see | What's going on | What to do |
| --- | --- | --- |
| No MCP server appears in the app's settings | Config file is in the wrong place, has a typo, or the app didn't pick it up. | Re-open the config file. Check the exact path. Verify URL and token were copied without extra spaces. Restart the app. |
| *"I don't have access to that information"* or error 401 | The token is missing, expired, or invalid. | Confirm the token was pasted in full (not truncated). If correct, contact your administrator with error code `401`. |
| Error 403 | The token is valid but lacks permission to query the MCP. | Contact your administrator with error code `403`. |
| Error 429 | Querying the MCP faster than the platform allows. | Wait one minute, retry. If frequent, contact your administrator with error code `429`. |
| Assistant cannot find the person you mentioned | The name or email is not in IDO under that spelling. | Double check the spelling (case insensitive). For accounts, specify the target system. For identities, try the HR employee ID. |
| Assistant returns invented names or values | Connection is fine; the model is guessing instead of calling the MCP. | Re-ask with a precise reference (full email, full name, login). |

**How to find the error code.** The error code is a three digit number (`401`, `403`, `429`, `503`...) usually shown in the assistant's reply. If it is not visible, open Cursor settings, select **Tools & MCPs**, and look for the red dot with a **Show output** button.

![Tools & MCPs panel showing the IDO server with an Error indicator and a Show Output link](./mcp-doc-screenshots/screenshot-006.png)

If you cannot resolve the issue yourself, send the error code (and one example of the question you asked) to your administrator.


## Using MCP in n8n

With n8n you can build automated workflows around the MCP: an internal chat panel, a scheduled risk summary posted to Slack, or an enrichment step in your existing IGA pipelines. The rest of this section walks through the simplest end to end flow: a chat interface backed by an AI agent that has the MCP tools at its disposal.

### 1. Get Four Things From Your Administrator

| Item | Looks like | What it's for |
| --- | --- | --- |
| **MCP URL** | `https://ido.example.com/acme/mcp/` | The endpoint n8n calls to query IDO. |
| **Token endpoint** | `https://auth.example.com/auth/realms/acme/protocol/openid-connect/token` | Where n8n asks for a fresh access token before each batch of calls. |
| **Client ID** | A short string, for example `mcp-n8n-finance` | The n8n identity when asking for tokens. |
| **Client Secret** | A long random string | The n8n password when asking for tokens. Treat as sensitive. |

You also need an **API key with an LLM provider** (OpenAI, Anthropic, Google Vertex AI, and so on). The MCP supplies the *tools*; the LLM provides the *reasoning*.

### 2. Create Credentials in n8n

1. In your n8n instance, click the **Credentials** tab, then **Create credential** (top right).

   ![n8n Credentials tab with the Create credential button highlighted](./mcp-doc-screenshots/screenshot-007.jpg)

   A dialog opens.

   ![Add new credential dialog with an empty Search for app field](./mcp-doc-screenshots/screenshot-008.png)

2. Enter **Bearer Auth**, select it, click **Continue**.

   ![Add new credential dialog with Bearer Auth selected](./mcp-doc-screenshots/screenshot-009.png)

3. Switch the credential to **Expression** mode, paste `{{ $json.raw_token }}` into **Bearer Token**, and click **Save**.

   ![Bearer Auth credential editor with the Bearer Token field set to {{ $json.raw_token }} in Expression mode](./mcp-doc-screenshots/screenshot-010.png)

4. Go back to the **Credentials** tab and click **Create credential** again.

   ![n8n Credentials tab with the Create credential button highlighted](./mcp-doc-screenshots/screenshot-011.jpg)

5. Enter your LLM provider name (e.g. **Mistral**) and click **Continue**.

   ![Add new credential dialog ready for the LLM provider name](./mcp-doc-screenshots/screenshot-012.png)

   ![Add new credential dialog with Mistral Cloud API selected](./mcp-doc-screenshots/screenshot-013.png)

6. Log in or paste the API key from your LLM provider, then click **Save**.

   ![Mistral Cloud API credential editor with the API Key field highlighted](./mcp-doc-screenshots/screenshot-014.png)

Both credentials are now configured.

![Credentials list with Mistral Cloud account and Bearer Auth account](./mcp-doc-screenshots/screenshot-015.png)

### 3. Create the Workflow

1. Copy the JSON below.

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
               { "name": "grant_type", "value": "client_credentials" },
               { "name": "client_id", "value": "<your-client-id>" },
               { "name": "client_secret", "value": "<your-client-secret>" }
             ]
           },
           "options": { "allowUnauthorizedCerts": true, "timeout": 10000 }
         },
         "id": "f5a4343b-0fc5-413f-9ef8-0f02630dabea",
         "name": "Get Token",
         "type": "n8n-nodes-base.httpRequest",
         "typeVersion": 4.2,
         "position": [320, 272]
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
         "position": [864, 272],
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
         "position": [1056, 528],
         "id": "44acd691-1a22-47d1-a8d1-c09ba846b380",
         "name": "MCP Client",
         "rewireOutputLogTo": "ai_tool",
         "credentials": {
           "httpBearerAuth": { "id": "jYyQAa7Jk2cNOvGK", "name": "Bearer Auth account" }
         }
       },
       {
         "parameters": {
           "options": { "allowFileUploads": false, "responseMode": "responseNodes" }
         },
         "type": "@n8n/n8n-nodes-langchain.chatTrigger",
         "typeVersion": 1.4,
         "position": [64, 272],
         "id": "c077d57e-12b6-4e14-b510-acb0131c10cc",
         "name": "When chat message received",
         "webhookId": "73ac790f-94d1-4dee-8e7c-6e3721ed5544"
       },
       {
         "parameters": {
           "assignments": {
             "assignments": [
               { "id": "f1e2d3c4-0001-4000-8000-aaaaaaaaaaaa", "name": "bearer_token", "value": "=Bearer {{ $json.access_token }}", "type": "string" },
               { "id": "f1e2d3c4-0002-4000-8000-bbbbbbbbbbbb", "name": "expires_in", "value": "={{ $json.expires_in }}", "type": "number" },
               { "id": "f1e2d3c4-0003-4000-8000-cccccccccccc", "name": "raw_token", "value": "={{ $json.access_token }}", "type": "string" }
             ]
           },
           "options": {}
         },
         "id": "eed66a37-9aa7-4a13-a1ff-884828670eff",
         "name": "Format Bearer Token",
         "type": "n8n-nodes-base.set",
         "typeVersion": 3.4,
         "position": [544, 272]
       },
       {
         "parameters": {
           "message": "={{ $json.output }}",
           "waitUserReply": false,
           "options": { "memoryConnection": false }
         },
         "type": "@n8n/n8n-nodes-langchain.chat",
         "typeVersion": 1,
         "position": [1376, 272],
         "id": "882ba150-efef-4b35-acd5-692647b6fe07",
         "name": "Respond to Chat"
       }
     ],
     "connections": {
       "Get Token": { "main": [[{ "node": "Format Bearer Token", "type": "main", "index": 0 }]] },
       "AI Agent": { "main": [[{ "node": "Respond to Chat", "type": "main", "index": 0 }]] },
       "MCP Client": { "ai_tool": [[{ "node": "AI Agent", "type": "ai_tool", "index": 0 }]] },
       "When chat message received": { "main": [[{ "node": "Get Token", "type": "main", "index": 0 }]] },
       "Format Bearer Token": { "main": [[{ "node": "AI Agent", "type": "main", "index": 0 }]] }
     },
     "pinData": {},
     "meta": { "templateCredsSetupCompleted": true, "instanceId": "cd3acb52c7bad51f685dccfc81efd411e6db265d5ed0655b62470689e0850912" }
   }
   ```

2. In n8n, open a workflow and paste (Ctrl+V). The default workflow appears.

   ![n8n workflow: When chat message received → Get Token → Format Bearer Token → AI Agent → Respond to Chat, plus MCP Client attached to AI Agent](./mcp-doc-screenshots/screenshot-016.png)

3. Open the **Get Token** node and replace:

   * `<your token open id request url>` with the **Token Endpoint** from your administrator.
   * `<your-client-id>` with the **Client ID**.
   * `<your-client-secret>` with the **Client Secret**.

   ![Get Token node parameters with URL, client_id, and client_secret fields highlighted](./mcp-doc-screenshots/screenshot-017.png)

4. Open the **MCP Client** node and replace `<mcp request url>` with the **MCP URL**.

   ![MCP Client node parameters with the Endpoint field highlighted](./mcp-doc-screenshots/screenshot-018.png)

5. Click **+** at the bottom of the **AI Agent** node, then search for and pick your LLM credential (here, **Mistral**).

   ![n8n Language Models node picker showing Mistral Cloud Chat Model and others](./mcp-doc-screenshots/screenshot-019.png)

6. In the dropdowns, select the credential you created and the model to use.

   ![Mistral Cloud Chat Model node parameters with Credential and Model dropdowns highlighted](./mcp-doc-screenshots/screenshot-020.png)

7. Save the workflow and click **Activate** (top right).

   ![n8n workflow header with the Active toggle switched on](./mcp-doc-screenshots/screenshot-021.png)

### 4. Try It Out

1. Click **Open Chat** next to the **When chat message received** node.

   ![n8n canvas with the Open Chat button highlighted](./mcp-doc-screenshots/screenshot-022.png)

2. Ask a question about a person you know exists in IDO. For example: *"Give me a quick summary of `<a.colleague@example.com>`, manager, department, risk level."*

   ![n8n chat panel with the example prompt typed in the input box](./mcp-doc-screenshots/screenshot-023.png)

If the agent errors out or invents data, see the troubleshooting table below.

### 5. Troubleshooting

| What you see | What's going on | What to do |
| --- | --- | --- |
| Chat returns error 401 from the MCP | Token not authorized or invalid. | Contact your administrator with error code `401`. |
| *"Permission denied"* or error 403 | Token is valid but the client lacks MCP permission. | Contact your administrator with error code `403`. |
| No green dot around **MCP Client** when executing | The MCP Client is not wired to the AI Agent as a tool. | Open **MCP Client**. The connector to the AI Agent must use the `ai_tool` port (bottom, dotted). If wrong, delete and re-add the node directly under the AI Agent; n8n wires it as a tool automatically. |
| Chat panel stays empty after sending a message | **Respond to Chat** is not connected. | Connect the **AI Agent**'s main output to the **Respond to Chat** node. |
| Agent invents data | Tools are reachable, but the model is not picking them. | Rephrase with a precise reference (full email or HR ID). If it persists, edit the *System Message* in the **AI Agent** node to insist on using the MCP tools. |
| Error 429 appears randomly | Workflow runs faster than the rate limit. | Add a small delay between calls, or ask your administrator to raise the rate limit. |

Every n8n node has an **Output** panel that shows the raw JSON exchanged with the MCP. That is the place to look for deeper details when investigating an error.
