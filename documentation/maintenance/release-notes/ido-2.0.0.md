---
title: RadiantOne IDO Release Notes
description: RadiantOne IDO Release Notes
---

# Identity Observability Release Notes

April, 2026

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.  
These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements](#improvements)
- [Bug Fixes](#bug-fixes)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)


## New Features

**IDO-265, IDO-263, IDO-261, IDO-256, IDO-226, IDO-225, IDO-204, IDO-199, IDO-172, IDO-146, IDO-114** – Migrated core IDO services (MCP server, ledger operations, writeback, observation supervisor, Explorer/Swimlane APIs, guided remediation, and graph pipelines) to the new storage v2–based realtime database to improve performance, scalability, and reliability.

**IDO-381, IDO-380, IDO-331, IDO-205, IDO-202, IDO-201** – Delivered the first full version of the duplicated identities workflow in Audit & Compliance, including analysis results, side-by-side comparison of identity data, remediation decisions, and remediation justifications.

**IDO-395, IDO-394, IDO-232, IDO-319** – Expanded the Security Events Hub with a dedicated service, configuration endpoints, event polling from third-party sources and Time To Live for control defects.


## Improvements

**IDO-376** – Added a Helm chart option to pass the AIDA activation flag to the Global scheduler so AIDA can be enabled or disabled at deployment time.

**IDO-510, IDO-508** – Refined the duplicated identities experience in Audit & Compliance by correcting display issues on the duplicates page and fixing inconsistencies when comparing manager fields.

**IDO-499** – Improved NATS pod placement and distribution to better support high availability and resilient message processing.

**IDO-490, IDO-403, IDO-243, IDO-113, IDO-112, IDO-111** – Enhanced the Query Builder and control model by optimizing DISTINCT usage, aligning realtime rules with true/false attributes, simplifying the data model for account owners, restoring result tables in the control and observation creation wizard, moving Query Builder to realtime search widgets and views, and improving tag display and editing.

**IDO-414, IDO-256** – Improved overall system performance and resilience through request-handling optimizations and Flink operator–based graph pipeline management for smoother upgrades and clearer error reporting.

**IDO-252** – Extended remediation authorization so persona-based access covers indirect managers, allowing them to remediate objects they are responsible for.


## Bug Fixes

**IDO-495, IDO-489, IDO-455, IDO-405, IDO-404, IDO-401, IDO-371, IDO-357, IDO-356, IDO-336, IDO-95** – Fixed multiple issues in Query Builder and identity/resource exploration, including broken tag-based sorting, missing or incorrect account/identity attributes, incorrect columns, erroneous filters, empty graphs for inaccessible resources, and departments without members not appearing in results.

**IDO-494, IDO-396** – Corrected UI issues in the Audit and Compliance React pages and Acount Detail pages, so email attributes and time slot selection are displayed properly.

**IDO-418, IDO-417** – Restored results on Audit & Compliance search pages and fixed incorrect or broken risk assessment explanatory text.

**IDO-392, IDO-390** – Fixed ordering problems for accounts in Query Builder and for the “Top X risky objects” on the homepage so lists are now sorted correctly by priority.

**IDO-391, IDO-311, IDO-310, IDO-280, IDO-279, IDO-277** – Resolved several guided remediation issues with AIDA in Slack, including HTTP 400 failures, specific remediation actions not working as expected, and ensuring newly created Slack users are taken into account via automated and on-demand reload mechanisms.

**IDO-385, IDO-384, IDO-234, IDO-229, IDO-198, IDO-163, IDO-125, IDO-124** – Addressed multiple Audit & Compliance review and remediation problems such as workflow errors when creating reviews, stuck perimeter previews, incorrect default reviewer selection, mismatched campaign owner information, failures when removing accounts, blank review detail pages, missing compliance reports, and synchronization issues between Realtime and Audit & Compliance.

**IDO-351** – Fixed polymorphic tag management issues so tags behave consistently across object types.

**IDO-254** – Corrected the Control Browser so control types are visible and discrepancies are reported correctly.

**IDO-453** – Fixed failures in the portal-setup job so initial portal configuration completes reliably.

## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base accessible from:  
https://support.radiantlogic.com  

If you do not have a user ID and password to access the site, please contact:  
support@radiantlogic.com
