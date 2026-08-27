# Key concepts for AI agents

RadiantOne's Observability for Agentic AI enables teams to apply established identity governance practices such as ownership assignment, access analysis, lifecycle tracking, tagging, controls, and remediation to AI agents alongside human and other non-human identities.

Use this document to understand the terms and key concepts related to Observability for Agentic AI.

## Key terms

| Term | Definition |
| --- | --- |
| Agentic AI | AI systems that can reason through multi-step tasks, use tools, and perform actions with varying levels of autonomy. |
| Agent | An AI agent discovered from a supported platform and represented in RadiantOne Observability as an agent identity. Each platform-native agent is represented by one agent record. |
| Agent identity | The platform record that represents an agent. It includes core agent details and structured information about its model, guardrails, capabilities, tools, resources, subagents, and access path. |
| Runtime identity | The non-human identity used by an agent when it runs, such as an IAM role, service account, managed identity, or service principal. This identifies what the agent acts as, rather than what the agent is. |
| Repository | The hosting scope where an agent is discovered, such as a cloud account, project, subscription, or platform environment. |
| Tool | A service, API, function, workflow, connector, knowledge base, or other adapter that an agent can invoke. |
| Guardrail | A policy that evaluates an agent's input or output. It can block a request, allow it and log the event, or send it for human review. |
| Skill | A named capability that an agent advertises to other agents. |
| Subagent | Another agent that an agent can invoke to complete part of a task. |
| Permission flow | A visual representation of the identities, policies, roles, groups, and resources involved when an agent accesses an asset. |
| Access chain | The end-to-end relationship path around an agent: owner → agent → runtime identity → permissions → resources and subagents. |
| Blast radius | The resources, privileges, and downstream agents that may be affected if an agent or its owner is misconfigured, compromised, or misused. |
| MCP | A protocol used to connect agents with tools and resource servers. |
| A2A | A protocol that supports task delegation and collaboration between agents. |
| Agent Card | A document an agent can publish to describe its capabilities and how other agents can interact with it. |
| Observation | A continuously updated list of entities that match defined criteria. Observations can support dynamic tag assignment. |
| Control | A defined risk check that evaluates agents or related entities against a condition and provides a risk level and remediation guidance. |
| Control defect | An instance where a control identifies a risk condition for an entity. Also called an issue. |
| Tag | A governance label applied to an object. Tags can be assigned manually or dynamically through rules. |
| Backend label | A provider-native label, such as a cloud resource tag, imported for visibility and filtering. Backend labels are not RadiantOne tags. |
| Quarantine | An authorized remediation action that marks an agent for restriction or isolation in the source platform. |

## Agent inventory

RadiantOne Observability normalizes agents from supported platforms into a shared canonical agent model. This allows teams to use the same dashboards, controls, tags, and reports across platforms.

Each agent record contains core identity, hosting, lifecycle, governance, and runtime information. It also includes structured objects that describe what the agent uses, what it can do, and what it can access.

Platform-specific information is retained in metadata fields. This preserves provider detail without changing the common agent model.

### Agent record components

| Component | Quantity | What it describes |
| --- | --- | --- |
| Agent | One | Agent identity, hosting context, lifecycle, governance, and runtime posture |
| Model | One | The foundation model used by the agent |
| Guardrails | Many | Policies applied to input and output |
| Features | Many | Behavioral, safety, and protocol-related properties |
| Skills | Many | Advertised capabilities |
| Tools | Many | Services, functions, APIs, and other adapters the agent can invoke |
| Resources | Many | Assets the agent can access |
| Subagents | Many | Other agents the agent can invoke |
| Permission flow | One graph | The access path from the agent to reachable resources |

An agent does not have a separate top-level role object. Its execution role is represented by its runtime identity and by the related nodes in its permission flow.

## Agent details

The agent record contains the attributes needed to identify, govern, and investigate an agent.

| Attribute | Description |
| --- | --- |
| `id` | Internal identifier assigned by RadiantOne Observability |
| `external_id` | Stable identifier from the source platform, used to identify and correlate the agent |
| `name` | Human-readable agent name |
| `description` | Description of what the agent does. This can be edited in RadiantOne Observability. |
| `intent` | The agent's business purpose. This is separate from its technical description and can be edited. |
| `version` | Agent version provided by the source platform |
| `agent_card_url` | Location of the agent's published Agent Card, when available |
| `url` | Agent endpoints, such as primary, health, metrics, and invocation endpoints |
| `provider` | Organization or platform associated with the agent |
| `platform` | Hosting platform or runtime classification |
| `tags` | Governance labels assigned to the agent |
| `status` | Current lifecycle status |
| `status_reason` | Explanation for the current status |
| `action_quarantined` | Control that requests quarantine through the source-platform connector |
| `runtime_identity` | Non-human identity used to execute the agent |
| Lifecycle details | Creation, publication, suspension, blocking, deletion, update, and invocation information |
| `metadata` | Provider-specific information that does not alter the canonical model |

Lifecycle events include both a timestamp and an actor, where available. For example, `published_at` records when the agent was published and `published_by` identifies who performed the action.

## Models and guardrails

### Models

The model object identifies the foundation model used by an agent.

| Attribute | Description |
| --- | --- |
| `provider` | Model provider |
| `modelId` | Provider-defined model identifier |
| `version` | Specific model version, when available |
| `metadata` | Model hosting, endpoint, region, credential reference, supported capabilities, and other provider-specific details |

An agent that uses a floating model alias rather than a pinned version can create change-management and governance risk. This condition can be evaluated by a control.

### Guardrails

Guardrails define how the agent responds when input or output matches a policy condition.

| Enforcement mode | Behavior |
| --- | --- |
| block | Prevents the request or response from proceeding |
| allow with log | Permits the activity but records it for review |
| HITL | Routes the activity to a human reviewer before it proceeds |

Guardrail information can include its name, description, version, enabled status, lifecycle details, content filters, redaction rules, blocked topics, and sensitive-data policies.

allow with log provides visibility but does not stop an action. block and HITL provide an enforcement point.

## Capabilities and access

### Features and skills

Features describe agent behavior, safety configuration, and supported protocols. Examples include streaming, knowledge access, tool execution, guardrail support, A2A support, and MCP support.

Skills are named capabilities an agent advertises to other agents. A skill can include a name, description, tags, supported input and output modes, parameters, response schema, authentication requirements, and references to the tools that support it.

### Tools

Tools enable an agent to take action outside its core model. A tool can be an API, function, data store, cloud function, MCP server, connector, workflow, knowledge base, or return-control mechanism.

Tool information can include:

- Tool name, description, and type
- Execution endpoint or code reference
- API schema or invocation contract
- Required permissions or scopes
- Target resources affected by the tool
- Runtime identity and credential reference
- OAuth configuration, where applicable
- Tool state, such as ENABLED, DISABLED, or DRAFT

Credential references identify the credential used by a tool without exposing secret material.

### Resources

Resources are external assets an agent can access. They can include applications, APIs, data stores, files, cloud services, knowledge bases, and other platform resources.

| Attribute | Description |
| --- | --- |
| `resourceExternalId` | Provider-specific resource identifier |
| `resourceType` | Type of resource |
| `displayName`, `description` | Resource name and purpose |
| `location` | Region, environment, or other location information |
| `ownerAccount` | Account, subscription, project, tenant, or environment that owns the resource |
| `accessLevel` | Effective access, such as READ, WRITE, FULL, RETRIEVE, INVOKE, PUBLISH, or SUBSCRIBE |
| `grantedThrough` | Mechanism that grants access, such as a role, service account, managed identity, or policy |
| `policyStatements` | Source policies that grant access |
| `principalId`, `principalType` | Identity used to access the resource |
| `credentialRef` | Credential reference, without the secret value |
| `trustChain` | Role-assumption, impersonation, or federation path used to reach the resource |
| `sensitivity` | Data classification: PUBLIC, INTERNAL, CONFIDENTIAL, or RESTRICTED |
| `tags` | Resource ownership, environment, cost center, and other labels |

Resource sensitivity is an important factor in risk assessment. Access to CONFIDENTIAL or RESTRICTED resources can increase the severity of a finding. Missing sensitivity classification can also be identified as a governance gap.

### Subagents

Subagents are agents that another agent can call as part of completing a task. They may be external collaborators, peer agents, delegated workers, or internally orchestrated child agents.

Subagent information can include:

- The source-platform identifier and name
- The published Agent Card location, when available
- The hosting platform or provider
- Relationship type, such as delegate, supervisor, peer, or toolLike
- Invocation protocol, such as A2A, MCP, HTTP, or a provider-native protocol
- Authentication method and delegation mode
- Data-sharing policy
- Invocation endpoint

## Permission flow and risk context

Permission flow shows how an agent reaches a resource. It captures the connected identities, roles, groups, policies, credentials, and resources that form the agent's access path.

For example, a permission flow might show:

Business owner → Agent → Runtime service account → IAM role → Data policy → Customer data store

This view helps teams investigate why an agent has access and understand the potential impact of a misconfiguration or compromise.

A permission flow contains one agent node and directed relationships between related objects. Node identifiers remain stable across scans so teams can identify meaningful changes over time. The flow should not contain cycles; if it does, the platform identifies the cycle as an anomaly.

### Access chain and blast radius

The access chain provides the full context around an agent:

Owner → Agent → Runtime identity → Permissions → Resources → Subagents

The blast radius is the set of assets, permissions, and downstream agents reachable through that chain. Teams can use the agent's graph view to investigate:

- Which sensitive resources an agent can access
- Which permissions enable that access
- Whether multiple agents share the same runtime identity
- Which downstream agents can receive delegated tasks or data
- How ownership or runtime-identity changes could affect access

## Identity, ownership, and repositories

Each discovered platform-native agent is represented by one agent record within its repository. The platform uses the provider's external identifier to avoid duplicates when an agent is observed again.

### Ownership

Ownership connects an agent to the people and organizational context responsible for it. An agent can have multiple owners with different responsibilities, including a business owner and a technical owner.

Assign owners to establish accountability for the agent's purpose, operation, access, risk remediation, and lifecycle decisions. Owner selection uses the standard identity selection experience and supports searches by name, email address, or HR code.

### Runtime identity

The runtime identity is the non-human identity used when an agent performs work. Depending on the platform, it can be an IAM role, service account, managed identity, or service principal.

The relationship between an agent and its runtime identity supports important risk checks:

- An agent without a dedicated runtime identity can be difficult to attribute, restrict, or revoke independently.
- Multiple agents using the same runtime identity share the same access path and can create a larger blast radius if that identity is compromised.

### Repositories

A repository represents the hosting scope where an agent is discovered.

| Platform | Repository scope |
| --- | --- |
| AWS | Account |
| Google Cloud | Project |
| Azure | Subscription |
| Copilot Studio | Environment |

Repository linkage is created automatically from connector data. A repository can contain accounts, groups, and agents, so existing Repository Manager access applies to agent data within the repositories a user manages.

## Governance and remediation

### Local edits

Some agent attributes can be updated locally in RadiantOne Observability to improve governance context.

| Locally editable attribute | Use |
| --- | --- |
| description | Clarify what the agent does |
| intent | Record the business purpose |
| tags | Classify the agent for governance and reporting |
| Sensitivity | Record or refine data-sensitivity context |
| Managers | Assign or update business and technical owners |

Local edits are stored in RadiantOne Observability and are not written back to the source platform. A locally updated description takes precedence over the provider-sourced description for governance purposes.

### Quarantine

Quarantine is the supported remediation action that can be written back to the source platform. It is disabled by default and requires authorized action.

Setting `action_quarantined` requests that the connector apply the platform-specific quarantine mechanism. Clearing it requests release from quarantine. When quarantine is active, the agent status is QUARANTINED, and the prior lifecycle state is retained in `status_reason`.

The available quarantine behavior depends on the connector and source platform. For AWS, quarantine can be recorded through a marker tag and can optionally include stronger restrictions, such as disabling invocation or applying a deny policy.

### Controls and control defects

Controls continuously evaluate agents and related entities against defined risk conditions. A control includes a risk level, risk description, and remediation guidance.

When a control identifies a matching condition, it creates a control defect, also called an issue. A defect records the affected entity, risk level, current state, audit history, and any approved exception.

### Tags and backend labels

Tags are governance labels used to organize agents, apply policy, and support reporting.

| Label type | Description |
| --- | --- |
| Manual tag | Assigned by an administrator |
| Dynamic tag | Assigned automatically based on rule criteria and read-only to users |
| Backend label | Source-platform label imported for visibility and filtering |

Backend labels, such as cloud resource tags, are displayed in the agent's provider details and can be searched and filtered by key or value. In RadiantOne, they are displayed as read-only values.

## Lifecycle and status

An agent's status indicates whether it is operational, restricted, pending retirement, or removed.

| Status | Meaning |
| --- | --- |
| CREATED | The agent exists but is not yet deployed |
| PUBLISHED | The agent is live and deployable |
| SUSPENDED | The agent is disabled or suspended |
| BLOCKED | The agent is blocked, such as by a policy |
| QUARANTINED | A quarantine action is active |
| DEPRECATED | The agent is pending retirement |
| DELETED | The agent has been soft deleted |

Status changes record when the change occurred and, where available, the identity that performed it. This history helps teams determine whether an agent is active, who changed its state, and whether it remains subject to quarantine.
