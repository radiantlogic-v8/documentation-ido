# RadiantOne Identity Observability Release Notes

**August, 2026**

These release notes contain important information about new features, improvements, and bug fixes for RadiantOne Identity Observability.

These release notes contain the following sections:

* [New Features](#new-features)
* [Improvements to Existing Features](#improvements-to-existing-features)
* [Bug Fixes](#bug-fixes)
* [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)


## New Features

- **IDO-1101** – Added MCP tools for Group details and issues.
- **IDO-1083** – Added `isQuarantined` support in the Query Builder.
- **IDO-1073**, **IDO-885** – Added configuration for the Agentic Amazon IDDM connector and connected the IDDM Azure connector.
- **IDO-1021** – Added the ability to change dynamic tags on custom observation and control detail pages.
- **IDO-1012** – Added dynamic markers to observation results.
- **IDO-1035**, **IDO-1015**, **IDO-970** – Added remediation capabilities for agent identities, including remediation operations and agent-detail remediation actions.
- **IDO-1084**, **IDO-895**, **IDO-986**, **IDO-985**, **IDO-861**, **IDO-962** – Added Agentic AI dashboards and views, including the CISO dashboard, security posture, risk sorting, family-level "View all" navigation, and deployment velocity.
- **IDO-847**, **IDO-846**, **IDO-836** – Added Agent Manager, Repository Manager, and home-page dashboards for owned agents and Agentic AI repositories.
- **IDO-746**, **IDO-745**, **IDO-752**, **IDO-857**, **IDO-896** – Added agent detail, security, and access-chain views, multi-hop dependency tracing, and line-manager visibility controls.
- **IDO-1023**, **IDO-751** – Added support for creating observations and controls on agents and managing agent data from detail pages.
- **IDO-916**, **IDO-911**, **IDO-910**, **IDO-909** – Added dynamic-tag configuration, display, and lineage capabilities, plus built-in tag support in control and observation engines.
- **IDO-881**, **IDO-982** – Added monitoring APIs and automatic monitoring-page refresh.
- **IDO-976**, **IDO-819**, **IDO-773**, **IDO-157** – Added the dormant-agent control, Agentic AI controls, graph-source agentic connectors, and tag-based observation queries.
- **IDO-926**, **IDO-925**, **IDO-924**, **IDO-944** – Added configurable remediation targets for write-back and dispatch policies, and completed Agentic AI front-end support.
- **IDO-984**, **IDO-1034**, **IDO-929**, **IDO-1024** – Added JSON attribute display updates, optional Query Builder columns for agent intent and provider, updated agent metadata, and multi-entity manual tag management.
- **IDO-1082**, **IDO-1048**, **IDO-1081** – Added role-specific restrictions for agent managers, repository owners, and line managers, and FIPS compliance for Secret Manager.
- **IDO-993**, **IDO-1037** – Added the Agentic AI portal feature flag and improved risk-description and remediation coverage for agent controls.


## Improvements to Existing Features

- **IDO-1049**, **IDO-1105**, **IDO-1120**, **IDO-1047** – Improved the control and observation experience with entity-type filtering, refresh actions for observations, and better back navigation across controls and agent dashboards.
- **IDO-940**, **IDO-938** – Improved Query Builder support for JSON subfields and ensured agent external identifiers are available on detail pages.
- **IDO-797** – Improved remediation-policy configuration so write-back can be selected by action with defaults and overrides.
- **IDO-1155**, **IDO-1031** – Improved control-detail date presentation and enabled exclusion of external Entra ID accounts from controls.
- **IDO-1080** – Improved remediation-target handling so JSON configuration can be edited.


## Bug Fixes

- **IDO-1221**, **IDO-1218** – Corrected the MCP server Keycloak URL and fixed the definition of the "No registered guardrails/ASI01_C1" control.
- **IDO-1215**, **IDO-1210**, **IDO-1209**, **IDO-1017** – Fixed missing agent-identity alert notifications and corrected agent status, quarantine, and unquarantine behavior in the Query Builder and portal.
- **IDO-1197**, **IDO-1139**, **IDO-1176** – Fixed Agentic data ingestion, pipeline write-back notifications, and stale external controls after sink-template changes.
- **IDO-1167**, **IDO-1112**, **IDO-1032**, **IDO-932**, **IDO-737**, **IDO-851**, **IDO-783** – Fixed missing department links, incorrect department and hierarchy counts, duplicated external controls, and incorrect department-level agent presentation.
- **IDO-1133**, **IDO-1122**, **IDO-995**, **IDO-923** – Fixed breakdown failures, relative-date controls and observations, unsupported non-string attributes, custom-control breakdown updates, and missing breakdowns on controls and observations.
- **IDO-988**, **IDO-892**, **IDO-952**, **IDO-833**, **IDO-623** – Fixed remediation pages for accounts, including empty email fields and missing identity attributes, and corrected manager and account operations after remediation.
- **IDO-967**, **IDO-957**, **IDO-943** – Fixed Slack notification recipient handling, the missing "Who Is Notified" dialog, and notification counters involving blacklisted recipients.
- **IDO-931**, **IDO-631** – Fixed agent identity detail-page issues, missing agent descriptions and intents, and incorrect control and repository ordering.
- **IDO-1051**, **IDO-1044**, **IDO-1014**, **IDO-768** – Fixed observation and control naming with unsupported special characters, date typing in the data model, tag filtering, and tag export to XLSX and CSV.
- **IDO-998**, **IDO-826**, **IDO-713**, **IDO-73** – Fixed Explorer navigation with searched items, account reset/reapply states, reconciliation issues, and campaign initialization master-detail behavior.
- **IDO-1159**, **IDO-1050** – Fixed the control-detail description fallback and the remediation success message shown after deleting an observation.


## Known Issues

- **IDO-1267** – Update from 2.4.0 might require a manual portal restart (via EOC).
- **IDO-1257** – In case of version upgrade 2.4.0 → 3.0.0, deprecated controls will remain visible (deprecated controls won't be shown in case of fresh installation of 3.0.0).
- **IDO-1130** – Manual reconciliation from the portal might not be persistent for technical accounts.
- **IDO-1290** – Guided remediation in Slack: "Assign to me" button might not function as expected when multiple managers are notified.

## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base:
https://support.radiantlogic.com

If you do not have a user ID and password, contact: support@radiantlogic.com
