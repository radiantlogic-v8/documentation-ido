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

- **ID-2384** – Added persona-based authorization for remediation actions in the Umbrella API, ensuring only authorized personas can perform remediation (UI support planned for next release).  
- **ID-1731** – Extended Query Builder to support bulk sensitivity and reason updates based on query results, simplifying large-scale data classification.


## Improvements

### Review Campaigns and Audit & Compliance
- **ID-1985** – Aligned statuses and user experience for post-GA review flows.  
- **ID-2446, ID-2507** – Improved Audit & Compliance app behavior and navigation, including control menu cleanup.  

### Review Campaign Management and Review Pages
- **ID-2453, ID-2455, ID-2510** – Ensured Review Campaign Management lists refresh correctly after create/edit/delete operations.  
- **ID-2484** – Fixed review campaign edition wizard to display the current campaign status.  
- **ID-2448, ID-2454** – Addressed layout issues on review page and campaign management headers.  

### Remediation and Writeback Flows
- **ID-2244, ID-2240** – Improved remediation flows to update account type directly and reuse created technical account purposes.  
- **ID-2091** – Ensured UI consistency when backend remediation descriptions are cleared.  
- **ID-2422** – Introduced dynamic updates of defect context (managers).  
- **ID-2418** – Hardened Entra ID writeback of the account enabled attribute.  

### Detail Pages and Landing Experience
- **ID-2364** – Added missing email attribute to Account Detail pages.  
- **ID-2350, ID-2336** – Improved group detail hierarchy and managed resource display.  
- **ID-2363** – Aligned identity and account setup between IDO and IDA.  
- **ID-2308** – Standardized defect counters across detail pages.  
- **ID-2252, ID-2295** – Improved wording and counters for departments and managers.  
- **ID-2344** – Refined wording when adding account descriptions.  

### Observation, Audit Trail, and UI Polish
- **ID-2230** – Made the “New issues” indicator available across all Observation views.  
- **ID-2154, ID-2480** – Improved audit trail display and navigation using entity display names.  
- **ID-2407** – Improved Group Managers tab display for departments.  
- **ID-2447** – Fixed header and active role display overlap issues.  
- **ID-2307** – Corrected inverted alert test configuration messages.  
- **ID-2420** – Standardized date display to MM/DD/YYYY where required.  

### Platform and AI Handling
- **ID-2345** – Prevented Flink pipelines from getting stuck after NATS disconnections during deployment.  
- **ID-2421** – Improved handling of LLM responses returned in table format.


## Bug Fixes

### Review Campaigns and Audit & Compliance
- **ID-2451, ID-2453, ID-2510** – Fixed multiple issues with campaign creation, saving, and deletion.  
- **ID-2453, ID-2455, ID-2448** – Ensured review lists and pages refresh correctly after actions.  
- **ID-2484** – Corrected campaign status display in the edition wizard.  
- **ID-2507, ID-2446** – Cleaned up Control Menu behavior and UI inconsistencies.  

### Remediation and Account Management
- **ID-2481** – Fixed “Mark as processed” action in Account Details remediation.  
- **ID-2240** – Restored reuse of technical account purposes during remediation.  
- **ID-2091** – Fixed UI behavior when clearing backend descriptions.  
- **ID-2418** – Corrected Entra ID enabled/disabled writeback reliability.  

### Detail Pages, Audit Trail, and Counters
- **ID-2350, ID-2336** – Fixed duplicate groups and incorrect hierarchies in detail pages.  
- **ID-2252, ID-2295** – Corrected department counters, visibility, and wording.  
- **ID-2344** – Fixed wording and behavior when adding account descriptions.  
- **ID-2308** – Corrected defect counter inconsistencies.  
- **ID-2480, ID-2154** – Fixed audit trail navigation and entity display names.  

### UI, Observations, and Platform
- **ID-2448, ID-2454, ID-2447** – Resolved layout and header overlap issues.  
- **ID-2407** – Corrected Group Managers tab display.  
- **ID-2307** – Fixed inverted alert test messages.  
- **ID-2230** – Fixed availability of the “New issues” indicator across Observation views.  
- **ID-2422** – Ensured defect context updates dynamically.  
- **ID-2345** – Fixed Flink pipelines stuck after NATS disconnections.


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center/Knowledge Base accessible from:  
https://support.radiantlogic.com  

If you do not have a user ID and password to access the site, please contact:  
support@radiantlogic.com
