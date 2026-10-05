# RadiantOne Identity Observability Release Notes

**September 30, 2026**

These release notes contain important information about new features, improvements, and bug fixes for RadiantOne Identity Observability (IDO) version 3.1.0.

These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements to Existing Features](#improvements-to-existing-features)
- [Bug Fixes](#bug-fixes)
- [Known Limitations](#known-limitations)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)


## New Features

* **IDO-486** – RadiantOne Observability can now be deployed in Self-Managed Kubernetes (K8s) mode.
* **IDO-449** – **Data Dock / Native CSV Upload**: Ability to provide CSV files as a data source with delta synchronization and direct ingestion into the Observability database.
* **IDO-1100** – The MCP server now supports questions about Agentic AI and Groups through four new tools.
* **IDO-1302** – Added an AWS Agent Refresh Monitor service to retrieve real-time data from AWS Bedrock.
* **IDO-1421** – Added agent-specific actions to collaborative remediation in Slack.
* **IDO-1287** – Added a manager resolution strategy for Guided Remediation on Agents.
* **IDO-718** – Manager updates made in real time are now automatically reflected in the Audit & Compliance point-in-time latest timeslot.


## Improvements to Existing Features

* **IDO-1410** – Added dynamic text on Agent controls.
* **IDO-1369** – Enabled navigation to the control detail page from the side panel even when only a single defect is present.
* **IDO-1313** – Excluded guest Entra ID accounts from the *Contractor with past ending date and active accounts* (`IDO_HR10`) and *Orphaned Account* (`IDO_ACC02`) controls.
* **IDO-1312** – Updated the Slack configuration file within Guided Remediation in Slack settings to support the Home Tab.
* **IDO-1311** – Clarified MCP tool terminology regarding "application" versus "resource."
* **IDO-1292** – Improved and refactored Guided Remediation in Slack workflows.
* **IDO-1274** – Relocated the "Connector Configuration" menu under "Event Monitoring" in the Data Sync Config endpoint.
* **IDO-1265** – Updated IDO Provisioning labels in the Data Sync Config endpoint.
* **IDO-1260** – Renamed *Identity Observability > Template Management* to *Mapping Profiles* in the Data Sync Config endpoint.
* **IDO-1259** – Added support for importing IDO Mapping Profiles in Control Panel within the Data Sync Config endpoint.
* **IDO-1212** – Added an **Account type** column to the Accounts tab on the Repository detail page.
* **IDO-1204** – Updated action labels for Agentic AI remediations (Enable/Disable and Quarantine/Un-quarantine).
* **IDO-1203** – Updated controls for Agentic AI.
* **IDO-1117** – Improved consistency between permissions control detail pages and remediation pages.
* **IDO-1113** – Enhanced Object Detail pages to display sources and defect comments for external control defects.
* **IDO-1107** – Added line manager remediation actions on Agentic AI controls for line manager end-users.
* **IDO-908** – Displayed dynamic tag information in the observation detail side panel on the Observation page.
* **IDO-907** – Improved table cell content management within the table widget.
* **IDO-817** – Limited the Observation database maintenance plan to the graph schema only.
* **IDO-692** – Synchronized manager changes from the Remediation page in real time with the latest point-in-time timeslot in Audit & Compliance.
* **IDO-627** – Added the ability to display the resource name on the Remediation page for resources.

---

## Bug Fixes

* **IDO-1395** – Fixed an issue where AWS change detection failed to detect newly added or deleted entities.
* **IDO-1375** – Fixed an issue where the tooltip displaying content for the Defect Comment column was not showing up.
* **IDO-1370** – Fixed an error encountered when sorting by Priority.
* **IDO-1297** – Fixed defect risk level labels for external controls so that descriptive labels are displayed instead of raw data.
* **IDO-1290** – Fixed an error occurring in Slack Guided Remediation when clicking the "Assign to me" action.
* **IDO-1276** – Resolved missing out-of-the-box Access Control Instructions (ACI) for the IDDM setup user that caused the Directory Browser to be unusable.
* **IDO-1275** – Fixed an issue where the control defect list failed to update.
* **IDO-1255** – Fixed pipeline reset behavior that inadvertently restarted the entire pipeline service and required an IDO Supervisor restart.
* **IDO-1237** – Fixed an issue where navigating back and forward from the Query Builder dashboard caused the Global Filter to be lost.
* **IDO-1208** – Fixed repository type display in the remediation policies configuration interface to properly show HR and Accounts types.
* **IDO-1181** – Fixed an issue where filtering by department was unavailable on identity and account control detail pages.
* **IDO-1143** – Fixed filtering by source in control tables.
* **IDO-1036** – Fixed an issue in the Control Library where search filters were not reapplied after adding a column.
* **IDO-412** – Corrected erroneous text in Slack remediation for built-in control `IDO_ACC09` (Password Not Required).


## Known Limitations

* **IDO-1267** – Upgrading to version 3.1.0 may require a manual portal pod restart.
* **IDO-1909** – Guided Remediation in Slack may fail due to object content size limits, particularly for groups with a very high number of members.
* **IDO-1547** – When remediation is performed via a third-party system, the defect status does not automatically update on the original card sent to the dedicated Slack channel.

## How to Report Problems and Provide Feedback

Feedback and problems can be reported through the Support Center / Knowledge Base:  
[https://support.radiantlogic.com](https://support.radiantlogic.com)

If you do not have a user ID and password, please contact: `support@radiantlogic.com`
