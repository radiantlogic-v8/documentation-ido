# Overview

Radiant Logic's Identity Data Platform offers observability for Agentic AI which gives security and identity teams a continuously updated view of AI agents across the enterprise: what exists, who owns it, how it runs, what it can access, and where it creates risk.

The capability treats agents as identities. It inventories agents across supported platforms, correlates them with their human owners and runtime non-human identities, evaluates them against security and governance controls, and manages them through existing dashboards, workflows, tagging, and remediation processes.

It helps teams answer questions such as:

- Which AI agents exist, including unmanaged or unreported agents?
- Which models, guardrails, tools, and data sources does each agent use?
- Who owns each agent, and which agents are orphaned?
- Which inactive agents retain access to sensitive resources?
- What is the organization's agent risk exposure by platform, department, or environment?

## The challenge

AI agents are created across cloud platforms, development environments, low-code tools, and business-managed applications. Each platform maintains its own inventory, metadata, lifecycle terminology, and access model.

This fragmentation prevents organizations from maintaining a complete, consistent view of agent ownership, access, lifecycle, and risk.

| Driver | Consequence |
| --- | --- |
| Scattered registries | No single source of truth for enterprise agents |
| Limited discovery | Manual tracking allows shadow AI to proliferate |
| Inconsistent metadata | Ownership, purpose, and criticality are incomplete or inconsistent |
| Low-code and no-code adoption | Agents are created faster than central teams can govern them |
| Protocol fragmentation | Agent capabilities are represented differently across platforms |
| Missing governance | Agents may lack owners, review cycles, or lifecycle controls |
| Audit requirements | Teams cannot readily identify agents with access to sensitive or regulated data |

The Radiant Logic Identity Data Platform addresses this fragmentation by bringing agent inventory, ownership, access relationships, risk posture, and governance into a single enterprise model.

## What differentiates the approach

### Connected identity context

Agents are not managed as isolated objects. Each agent is correlated with its runtime identity, human owners, accessible resources, and the access paths that connect them.

### Canonical agent model

Agents from different platforms are normalized into a common schema. Platform-specific details remain available in structured extension fields, while controls, tags, dashboards, and reports operate consistently across environments.

### Security posture management

Agents are evaluated against defined security and governance controls. Findings identify gaps in ownership, lifecycle management, access exposure, configuration, and operational posture.

### Scalable governance

Dynamic, rule-driven tags apply controls to groups of agents based on attributes such as environment, department, data sensitivity, ownership status, or activity level. This enables consistent governance without managing every agent individually.

## Who it is for

Observability for Agentic AI provides role-based views of the same underlying data. It uses the existing platform role model and does not introduce separate agent-specific administrator roles.

| Persona | Primary need | Platform role | Scope |
| --- | --- | --- | --- |
| CISO | Enterprise visibility into agent adoption, risk exposure, and issues requiring leadership attention | Technical Administrator | Enterprise-wide, limited to configured departments and sub-departments |
| Observability Owner | Day-to-day governance, including ownership assignment, remediation, investigations, quarantine, and decommissioning | Technical Administrator | All agents |
| Repository Manager | Visibility into agents in managed cloud accounts, groups, repositories, or platform environments | Repository Manager | Owned repositories only |
| Agent Owner | A focused view of owned agents, including usage, issues, and recommendations | Line Manager, extended to users who manage at least one agent | Owned agents only |

The Agent Owner view is created automatically for a user with line manager role. No separate provisioning is required.

## Getting started

To start using this feature, complete these three main steps:

1. Enable observability for Agentic AI in your environment.
2. Connect one or more supported agent platforms.
3. Allow the first synchronization to occur and review the dashboards in the Identity Observability portal.

After the first synchronization, the platform begins applying the included agent controls, dashboards, and canonical data mapping. You can then review discovered agents, investigate their access, assign owners, and address identified risks. You can also create custom controls and observations.

### Enable the capability

Ensure that you are subscribed to the Agentic AI License to access this feature and review the following:

1. In the **Environment Operations Center**, make sure the **Agentic AI** option is enabled for your Identity Observability application. You must have Identity Observability version 3 or higher to access this feature.
2. Assign the **AI Agents** entitlement to the users and groups who need access.

Activating the Agentic AI capability enables the agent data model, controls, dashboards, and interface components available. Agent-specific pages remain hidden until Radiant One connects with agent data. For example, the agent dashboard does not appear when no agents have been discovered. Similarly, a repository does not show an **Agents** tab until it contains at least one agent.

### Connect agent platforms

Configure agent data synchronization using [RadiantOne data source](to-add) for at least one supported agent platform.

By using a data source connector, Radiant One retrieves identity, configuration, access, and metadata information needed for observability, but does not retrieve secret values. Where credentials are detected, Radiant One records credential references and material types rather than secret contents.

The only supported write-back action is quarantine. It is disabled by default and must be explicitly enabled before Radiant One can request a source-platform quarantine action.

#### Supported platforms

| Platform | Supported agent types | Identity context |
| --- | --- | --- |
| Amazon Web Services | Amazon Bedrock Agents and Bedrock AgentCore Runtimes | AWS IAM, IAM roles, and STS AssumeRole chains |
| Microsoft Azure | Azure AI Foundry agents | Microsoft Entra ID, Azure RBAC, service principals, managed identities, and Dataverse security roles |
| Google Cloud | Vertex AI agents | Google Cloud IAM, service accounts, and workload identity |

For connector-specific requirements, setup instructions, and least-privilege permissions, refer to the connector listing in Connector marketplace.

## Next step

Learn how to complete your first observability review using the [AI Agent Dashboard](./ai-agent-dashboard.md). 
