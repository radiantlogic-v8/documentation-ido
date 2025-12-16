---
title: RadiantOne IDO Release Notes
description: RadiantOne IDO Release Notes
---

# Identity Observability Release Notes

December, 2025

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.
These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements](#improvements)
- [Bug Fixes](#bug-fixes)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)


## New Features

- **ID-2109** – Implemented Guided Remediation in Slack feature with AIDA.  
- **ID-2291** – Added configuration interfaces for Guided Remediation in Slack in the Portal Settings.  
- **ID-2282** – Added an option when configuring Controls to activate Guided Remediation in Slack.  
- **ID-1725** – Implemented the AIDA service for Identity Observability.  
- **ID-2110** – Added MCP protocol support v1 for AIDA within the Guided Remediation in Slack feature.  
- **ID-2265** – Added an activation button for SSF CAEP when configuring controls.  
- **ID-2096** – Provided all required services for CAEP notifications.  
- **ID-2256** – Added URL links to Identity Observability in notifications to ease further investigation within IDO.  
- **ID-2216** – Added data-visualization swimlane representations starting from groups and accounts in the Explorer.  
- **ID-2119** – Added group member management capabilities in group and account detail pages (add/remove accounts from groups).  
- **ID-2120** – Added a new remediation method: Remediation Notifications to third-party systems (Slack, Teams, ServiceNow, etc.).  
- **ID-2111** – Added remediation policy configuration interfaces in the Identity Observability Portal for Remediation Notifications.  
- **ID-2105** – Updated existing home pages for Line Manager, Repository Manager, Resource Manager, and Technical Administrator roles.
- **ID-2331** – Added new home pages for Technical Account Manager, Group Manager, and User roles.


## Improvements

- **ID-2232** – Standardized terminology for observations to consistently use Enabled/Disabled Observations in the user interface.  
- **ID-2231** – Standardized button styling for observations in the user interface.  
- **ID-2251** – Standardized terminology used when assigning a manager during remediation.  
- **ID-2383** – Restricted remediation tasks in the Umbrella API to users with the technicaladmin role.  
- **ID-2323** – Moved Identity Picker modal scrollbar inside the table when selecting an identity from a pop-up.  
- **ID-2155** – Updated IDP Console, Identity Observability Portal, and Data Sync Config login pages to align with Radiant Logic branding.  


## Bug Fixes

- **ID-2092** – Fixed an error when sorting the Account Type column that caused data loading failures.  
- **ID-2065** – Corrected incorrect counters for Number of Direct Members and Number of Total Members in departments.  
- **ID-2113** – Fixed visibility cone violations allowing Resource Manager and Identity Manager roles to view unauthorized identities in Explorer.  
- **ID-2189** – Fixed visibility cone issues for departments without managers.  
- **ID-2202** – Fixed alert payloads to correctly update control/observation names after modification.  
- **ID-2233** – Fixed duplicated control positioning and restricted duplication to custom controls only.  
- **ID-2252** – Restored missing departments in landing page counters and Explorer menu.  
- **ID-2273** – Fixed error when unchecking Only Direct Group Access in the Groups tab of Resource Detail pages.  
- **ID-2266** – Fixed service and technical accounts incorrectly displayed as orphaned in Query Builder.  
- **ID-2279** – Fixed non-functional Save button when re-editing alerts on custom observations.  
- **ID-2281** – Fixed unrelated objects appearing in the issues side panel on Department Detail pages.  
- **ID-2292** – Fixed empty Last Update column in Observation Detail pages.  
- **ID-2330** – Corrected affected item count when deleting multiple tags in Tag Management.  
- **ID-2315** – Fixed display issues with gauge components in Custom Dashboards (Mashup).  
- **ID-2300** – Fixed column display issues when switching object types in Query Builder.  
- **ID-2288** – Fixed inability to reuse the same tag label across different object types and observations/controls.  


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center/Knowledge Base accessible from:  
https://support.radiantlogic.com  

If you do not have a user ID and password to access the site, please contact:  
support@radiantlogic.com
