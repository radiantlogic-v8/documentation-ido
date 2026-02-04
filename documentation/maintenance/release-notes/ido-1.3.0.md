---
title: RadiantOne IDO Release Notes
description: RadiantOne IDO Release Notes
---

# Identity Observability Release Notes

February, 2026

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.  
These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements](#improvements)
- [Bug Fixes](#bug-fixes)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)


## New Features

- **ID-1731** – Extended the Query Builder to allow bulk updates of sensitivity levels and reasons based on query results, simplifying large-scale data classification.  
- **ID-2384** – Introduced persona-based authorization for remediation actions in the Umbrella API, ensuring only authorized personas can perform remediation (end-user interface support planned for the next release).


## Improvements

- **ID-1985** – Aligned statuses across Identity Observability for risk level, sensitivity level, and priority to ensure consistency across interfaces. 
- **ID-2154** – Improved the Audit Trail to display the affected entity’s display name instead of the technical key.  
- **ID-2230** – Made the “New issues” indicator available across all Observation views for improved visibility.  
- **ID-2240** – Improved remediation behavior to allow reuse of created technical account purposes.  
- **ID-2244** – Improved the remediation flow to allow updating the account type directly from the Account Detail page.  
- **ID-2336** – Improved display of managed technical accounts in the Identity Detail Managed Resources tab.  
- **ID-2344** – Improved the remediation flow when adding a description to an account.  
- **ID-2345** – Improved platform resilience by preventing Flink pipelines from getting stuck after a NATS disconnection during deployment.  
- **ID-2363** – Aligned the setup identity and account used between Identity Observability and Identity Analytics.  
- **ID-2364** – Added the missing email attribute to the Account Detail page.  
- **ID-2407** – Improved the display of the Group Managers tab for departments.  
- **ID-2420** – Standardized date display to US format (MM/DD/YYYY) where required.  
- **ID-2421** – Improved handling of LLM responses returned in table format to ensure consistent consumption.  
- **ID-2422** – Introduced dynamic updates of defect context (managers) to keep remediation context in sync.  
- **ID-2446** – Improved overall behavior and UI consistency in the Audit & Compliance application.  
- **ID-2448** – Improved the Review Campaign Management header layout.  


## Bug Fixes

- **ID-2091** – Fixed an issue where clearing the description field in the backend did not clear the value in the UI.  
- **ID-2252** – Fixed missing departments in landing page counters and the Explorer.  
- **ID-2295** – Fixed incorrect wording on Department Detail pages when a department has no manager.  
- **ID-2307** – Fixed inverted alert test configuration messages.  
- **ID-2308** – Fixed inconsistencies in defect counters across detail pages.  
- **ID-2350** – Fixed duplicate and incorrect group hierarchies in the Group Detail page.  
- **ID-2418** – Fixed intermittent Entra ID writeback issues for the account enabled attribute.  
- **ID-2447** – Fixed overlap issues between page headers and the active role display.  
- **ID-2451** – Fixed failures occurring when saving newly created review campaigns.  
- **ID-2453** – Fixed Review Campaign Management list not refreshing after create, edit, or delete actions.  
- **ID-2454** – Fixed broken layout on review page headers.  
- **ID-2455** – Fixed Review Page actions that did not refresh the page.  
- **ID-2480** – Fixed navigation issues originating from the Audit Trail.  
- **ID-2481** – Fixed an issue where clicking “Mark as processed” in Account Details remediation had no effect.  
- **ID-2484** – Fixed the Review Campaign edition wizard to correctly display the current campaign status.  
- **ID-2508** – Fixed an error preventing creation of reviews in the Audit & Compliance application.  
- **ID-2510** – Fixed an issue where deleting a review campaign did not complete and looped continuously.  


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base accessible from:  
https://support.radiantlogic.com  

If you do not have a user ID and password to access the site, please contact:  
support@radiantlogic.com
