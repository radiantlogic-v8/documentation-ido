---
title: RadiantOne IDO Release Notes
description: RadiantOne IDO Release Notes
---

# RadiantOne Identity Observability Release Notes

November 02, 2026

These release notes contain important information about new features, improvements, and bug fixes for RadiantOne Identity Observability (IDO) version 3.2.0.

These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements to Existing Features](#improvements-to-existing-features)
- [Bug Fixes](#bug-fixes)
- [Known Limitations](#known-limitations)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)

## New Features

- **IDO-207** – Prepared the front end for internationalization (i18n) and localization to support future interface translations.
- **IDO-843** – Added the ability to override a remediation target on the local IDO database only.
- **IDO-844** – Added options to deactivate all remediations, or deactivate remediations per repository, when the target is not local.
- **IDO-1277** – Added the ability to define a custom project instead of relying on the default repository.
- **IDO-1294** – Added automated remediation processing when a control detects a new defect.
- **IDO-2008** – Expanded Mashup Dashboards to allow targeting controls and observations.
- **IDO-2150** – Added reliable portal project synchronization for Git projects.
- **IDO-2180** – Embedded the default IDO project in Helm to support self-managed deployment and project customization.

## Improvements to Existing Features

- **IDO-901** – Updated defect status handling to manage "in progress" remediation statuses as "submitted."
- **IDO-1096** – Updated deployment support for Azure with the new Datadock / CSV Native connector feature.
- **IDO-1205** – Enabled remediation action details to be displayed directly from remediation pages.
- **IDO-1227** – Improved identity pipeline handling to support very large entries and events end-to-end.
- **IDO-1228** – Added support for excluding unwanted source entries from the identity pipeline.
- **IDO-1243** – Added a disabled column to the accounts table in Query Builder.
- **IDO-1303** – Improved support for MCP management queries.
- **IDO-1314** – Updated MCP tool behavior so large groups no longer cause the `accounts[]` field to be truncated after 100 members.
- **IDO-1360** – Enhanced MCP permission and identity tools to include permissions granted through groups.
- **IDO-1401** – Updated remediation policy labels and wording for clarity.
- **IDO-1546** – Improved the display of remediation previews for agents.
- **IDO-1547** – Improved Slack card and note handling for guided remediation outside of Writeback.
- **IDO-1555** – Updated ledger operations using the `MD5()` PostgreSQL function to ensure full compatibility with the Postgres FIPS image.
- **IDO-1556** – Replaced MD5 with SHA-256 in ledger operations for FIPS compliance.
- **IDO-2148** – Improved Mashup Dashboards by hiding irrelevant aggregations depending on the selected entity.
- **IDO-2149** – Restricted indicator detail links in Mashup Dashboards so they are available only when the first indicator is selected.
- **IDO-2157** – Added missing predicates for observations in SmartSearch.
- **IDO-2165** – Improved Mashup Dashboard autocomplete for observation searches by code and name.
- **IDO-2184** – Updated the connector setup user interface to include Security Event Hub connectors.
- **IDO-2688** – Updated YAML configuration files regarding computed attributes in version 3.2.

## Bug Fixes

- **IDO-906** – Fixed confusing labeling in Review Strategies.
- **IDO-1085** – Resolved Excel export formatting issues on the Tag Management page.
- **IDO-1119** – Resolved an issue where using the `concat` function in a pipeline introduced unexpected characters.
- **IDO-1130** – Fixed an issue where manual reconciliation failed when attempting to flag accounts as service accounts.
- **IDO-1171** – Fixed an issue where the agent search list export included raw JSON formatting for certain columns.
- **IDO-1361** – Fixed an issue in Tag Management where the "Source & Usage" field remained blank instead of displaying the control/observation name when a tag was assigned to a control.
- **IDO-1368** – Fixed the Repository Managers tab on the Repository Detail page so the Type column is populated when the manager is a contractor.
- **IDO-1415** – Fixed a graph pipeline reconciliation crash loop that occurred when a non-human account lost its DIT-based rule and lacked an owning identity.
- **IDO-1427** – Fixed an issue on the Account Detail page Groups tab where sorting was disabled when the Direct group membership checkbox was checked.
- **IDO-1550** – Fixed an issue where refreshing any IDO page redirected the user to the landing page.
- **IDO-1554** – Resolved an IDDM authentication error with the built-in AWS Change Detection service after upgrading from 3.0.0 to 3.0.1.
- **IDO-1909** – Resolved guided remediation failures caused by large volumes of data for groups.
- **IDO-1910** – Fixed an issue where agent remediation actions were missing from Slack guided remediation.
- **IDO-1970** – Resolved an issue with the "from" field in email alert and target configuration.
- **IDO-2092** – Fixed custom metadata string slot handling so characters are no longer truncated at the configured limit (`limit - 1`).
- **IDO-2093** – Fixed an issue where custom metadata upload from Realtime to Audit & Compliance generated invalid SQL for `ext:` columns, causing the run to abort.
- **IDO-2132** – Fixed an issue where AWS Change Detection configuration was lost when the application stopped or a pod crashed.
- **IDO-2137** – Fixed an issue that prevented users from deleting tags on entities (such as Groups and Resources).

## Known Limitations

- **IDO-2265** – Remediation actions performed as a Line Manager are currently not available.
- Customization of projects in SaaS environments requires version 2.1 of EOC (scheduled for release later this month).

## How to Report Problems and Provide Feedback

Feedback and problems can be reported through the Support Center / Knowledge Base:

https://support.radiantlogic.com

If you do not have a user ID and password, please contact: support@radiantlogic.com 
