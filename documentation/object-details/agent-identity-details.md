## Overview

An agent identity represents a non-human AI agent that runs on a hosting platform and acts on resources on its own. Selecting an agent object link from any Identity Observability interface (such as Observations or Controls) opens a detailed page about that agent.

The details include the agent's identification and configuration, its lifecycle history, what it is made of, what it can reach, and who owns it. An example of the page is included below:

![The agent detail page, showing the page header with status pill and issue chips above the Details section](agent-detail-page.png)

The status next to the agent name shows the agent's current lifecycle status — for example "Active". The status changes if you quarantine or re-enable the agent.
Clicking the **Access Chain** button opens a data visualization interface where you can explore the access chain of the selected agent in the Explore menu.

## Page Header

The header carries the agent's identity, its current risk posture and the actions available on it.

| Element | Description |
| --- | --- |
| Entity title | The agent name, prefixed by the object type (`Agent Identity`). |
| Status | The agent's current lifecycle status. Updates in place when you quarantine or re-enable the agent. |
| Issue chips | The number of open issues on this agent, broken down by risk priority (Critical, High, Medium, Low), with a "View all Issues" link. |
| Access Chain | Opens the access-chain graph — the 360° view of everything the agent can reach. |
| Export JSON | Presents the agent's complete record as a JSON object, in a new page or a side panel. |
| Remediation menu | The overflow (…) menu listing the remediation actions available for this agent. |

## Key Agent Attributes

The first section of the agent detail page describes how the agent is identified and configured. The attributes are listed below.

| Attribute | Description |
| --- | --- |
| External ID | The agent's stable canonical identifier, as held by the platform of record. |
| Name | Human-readable name of the agent. |
| Description | What the agent does. Inline-editable. |
| Intent | The declared purpose the agent is authorized to pursue. Inline-editable. |
| Version | Agent version. |
| Tags | Identity observability tags. Manual tags are added by end-users; dynamic tags are derived by the platform and shown separately. |
| Sensitivity Level & Reason | Sensitivity classification of the agent, either defined by a data source or the end-user, with an explanation for the assigned level. |
| Agent Card URL | Where the agent's A2A card is published. Empty if the agent publishes none. |
| URLs | The agent's primary, health-check, metrics and invocation endpoints. |
| Provider | Owning organisation of the agent. |
| Platform | Hosting runtime the agent runs on (e.g., AWS Bedrock). |
| Runtime Identity | The non-human identity the agent executes as. |
| Agent Repository | The hosting account or environment, with a link to its repository detail page. |


## Provider and Lifecycle Sections

Below the Details section, two collapsible sections summarise where the agent comes from and its lifecycle details.

![The collapsed Provider and Lifecycle sections above the capability tab strip](agent-lifecycle-tabs.png)

### Provider

The provider section lists details of the organization that owns the agent. Expand the section, or click "View details", to see the full provider record.

### Lifecycle

| Attribute | Description |
| --- | --- |
| Status | The agent's current lifecycle status. |
| Status Reason | Why the agent is in its current status. |
| Status Changed At | When the status last changed. |
| Created At | When the agent record was created. |
| Published At | When the agent was published. |
| Suspended At | When the agent was suspended, if it has been. |
| Blocked At | When the agent was blocked, if it has been. |
| Deleted At | When the agent was deleted, if it has been. |
| Last Updated At | When the agent record was last updated. |
| Last Invoked At | When the agent last ran. |

## Capability Tabs

| Tab | Presentation | Shows |
| --- | --- | --- |
| Models | Cards | The foundation model behind the agent. |
| Guardrail | Table | The guardrails enforced on the agent. |
| Features | Cards | Each declared capability and what it means. |
| Skills | Cards | Each skill the agent advertises. |
| Tools | Table | Every adapter the agent can invoke. |
| Resources | Table | Every asset the agent can reach. |
| Subagents | Cards | Each agent this agent may invoke. |
| Agent Managers | Panel | The agent's owners. |


### Models

Each card shows the foundation model the agent runs on.

| Attribute | Description |
| --- | --- |
| Provider | The model provider (e.g., `AWS_BEDROCK`). |
| Model Identifier | The model ID, linked to its complete detail as a JSON object. |
| Version | Version of the model in use. |

### Guardrail

The Guardrail section is presented as a table, one row per guardrail applied to the agent.

| Attribute | Description |
| --- | --- |
| Identifier | Unique identifier of the guardrail. |
| Name | Display name of the guardrail. |
| Description | Purpose and function of the guardrail. |
| Status | Whether the guardrail is currently in force. |
| Enforcement Mode | How the guardrail is applied when it triggers. |
| Version | Version of the guardrail definition. |
| Created At / Created By | When the guardrail was created, and by which actor. |
| Updated At / Updated By | When the guardrail was last updated, and by which actor. |

### Features

Presented as cards, one per declared capability. Each card names the capability and explains what it means.

### Skills

Presented as cards, one per advertised skill.

| Attribute | Description |
| --- | --- |
| Identifier | Unique identifier of the skill. |
| Name | Display name of the skill. |
| Description | What the skill does. |
| Capability Tags | Tags describing the capabilities the skill exercises. |

### Tools

Presented as a table, one row per invokable adapter.

| Attribute | Description |
| --- | --- |
| Type | The kind of adapter. |
| Executor Reference | What executes the tool. |
| Schema Reference | The schema the tool's input and output conform to. |
| Target Resources | The resources the tool acts on. |
| Required Permissions | The permissions the tool needs to run. |
| Executing Principal | The principal the tool executes as. |
| Credential Reference | The credential the tool presents. |
| Credential Material Type | The kind of credential material used. |
| OAuth Configuration | The tool's OAuth settings, where it authenticates that way. |
| State | Whether the tool is currently available to the agent. |
| Created By / Updated By / Last Invoked By | The actors that created, last updated and last invoked the tool. |

### Resources

Presented as a table, one row per reachable asset.

| Attribute | Description |
| --- | --- |
| Resource Type | Category of resource. |
| Name | Display name of the resource, with a link to the resource detail page. |
| Location | Where the resource lives. |
| Owning Account | The account that owns the resource. |
| Access Level | The level of access the agent holds on the resource. |
| Access Grant Method | How that access is granted. |
| Policy Statements | The raw policy statements behind the grant. |
| Principal | The principal used to reach the resource. |
| Credential | The credential used to reach the resource. |
| Trust Chain | The chain of trust traversed to obtain the access. |
| Sensitivity Classification | Sensitivity classification of the resource. |
| Labels | Labels applied to the resource. |

### Subagents

Presented as cards, one per agent this agent may invoke.

| Attribute | Description |
| --- | --- |
| Identifier | Unique identifier of the subagent. |
| Agent Card URL | Where the subagent's A2A card is published. |
| Name | Display name of the subagent. |
| Description | What the subagent does. |

### Agent Managers

Presented as a panel listing the agent's owners. It mirrors the resource-managers block on resource detail pages. Multiple owners are supported, each with a role — business owner only in the current version. The panel is editable; see [Editing an Agent](#editing-an-agent).

## Editing Agent Details

The following attributes are editable. Everything else on the page is sourced from the platform of record and is read-only.

| Field | How to edit it |
| --- | --- |
| Description | Click the pencil icon next to the value. An inline text area opens with **Save** and **Cancel**. Saving records an audit event. |
| Intent | Click the pencil icon next to the value. An inline text area opens with **Save** and **Cancel**. Saving records an audit event. |
| Sensitivity Level | Click the pencil icon next to the value. An inline text area opens with **Save** and **Cancel**. Saving records an audit event. |
| Tags | A chip-strip editor, with auto-completion from existing tag values. Reserved prefixes such as `env:`, `owner:` and `costCenter:` are preserved. Only manual tags are editable; dynamic tags are read-only and shown with a lock indicator. |
| Agent Managers | Click **+ Add Manager** to open a side panel identical to the platform's account-owner selection panel, with a suggested list and search by name, email or HR code, and **Cancel** / **Assign Identity** actions. Remove an owner with the **X** on their chip; removing the last remaining owner asks for confirmation. |

## Available Actions

Users can perform the following actions in the agent detail interface:

- Edit the agent's description, intent and sensitivity level inline.
- Add or remove manual tags on the agent.
- Assign or remove agent managers.
- Quarantine or re-enable the agent.
- Fix issues related to the agent through the remediation menu.
- Explore the agent's access chain through the **Access Chain** button.
- Export the agent's complete record as a JSON object.
