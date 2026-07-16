# RadiantOne Identity Observability Release Notes

**July, 2026**

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.

These release notes contain the following sections:
* [New Features](#new-features)
* [Improvements to Existing Features](#improvements-to-existing-features)
* [Bug Fixes](#bug-fixes)
* [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)

## New Features

* **IDO-821** – New Security Event Hub extractor for CrowdStrike, enabling ingestion of identity threat protection events into the Security Events Hub.
* **IDO-765** – Support for ProofPoint VAP and TopClickers security events in the Security Event Hub, broadening the range of third-party identity-centric signals.
* **IDO-869, IDO-868** – Expanded the External Controls presence across the portal: external controls are now included in the home page grouped by family for all roles (tech admin, line manager, resource owner, etc.), and are also surfaced in detail pages of related entities such as accounts and identities.
* **IDO-853** – Custom Attributes now support a JSON configuration to manage the upload from realtime to point-in-time, enabling consistent attribute enrichment across both views.
* **IDO-791** – New MCP tools for Application and Permission details, enabling your AI assistants to query identity data programmatically.
* **IDO-781** – The DataViz Explorer access chain now extends from identities to display technical account, group, and resource managers for a more complete visibility picture.

## Improvements to Existing Features

* **IDO-622, IDO-806, IDO-739** – Pipeline Configuration improvements: a new monitoring page displays "parameter conformity" issues in pipeline configuration YAML, edge retry stream tuning improves pipeline reliability, and error data retention period has been reduced to 1 week for cleaner operations.
* **IDO-70** – Custom attributes are now displayed in real-time across all entity-related detail pages.
* **IDO-102** – End-users can now manage recipient blacklists directly from the settings interface for notifications, without requiring admin intervention.
* **IDO-278** – Added UI configuration for Guided Remediation in Slack to schedule and force the reload of the contact list.
* **IDO-248** – Improved handling of defect statuses in Remediation, particularly when a remediation action does not fully resolve the event.
* **IDO-788** – Permission hierarchy is now displayed in the DataViz Explorer for better access structure visibility.
* **IDO-789, IDO-596** – EOC enhancements: new option to enable MCP Server Services, and new option to enable external access to the Portal API.
* **IDO-827** – Fields can now be copied from the Identity Observability portal interface.

## Bug Fixes

* **IDO-854, IDO-840, IDO-823, IDO-822** – Fixed several Query Builder issues: removed invalid search criteria containing (+self) from all entities, fixed duplicated entries, resolved duplicate rows when querying departments with child departments, and corrected an incorrect warning when bulk-removing Sensitivity Level.
* **IDO-782, IDO-777, IDO-759** – Fixed Visibility Cone issues: resource managers can no longer remediate on resources they don't manage from the landing page, remediation for Highly Sensitive Group Manager role is resolved, and Line/department managers can now see their sub-departments in the homepage and Query Builder page.
* **IDO-814, IDO-798, IDO-795** – Fixed User Access Review issues: technical accounts are now correctly reviewed by the Application Owner instead of the Default reviewer, review campaign configuration counter issues are resolved, and the correct Identity Observability controls have been added to AIDA for the anomalous detection step in UAR.
* **IDO-785, IDO-784** – Fixed Audit & Compliance issues: Active Directory Controls Mashup Dashboard issues are resolved, and service and technical accounts are now loaded correctly in the point-in-time database.
* **IDO-922** – Fixed Guided Remediation in Slack so that AIDA-based remediation now correctly takes into account the blacklisted addresses list.
* **IDO-898** – Fixed a spurious error message appearing when enabling or disabling a control or observation in the Observation/Control Management page.
* **IDO-804** – Fixed display issues on external control detail pages.
* **IDO-897** – Fixed DataViz issue where swimlanes were not displayed correctly.
* **IDO-825** – Fixed Reconciliation issue where orphaned accounts were displayed as "Unknown" in environments using static datasets.
* **IDO-502** – Fixed missing entries in the Department Detail Page Hierarchy Tab.

## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base:
[https://support.radiantlogic.com](https://support.radiantlogic.com)

If you do not have a user ID and password, contact:
[support@radiantlogic.com](mailto:support@radiantlogic.com)
