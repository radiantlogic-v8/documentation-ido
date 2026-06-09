# RadiantOne Identity Observability Release Notes

**June, 2026**

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.

These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements to Existing Features](#improvements-to-existing-features)
- [Bug Fixes](#bug-fixes)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)

## New Features

- **IDO-620, IDO-621, IDO-630, IDO-733, IDO-764** – Expanded the Security Event Hub with new capabilities for external controls: predefined (built-in) external controls can now be configured, the control management page displays external controls in a dedicated "External" tab as well as in the "All" tab, a new dedicated external control detail page shows the list of objects in defect, the configuration of predefined external controls has been extended, and sink template dynamic controls now support different risk levels for finer-grained risk assessment.

- **IDO-728, IDO-762** – Enhanced the Data Model with new capabilities: different edge type definitions can now be configured per source object for the same edge, and JSON attributes are now supported in graph-access tag insertion.

- **IDO-521** – Connectors now support access-pass fan-out from complex JSON attributes for SailPoint, CyberArk and other connectors, enabling richer data extraction from nested structures.


## Improvements to Existing Features

- **IDO-723** – Connectors now support incremental configuration changes for fan-out and derived source objects for SailPoint, CyberArk and other connectors, reducing the need for full re-synchronization after configuration updates.

- **IDO-324, IDO-438** – Improved visibility about connector deployment issues: technical administrators now see connector connection issues in a dialog box, and deployment problems are surfaced more clearly.

- **IDO-325, IDO-721** – Enhanced Data Mapping with a new functional conformity module that checks mandatory attributes, and improved error handling and detection of corrupted configuration.

- **IDO-88, IDO-691** – Changes to account types (user, technical/service, orphaned accounts) and object managers made via the real-time detail pages are now immediately reflected in Audit & Compliance.

- **IDO-171, IDO-603** – Improved Realtime User Interface: all formatted dates in Control & Remediation detail pages now use a consistent 3-letter month format (e.g., "Jan", "Feb"), and aggregated job titles are now handled in realtime views.

- **IDO-706** – Vulnerability fixes reported from CoreProduct have been applied to IDO.

- **IDO-705** – Fixed the Guided Remediation in Slack interface so that only the appropriate action button ("assign to me" or "disabled") is displayed during remediation, rather than both simultaneously.

- **IDO-397** – Cleaned up WriteBack service logs to improve remediation traceability and reduce noise.


## Bug Fixes

- **IDO-246, IDO-241, IDO-535, IDO-517, IDO-625, IDO-563** – Fixed several Realtime User Interface issues: Line Managers and department managers can now see their sub-departments in the Control & Remediation page, simple users can no longer access the detail page of their managers, tag advanced filtering now displays tag labels instead of internal names across all dashboards, the tag management page no longer bleeds object lists over text when the list exceeds the page boundary, permission control detail and remediation pages now display the permission name in addition to the permission code, and the Query Builder identity search now shows the correct job title column label.

- **IDO-647, IDO-637, IDO-663, IDO-726** – Resolved data display issues in Realtime User Interface: empty attributes in underlying real-time views are now correctly populated, the landing page error for repository managers has been fixed, multivalued department memberships are now taken into account on identity detail pages, and table XLSX and CSV exports of JSON attributes are now generated correctly.

- **IDO-715, IDO-699, IDO-649, IDO-237, IDO-190** – Fixed multiple User Access Review issues: the review strategy no longer incorrectly assigns user accounts to the default reviewer, cross-table cluster sorting on pages called from React pages with resource paths now works correctly, the review page of "sticky to timeslot" campaigns no longer allows selecting other timeslots, additional supported languages (Spanish and English) are now provided by default for mail templates, and out-of-memory errors in the Audit & Compliance review management page have been resolved.

- **IDO-690, IDO-688, IDO-678, IDO-553, IDO-75** – Fixed several Audit & Compliance issues: leaver account numbers are now correctly displayed in dashboards, the Active Directory control mashup dashboard can now select the AD domain, looping execution plans during realtime-to-Audit & Compliance data upload have been resolved, mashup dashboards have been updated to match built-in controls from real-time data, and the number of applications in the repository is now correctly computed when uploading data from realtime to Audit & Compliance.

- **IDO-684, IDO-619, IDO-618, IDO-606** – Addressed Duplicate Identities issues: a unique identity with multiple jobs and departments is no longer displayed as several identities, the ServiceNow ticket "requested on" field now contains the correct value for duplicated identities, the Remediation Strategy Management page now includes a column to count tickets related to identity remediation, and the format of generated ServiceNow CSV attachments has been corrected.

- **IDO-636, IDO-565, IDO-247** – Fixed Remediation issues: resource managers can now remediate permissions again, overlapping information in the remediation dashboard when initiating remediation from a control detail page has been resolved, and the status of defects in the audit trail page is now correctly updated.

- **IDO-704, IDO-652** – Fixed Controls issues: the IDO_HR06 "Active identity with only dormant accounts" built-in control can now be published to the observation supervisor without error, and clicking Apply twice in the Query Builder no longer clears query results.

- **IDO-689** – Fixed a Data Model issue where the portal did not load remote schema from the schema manager, which is required to support data model extension.

- **IDO-577** – Fixed the Guided Remediation in Slack configuration file provided to configure Slack for Guided Remediation with AIDA, which was previously invalid.

- **IDO-525** – Resolved an Out of Memory error occurring on the Edge browser.


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base:

[https://support.radiantlogic.com](https://support.radiantlogic.com)

If you do not have a user ID and password, contact:

[support@radiantlogic.com](mailto:support@radiantlogic.com)
