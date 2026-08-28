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
