# Identity Observability Release Notes

**May, 2026**

These release notes contain important information about new features, improvements and bug fixes for RadiantOne Identity Observability.

These release notes contain the following sections:


## New Features

* **IDO-400** – Enabled Role Mining interfaces in Audit & Compliance, allowing administrators to analyze and refine roles based on current access patterns.

* **IDO-523** – In Audit & Compliance, delegated remediation for identities is now supported directly from the Remediation Management screens, and Role Mining is available to help you analyze and refine roles.

* **IDO-580, IDO-612, IDO-613, IDO-614, IDO-639** – Expanded the Security Event Hub with new APIs to list and check the status of external controls, configurable sink templates that dynamically generate external controls, additional attributes on controls and control defects (origin, source, comment), and the ability to enable or disable sources, sinks, and sink templates for finer-grained event flow control.


## Improvements

* **IDO-323** – Technical administrators can now re-run a duplicate identity analysis on a new data snapshot. The interface shows the date of the last run, disables the button when no new snapshot is available, highlights newly added identities in clusters, and carries forward prior remediation and decision statuses.

* **IDO-520** – Improved the remediation preview in Audit & Compliance Duplicate Identity so that users can review all pending actions before applying them and selectively cancel individual actions.

* **IDO-533** – The Query Builder account-type filter now displays "Orphaned" instead of an empty label when filtering for orphaned accounts, improving usability across entity detail and relationship tables.

* **IDO-543** – Decision comments added during duplicate identity remediation are now displayed on the duplicate identity page, giving reviewers visibility into prior justification.

* **IDO-544** – Added action-specific explanatory text for each remediation action (Update, Deactivate, Mark as Namesake, Mark as Exception) when remediating duplicate identities, replacing the previous generic prompt.

* **IDO-546** – Repositioned the Duplicate Identity entry to the end of the Review menu in Audit & Compliance so that more commonly used review options appear first.

* **IDO-550** – Optimized Query Builder performance for searches that significantly reduce the result set. Searches such as "accounts having access to a resource named X" that previously took 20–30 seconds on large databases now complete in under 0.5 seconds.


## Bug Fixes

* **IDO-54, IDO-316, IDO-590, IDO-591** – Fixed several AIDA and User Access Review issues: AIDA logs for UAR are now properly captured, the AIDA co-pilot starts reliably, the AIDA switch icon is displayed when the service is enabled, and AIDA options are correctly hidden when the feature flag is deactivated.

* **IDO-74, IDO-76, IDO-77, IDO-78** – Fixed Realtime-to-Audit & Compliance data upload so that metadata counts (number of accounts per permission, number of objects per department, number of applications per identity, and number of objects per application) are correctly computed.

* **IDO-121, IDO-126** – Fixed Audit & Compliance navigation issues: clicking the back button from the Review Page to access the 360 view no longer causes a temporary error, and the Review Campaign Management progress bar now reflects the correct status on initial page load.

* **IDO-219** – Updated trend wording so that new issues display "no new issues" instead of the confusing "no issues" label.

* **IDO-236** – Corrected the application access rights review perimeter in Audit & Compliance so that the number of Accounts and Identities are accurate.

* **IDO-245** – Line Managers' remediation buttons on accounts are now correctly disabled as intended.

* **IDO-308, IDO-327** – Fixed custom control remediation: attribute lists are now updated by entity type and remediation dialogs correctly display content for all action types.

* **IDO-332** – Removed the ability to add or remove tags from point-in-time interfaces in Audit & Compliance, since tags are managed exclusively in real-time interfaces.

* **IDO-413** – Updated the Identity Detail Page Managed Resources tab to display the expected columns and information consistent with the non-storage v2 version.

* **IDO-515** – Fixed the Query Builder account-type global filter so that filter values match the values displayed in the table.

* **IDO-512, IDO-519, IDO-539, IDO-541, IDO-542, IDO-545** – Addressed multiple UI issues in the Duplicate Identity pages: primary identity actions can now be removed, the "Apply Remediation" button is no longer available on the Compare page, birthdate is no longer incorrectly highlighted, dashboards are shared among technical administrators, primary identity selection enforces multi-identity comparison, and selection checkboxes no longer remain stuck in a checked state.

* **IDO-547, IDO-548** – Corrected the "Pick an active manager" dialog to show the missing department column and fixed failures when adding an identity as a manager to remediate missing-manager control defects.

* **IDO-574** – Fixed job and department aggregation for identity search results.

* **IDO-585** – Accounts with more than one matching identity are no longer incorrectly flagged as orphaned.

* **IDO-594, IDO-595, IDO-604** – Fixed multiple issues affecting the duplicated identities workflow in Audit & Compliance: remediation strategy is now properly carried over between timeslots, additional column display issues in cluster dashboards are resolved, and remediation tickets are correctly created.

* **IDO-626, IDO-632** – Fixed duplicate-row display bugs in Technical Accounts observations and Group Control defect tables so that each object appears only once.

* **IDO-628, IDO-629** – Resolved data-loading errors when navigating from the landing page to the remediation page and when configuring real-time mashup dashboards on the new storage v2 database schema.

* **IDO-633, IDO-635** – Corrected access-control issues where resource owners received a data-loading error when viewing their permissions and where unprivileged users could see all accounts and identities.

* **IDO-640** – Review campaigns can now be finalized even when the initial starting timeslot has been purged.

* **IDO-651** – Fixed the external remediation email process so that already-sent items are no longer resent on subsequent runs.


## How to Report Problems and Provide Feedback

Feedback and problems can be reported from the Support Center / Knowledge Base:

[https://support.radiantlogic.com](https://support.radiantlogic.com)

If you do not have a user ID and password, contact:

[support@radiantlogic.com](mailto:support@radiantlogic.com)
