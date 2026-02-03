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

- **ID-1731** – Extended Query Builder to support bulk sensitivity and reason updates based on query results, simplifying large-scale data classification.  
- **ID-2384** – Added persona-based authorization for remediation actions in the Umbrella API, ensuring only authorized personas can perform remediation (UI support planned for next release).


## Improvements

- **ID-1985** – Aligned statuses and user experience for post-GA review flows.  
- **ID-2091** – Ensured UI consistency when backend remediation descriptions are cleared.  
- **ID-2154** – Improved audit trail display so “Affected Entity” uses the entity display name.  
- **ID-2230** – Made the “New issues” indicator available across all Observation views.  
- **ID-2240** – Improved reuse of created technical account purposes during remediation.  
- **ID-2244** – Improved remediation flows to update account type directly from the account detail page.  
- **ID-2252** – Improved wording and counters for departments on landing and explorer pages.  
- **ID-2295** – Improved wording when departments have no manager.  
- **ID-2308** – Standardized defect counters across detail pages.  
- **ID-2336** – Improved managed resource display for identities.  
- **ID-2344** – Refined wording when adding account descriptions.  
- **ID-2350** – Improved group detail hierarchy to avoid duplicates and incorrect structures.  
- **ID-2363** – Aligned setup of identities and accounts between IDO and IDA.  
- **ID-2364** – Added missing email attribute to Account Detail pages.  
- **ID-2407** – Improved display of the Group Managers tab for departments.  
- **ID-2418** – Hardened Entra ID writeback of the account enabled attribute.  
- **ID-2420** – Standardized date display to MM/DD/YYYY where required.  
- **ID-2421** – Improved handling of LLM responses returned in table format.  
- **ID-2422** – Introduced dynamic updates of defect context (managers).  
- **ID-2446** – Improved Audit & Compliance app behavior and UI consistency.  
- **ID-2447** – Fixed header and active role display overlap issues.  
- **ID-2448** – Addressed layout issues on review page headers.  
- **ID-2453** – Ensured Review Campaign Management lists refresh correctly after actions.  
- **ID-2454** – Addressed layout issues on campaign management headers.  
- **ID-2455** – Improved refresh behavior after review page actions.  
- **ID-2480** – Improved navigation from the audit trail back to entities.  
- **ID-2507** – Cleaned up control menu behavior in the Audit & Compliance app.  


## Bug Fixes

- **ID-2091** – Fixed backend behavior where clearing the description field did not clear the value in the UI.  
- **ID-2154** – Fixed audit trail to display entity display names instead of raw keys.  
- **ID-2230** – Fixed availability of the “New issues” indicator across all Observation views.  
- **ID-2240** – Fixed bug preventing reuse of technical account purposes during remediation.  
- **ID-2252** – Fixed missing departments in landing page counters and explorer.  
- **ID-2295** – Fixed wording and behavior when a department has no manager.  
- **ID-2307** – Corrected inverted alert test configuration messages.  
- **ID-2308** – Fixed defect counter inconsistencies across detail pages.  
- **ID-2336** – Fixed display issues for accounts in the Identity Detail managed resources tab.  
- **ID-2344** – Fixed wording and behavior when adding account descriptions.  
- **ID-2350** – Fixed duplicate and incorrect child group hierarchies in Group Detail pages.  
- **ID-2407** – Fixed display issues in the Group Managers tab for departments.  
- **ID-2418** – Corrected Entra ID enabled/disabled writeback reliability.  
- **ID-2422** – Fixed defect context so manager information updates dynamically.  
- **ID-2446** – Fixed various UI inconsistencies in the Audit & Compliance app.  
- **ID-2447** – Fixed overlap between page headers and active role display.  
- **ID-2448** – Resolved layout issues on review pages.  
- **ID-2451** – Fixed failures during review campaign creation and deletion.  
- **ID-2453** – Fixed Review Campaign Management list not updating after actions.  
- **ID-2454** – Fixed layout issues on Review Campaign Management header.  
- **ID-2455** – Fixed review page refresh issues after actions.  
- **ID-2480** – Fixed navigation issues from the audit trail.  
- **ID-2481** – Fixed “Mark as processed” action in Account Details remediation.  
- **ID-2484** – Fixed review campaign edition wizard to show the correct campaign status.  
- **ID-2507** – Fixed Control Menu behavior in the Audit & Compliance app.  
- **ID-2510** – Fixed non-terminating delete actions for review campaigns.  
- **ID-2345** – Fixed Flink pipelines getting stuck after NATS disconnections during deployment.


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center/Knowledge Base accessible from:  
https://support.radiantlogic.com  

If you do not have a user ID and password to access the site, please contact:  
support@radiantlogic.com
```
