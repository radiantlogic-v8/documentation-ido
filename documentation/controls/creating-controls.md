## Overview 
 
Controls provide automated detection mechanisms to identify potential security risks and compliance violations within the identity infrastructure. Users can enable relevant controls for their specific context or create their own custom controls. 
A control represents an issue with a risk level, risk description and a remediation plan to fix it. Control risk levels are used to compute the risk score of an item (Accounts, Identities, Departments, and More). 

Like observations, controls are based on a rule that contains the criteria of what needs to be monitored in real time. The result of the rule which is a list of items is maintained in real time. Those controls can be used in out-of-the-box or custom dashboards. Notifications can also be activated by control to notify people when a change occurs in the list of items, meaning new items are returned or removed from the list by the rule. 

 
## Default controls 
 
Default controls are pre-configured security and compliance checks designed to identify and manage risks in your identity environment. These controls help enforce best practices by automatically monitoring for potentially dangerous situations, enabling you to catch issues proactively. Default controls include controls for human identities as well as agent identities. 
 
### Examples 

The control "Account with abnormal login attempts" alerts you to accounts with repeated unsuccessful login attempts, flagging them for possible brute force attacks. 
"Contractor with past ending date and active accounts" highlights cases where contractors who have left the company still possess active accounts, which could be exploited for unauthorized access.  
 
Another common default control is "MFA is not registered," which identifies accounts that are missing multi-factor authentication, exposing them to higher risk of takeover. 
 
You can enable or disable any of these default controls. You can also create your own control by following the steps below.  

### Agent controls

Agent controls are a specialized subset of controls targeting AI agents registered in the platform. Agents are treated as first-class entities within the existing governance framework — the same risk-scoring engine used for accounts aggregates agent control defects into per-agent risk scores and family-level rollups displayed across dashboards. No separate scoring system is introduced for agents.

Agent controls are organized around the OWASP Agentic Security Initiative (ASI) Top 10 (2026) — a taxonomy designed specifically for agentic systems, distinct from the OWASP Top 10 for Web Applications and the OWASP Top 10 for LLM Applications. Each ASI family maps to exactly one control family in the platform.

These controls are available across the following ASI families as preconfigured default agent controls. 


| ASI family                                    | Controls count | Highest risk |
| --------------------------------------------- | -------- | ------------ |
| ASI01 — Agent Goal Hijack                     | 3        | Critical     |
| ASI02 — Tool Misuse and Exploitation          | 1        | High         |
| ASI05 — Unexpected Code Execution (RCE)       | 3        | High         |
| ASI07 — Insecure Inter-Agent Communication    | 3        | High         |
| ASI09 — Human-Agent Trust Exploitation        | 6        | Critical     |
| ASI10 — Rogue Agents                          | 4        | Critical     |
| ASI00 — Others                                | 1        | Medium       |

For the full catalogue of built-in agent controls with descriptions and remediation guidance, see [Agent control catalogue](#agent-control-reference).

To create a custom control beyond the default controls, see [Creating a new control](#creating-a-new-control) below.

## Creating a new control 

1. Click the Control menu item on the left navigation. This opens the Control management interface, which lists existing controls and provides a button to create a new one.

   ![Image of control in the navigation menu](Media/controls-menu.png "Image showing where Control is located in the navigation")

2. Click the "Create control" button to open the control creation interface.

3. On the Criteria step, use the entity dropdown on the left to select the population the control evaluates: Identities, Departments, Accounts, Groups, or Agent Identities. For standard account and access controls, choose the relevant identity or entitlement entity. To evaluate AI agents, choose **Agent Identities**.

   ![Image of the entity selection on the control creation page](Media/control-criteria.png "Image showing the entity dropdown on the Criteria step")

   Define one or more conditions using **Criterion** and **Group**. Each criterion is built from an attribute, an operator, and a value, and the attributes offered depend on the entity you selected. Departments, for example, can be evaluated on conditions such as having tags, not having tags, having assigned agents, not having assigned agents, managed by identity, managed by person, and not managed by identity. Agent identities can be evaluated on lifecycle events such as published, suspended, blocked, and deleted, on the identity that created, published, invoked, blocked, or deleted the agent, and on values such as last updated or aggregated risk level.

   ![Image showing how a criterion is built](Media/control-criterion.png "Image showing the attribute list for a criterion")

   Use **Group** to nest conditions and control how they combine. A nested group carries its own **Criterion** and **Group** options, so you can express conditions such as an agent that has at least one tool with no API schema, or a subagent endpoint using plaintext HTTP.

   > Risk is surfaced at the entity level rather than the sub-object level. When an agent matches a control, its at-risk tools, subagents, and resources are implied. The control does not raise a separate defect for each sub-object.

4. Click **Apply**. The table below refreshes with the entities that currently match, so you can confirm the criteria return what you expect. Click **Next**.

   ![Image of control criteria applied](Media/control-apply.png "Image showing applied criteria on the Criteria step")

5. On the Control Details step, complete the following sections.

   **5.1. Control Details.** Give the control a unique, descriptive name and a description that explains its purpose and scope to the rest of your team. Select the **Control Family** that best fits: Authentication, Identity Lifecycle, Privilege and Access, or Identity Hygiene. Use the status toggle to decide whether the control begins monitoring right away or is saved in a disabled state for later. A disabled control is saved but does not evaluate criteria or raise alerts.

   ![Image of the Control Family list](Media/control-family.png "Image showing the four control families")

   **5.2. Dynamic Tags.** Click **Add Tag** to attach one or more tags to the control. Select an existing tag from the list or type a new name and click **Create** to add one. Tags are assigned dynamically to every entity that matches the control, so they follow the population as it changes. Click **Edit** to change the tags later.

   ![Image of the dynamic tag picker](Media/control-tags.png "Image showing selecting or creating a dynamic tag")

   ![Image of the Control Details step](Media/control-details.png "Image showing control name, description, family, status toggle, and dynamic tags")

   **5.3. Breakdown Configuration.** This section is optional. Click **Configure Breakdown**, then use **Breakdown by** to select the criteria you want results segmented on, for example Department Type, Sensitivity Level, Priority based on Risk Level, Employee Type, or Identity Job Titles. Click **Remove Breakdown** to clear the selection.

   ![Image of breakdown configuration](Media/control-breakdown.png "Image showing a selected breakdown criterion")

   Click **Next**.

6. On the Risk Assessment step, select the default risk level for the control: Critical for severe impact such as a major breach or service outage, High for serious impact, Medium for moderate impact, or Low for limited impact. Use **Risk Description** to explain the potential impact and consequences of the risk. Click **Next**.

   ![Image of Risk Assessment](Media/control-risk.png "Image showing risk level selection and risk description")

7. On the Remediation step, turn on **Enable Remediation** to define how matches are resolved.

   * **4.1. Suggested Remediation.** Provide clear, actionable steps that teams can follow to mitigate the risk.
   * **4.2. Remediation Configuration.** Select which actions are offered to the people working the control. **False Positive**, **Mark as Exception**, and **Mark as Processed** are available by default. Click **Add New Action** to define an additional action.

   ![Image of the Remediation step](Media/control-remediation.png "Image showing suggested remediation and remediation actions")

   Leave the toggle off to create the control without remediation guidance. Click **Next**.

8. On the Setting Visible Attributes step, choose which attributes appear as columns in the control table. Click the eye icon next to an attribute to show or hide it. Viewers can still adjust column visibility from the table settings afterwards, so this sets the starting view rather than a permanent one.

   ![Image of the Setting Visible Attributes step](Media/control-visible-attributes.png "Image showing attribute visibility selection")

   If an attribute you want is not listed, click **Advanced settings**. In the dialog, search for an attribute and click the plus icon to add it, or click the X to remove one from the table. Use **+ Add All** and **Remove All** to work with the full list at once. Click **Apply** to return to the step with your changes in place, then click **Next**.

   ![Image of the Advanced settings dialog](Media/control-advanced-settings.png "Image showing the Advanced settings column picker")

9. On the Alert Configuration step, decide how the control signals its findings.

   **Enable SSF/CAEP.** Turn this on to publish Shared Signals Framework (SSF/CAEP) events for the control. Click **Choose file** to upload the transmitter configuration, or click **Preview Sample Configuration** to see the expected format. The step reports **Config required** until a valid configuration is supplied.

   ![Image of the Alert Configuration step](Media/control-alerts.png "Image showing the Alert Configuration step")

   ![Image of SSF/CAEP configuration](Media/control-ssf.png "Image showing SSF/CAEP configuration upload")

   **Enable notifications for this control.** Turn this on to activate and configure how and when alerts fire for this control:

   * **Triggering event.** When to fire: on new defect opened, defect closed, risk level change, or a scheduled digest.
   * **Grouping window.** The time window within which multiple matching events are bundled into a single alert notification, for example every 30 minutes, 1 hour, 4 hours, or daily. Use this to avoid alert fatigue when a control matches many agents at once.
   * **Flood protection.** The maximum number of alerts sent within a rolling time period. Once the threshold is reached, further notifications are suppressed until the window resets.
   * **Channel.** Delivery method: Email, Slack, Microsoft Teams, or Webhook.
   * **Recipients.** The users, groups, or webhook endpoints to notify.

   > Alert delivery channels must be configured by a technical administrator under **Admin > Settings > Alert templates** before they are available for selection here. Contact your platform administrator if a channel is missing. Until channels are configured, this step shows a notice in place of the notification settings.

10. Click **Submit**. To view the control you created, go to Controls > My Controls and click its name. If you saved the control in a disabled state, you can activate it from the My Controls page.

## Agent control reference

Use this reference to review the built-in controls available for AWS agents. Controls are organized by ASI family and identify common configuration, lifecycle, and security gaps.

You can enable or disable each control. You can also duplicate a control and modify it to meet your project’s requirements.

### ASI01: Agent Goal Hijack

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | --- |
| ASI01-C1 | No registered guardrails | Privilege & Access | Critical | A published AWS agent has no guardrail that actively enforces a restriction. This can mean no guardrails are configured, or that none are enabled with a blocking or human-review action. |
| ASI01-C4 | Agent version is missing | Identity Hygiene | Medium | The AWS agent does not have a recorded version. Without version information, it is difficult to track configuration changes over time. |
| ASI01-C5 | Agent has no description | Identity Hygiene | Medium | The AWS agent does not have a documented purpose or scope. Without a defined baseline, it is difficult to tell whether the agent is behaving as intended or has drifted from its original goal. |

### ASI02: Tool Misuse and Exploitation

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | --- |
| ASI02-C3 | Enabled tool is undocumented | Identity Hygiene | High | An enabled tool has no executor reference or API schema. Reviewers cannot validate its inputs or determine which code runs when the tool is called. |

### ASI05: Unexpected Code Execution

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | ---|
| ASI05-C2 | Code-execution tool has no API schema | Identity Hygiene | High | The tool does not provide an API schema, so its inputs cannot be validated before execution. |
| ASI05-C3 | Code-execution tool has no executor reference | Identity Hygiene | High | The tool does not identify its runtime target. This may allow the agent to resolve the target dynamically from untrusted context. |
| ASI05-C4 | Inactive published agent has code-execution tools | Identity Lifecycle | High | A published AWS agent has never been invoked but still has code-execution tools enabled. This creates a remote-execution exposure that may go unnoticed because the agent is inactive. |

### ASI07: Insecure Inter-Agent Communication

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | --- |
| ASI07-C3 | Full conversation data sent to an external subagent | Authorization | High | The AWS agent sends its full conversation context to an external agent. Because the outbound destination is outside your control, this can expose more data than necessary. |
| ASI07-C5 | A2A-compliant agent has no skills | Identity Hygiene | Medium | The AWS agent declares A2A compliance but does not publish any skills. This may indicate that the agent advertises capabilities it cannot provide. |
| ASI07-C6 | Subagent endpoint uses plaintext HTTP | Authentication | High | The subagent endpoint uses `http://` rather than HTTPS. Data sent to the endpoint may be transmitted in plaintext, regardless of the configured authentication method. |

### ASI09: Human-Agent Trust Exploitation

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | --- |
| ASI09-C4 | State transition history is disabled | Identity Hygiene | Medium | The AWS agent does not have the `stateTransitionHistory` feature enabled. This can limit visibility into changes to the agent’s state. |
| ASI09-C9 | No business owner assigned | Identity Hygiene | Medium | The AWS agent does not have an assigned business owner. |
| ASI09-C10 | No technical owner assigned | Identity Hygiene | Medium | The AWS agent does not have an assigned technical owner. |
| ASI09-C11 | Business owner is no longer employed | Identity Hygiene | Medium | The assigned business owner is no longer part of the organization. |
| ASI09-C12 | Technical owner is no longer employed | Identity Hygiene | Medium | The assigned technical owner is no longer part of the organization. |
| ASI09-C13 | Business owner is a contractor | Identity Hygiene | Medium | The assigned business owner is external to the organization. Consider assigning an internal owner who can provide long-term accountability. |

### ASI10: Rogue Agents

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | ---|
| ASI10-C1 | Published agent has no publication date | Identity Lifecycle | Critical | The AWS agent is in `PUBLISHED` status, but its publication timestamp is missing. |
| ASI10-C7 | Agent remains in the created state | Identity Lifecycle | Medium | The AWS agent has been in `CREATED` status for more than 30 days. This can indicate abandoned provisioning or an agent that is outside the expected lifecycle process. |
| ASI10-C8 | Published agent has never been invoked | Identity Lifecycle | Medium | The AWS agent was published more than 30 days ago but has never been invoked. This creates inactive attack surface and provides no usage baseline for detecting unexpected activity. |
| ASI10-C9 | Previously used agent is now inactive | Identity Lifecycle | Medium | The AWS agent was used in the past but has not been invoked in the last 90 days. Review the agent to determine whether it should be reactivated, retired, or formally deprecated. |

### ASI00: Other Controls

| ID | Control | Category | Risk | What it identifies |
| --- | --- | --- | --- | ---|
| ASI00-C1 | Dormant agent | Identity Lifecycle | Medium | A published AWS agent has not been used for more than 90 days. Its associated workflow may no longer be needed, but the agent can retain access and permissions until it is decommissioned. |
