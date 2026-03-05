---
title: RadiantOne IDO Release Notes
description: RadiantOne IDO Release Notes
---

# Identity Observability Release Notes

March, 2026

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.  
These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements](#improvements)
- [Bug Fixes](#bug-fixes)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)


## New Features

- **IDO-193, IDO-141** – Introduced the first version of the MCP server for identity and account visibility, enabling agentic AI to query real-time identity, account, context, and issue data in IDO.

- **IDO-117** – From the Identity Observability home page (real-time dashboard), users with assigned User Access Reviews now see a direct link to their active review tasks.

- **IDO-143** – Remediation actions are now available for custom controls directly from the interface.

- **IDO-231** – Remediation by persona is available from the interface for all relevant personas.

- **IDO-94** – Application and permission detail pages now display the hierarchy of permissions in the permission relationship tables.


## Improvements

- **IDO-139** – Audit & Compliance updated to align with Identity Analytics 3.5, ensuring consistency with the latest IDA capabilities.

- **IDO-101** – The actions available in guided remediation via Slack are now aligned with the actions offered in the frontend.

- **IDO-100** – Guided remediation in Slack has been optimized to significantly improve AIDA response times when users submit questions.

- **IDO-46** – AIDA logging has been improved to comply with the logging capture policy.

- **IDO-47** – Kubernetes probes have been added for the AIDA service to improve health and readiness monitoring.

- **IDO-48** – A LiteLLM Proxy has been added to the AIDA stack to improve scalability and performance.

- **IDO-238** – Font size issues have been corrected when configuring notifications at control and observation levels.

- **IDO-161** – The reconciliation_type attribute is now used in the query builder widget for control observations.

- **IDO-160** – The new reconciliation_type attribute is fully leveraged in the model and UI to refine reconciliation-based use cases.

- **IDO-145** – The graph database dataset used by the dev team and in samples has been updated with correct reconciliation data.

- **IDO-144** – The observation portal endpoint has been extended to handle remediation information, enabling richer remediation flows.

- **IDO-140** – Licenses used in EOC for IDO have been updated to reflect the current feature set.

- **IDO-138** – Umbrella-API error details are now displayed in context pop-up messages for better troubleshooting.

- **IDO-161, IDO-160, IDO-145** – Overall reconciliation and control observation capabilities have been refined for more accurate investigations.

- **IDO-222** – A new Grafana dashboard has been added for Flink pipelines, improving monitoring and observability.

- **IDO-49** – The AIDA service for User Access Reviews has been added and activated in the license for Audit & Compliance.

- **IDO-306** – A dedicated AIDA deactivation flag is now handled consistently so AIDA behavior matches its activation state.

- **IDO-215** – The control browser has been re-enabled in the Audit & Compliance menu for easier navigation of controls.


## Bug Fixes

- **IDO-296** – Fixed an issue where only 100 controls could be displayed in the control UI, improving scalability for large control sets.

- **IDO-253** – Fixed the layout so the button to configure breakdown for custom controls is visible without having to scroll excessively.

- **IDO-242** – Corrected the label on the “Disable all accounts” remediation button to accurately reflect its action.

- **IDO-239** – Corrected the identity/disable_all_accounts remediation action to reference account_disabled instead of account_enabled.

- **IDO-213** – Reduced redundant validation name queries in Control Configuration, improving performance.

- **IDO-164** – Fixed refresh behavior and wording issues in the remediation flow to update account type from the account detail page.

- **IDO-163** – Fixed an issue with Realtime database to Audit & Compliance database synchronization so pre-existing review data is handled properly.

- **IDO-159** – Fixed the import of custom control/observation configurations so TAGS are now correctly taken into account.

- **IDO-100** – Addressed performance problems in guided AIDA remediation via Slack, speeding up answers from AIDA.


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base accessible from:  
https://support.radiantlogic.com  

If you do not have a user ID and password to access the site, please contact:  
support@radiantlogic.com
