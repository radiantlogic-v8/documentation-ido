---
title: RadiantOne IDO v1.1.0 Release Notes
description: RadiantOne IDO v1.1.0 Release Notes
---

# Identity Observability v1.1.0 Release Notes

November, 2025

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability v1.1.0.
These release notes contain the following sections:

- [New Features](#new-features)
- [Improvements](#improvements)
- [Security Vulnerability Fixes](#security-vulnerability-fixes)
- [Bug Fixes](#bug-fixes)
- [Known Issues](#known-issues)
- [How to Report Problems and Provide Feedback](#how-to-report-problems-and-provide-feedback)

> In this version, the application upgrade in Environment Operations Center supports porting of **sample (static demo) data only** provided by the application. For deployments using live data and connectors, you must create a new application instance with this version and manually copy the configurations from the original instance.


## New Features

- **ID-2087** – Extended notifications to support built-in controls and observations.  
- **ID-2099** – Added accounts and group data visualizations to the Explorer (graph and swimlane views).  
- **ID-2151** – Added user interface feedback showing data upload status for Realtime to Audit and Compliance.  
- **ID-2120** – Made remediation policies configurable by technical administrators through the Identity Observability Portal.  
- **ID-1835** – Implemented Tag Management user interface.  

## Improvements

- **ID-2088** – Added group hierarchy to relationship tables in detail pages.  
- **ID-2098** – Added hierarchy visualization for group and department relationship tables in detail pages.  
- **ID-2130** – Adjusted “Include Disabled Accounts” flag behavior on Identity Detail pages.  
- **ID-2170** – Added restrictions that prevent export and duplication of built-in observations and controls.  
- **ID-2185** – Added restrictions that prevent creation of observations and controls with identical names.  
- **ID-2246** – Reorganized observation/control actions.  
- **ID-2102** – Stabilized settings for Realtime to Audit and Compliance.  
- **ID-2152** – Improved automatic purge configuration behavior in Realtime to Audit and Compliance.  
- **ID-2213** – Enhanced metadata handling to prevent duplicate key issues during data upload in Realtime to Audit and Compliance.  
- **ID-2097** – Improved handling of Reconciliation Type in sync pipeline.  
- **ID-2191** – Improved performance of Query Builder display for controls and observations.  
- **ID-1979** – Home Page design improvements.  
- **ID-1983** – Audit Trail design improvements.  
- **ID-2060** – Query Builder table improvements for all object types.  

## Bug Fixes

- **ID-1917** – Fixed unusable export for “Number of Active vs Resolved Issues” graph in control detail page.  
- **ID-1992** – Restored missing shield icon when launching remediation.  
- **ID-2085** – Fixed regression causing alert notification creation to fail.  
- **ID-2175** – Fixed observations display status to correctly reflect enabled/disabled state.  
- **ID-2217** – Restored ability to set alerts on new or modified controls/observations.  
- **ID-2260** – Fixed breakdown graph on Control Detail page to display immediately without refresh.  
- **ID-2019** – Fixed back button issue that erased filters in Query Builder.  
- **ID-2058** – Resolved table filter issues in Query Builder.  
- **ID-2090** – Fixed broken links for “Repository” and “Permission” entities.  
- **ID-2112** – Restored search by permission name and repository name.  
- **ID-2160** – Fixed crash when sorting rule results by account types.  
- **ID-2187** – Corrected organization display to include all, not just those with managers.  
- **ID-2064** – Fixed landing page and Query Builder issues for Line Manager role.  
- **ID-2086** – Fixed counter links in landing page that previously directed users to empty Query Builder.  
- **ID-2093** – Fixed empty “Top 6 identities and departments” in Line Manager view.  
- **ID-2189** – Fixed visibility for departments without managers.  
- **ID-2252** – Fixed missing empty departments from landing page counter and explorer.  
- **ID-2078** – Fixed missing “Account Type” column in “Access Rights” tab.  
- **ID-2188** – Fixed failure (404 error) when disabling an account.  
- **ID-2192** – Corrected cancel behavior when configuring or canceling technical/account owner operations.  
- **ID-1460** – Fixed issue where adding a description to a critical group removed its sensitivity level and reason.  
- **ID-2015** – Fixed issue where “Resource without description” remediation stayed “In Progress.”  
- **ID-2177** – Corrected inconsistent spelling in remediation filters and tables.  
- **ID-2250** – Fixed “New Managers” column to show display name instead of UID during remediation.  
- **ID-2076** – Fixed search filter behavior in Audit & Compliance Identity Search.  
- **ID-2082** – Removed unnecessary Datasource Management menu from Audit & Compliance.  
- **ID-2197** – Fixed Audit Trail link to correctly open modal instead of redirecting.  
- **ID-2245** – Improved alert channel deletion logic to validate usage or display info before deletion.  
- **ID-2195** – Fixed regression where tables with functional fields stopped working.

## Known Issues

For known issues reported after the release, please see the Radiant Logic Knowledge Base: 
https://support.radiantlogic.com/hc/en-us/categories/4412501931540-Known-Issues

## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center/Knowledge Base accessible from: https://support.radiantlogic.com
If you do not have a user ID and password to access the site, please contact: support@radiantlogic.com

