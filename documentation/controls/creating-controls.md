# Controls
 
This document provides an overview of Observations and outlines the steps to create and view observations. 

## Overview 
 
Controls provide automated detection mechanisms to identify potential security risks and compliance violations within the identity infrastructure. Users can enable relevant controls for their specific context or create their own custom controls. 

A control represents an issue with a risk level, risk description and a remediation plan to fix it. Control risk levels are used to compute the risk score of an item (Accounts, Identities, Departments, and More). 

Like observations, controls are based on a rule that contains the criteria of what needs to be monitored in real time. The result of the rule which is a list of items is maintained in real time. Those controls can be used in out-of-the box or custom dashboards. Notifications can also be activated by control to notify people when a change occurs in the list of items, meaning new items are returned or removed from the list by the rule. 
 
## Default controls 
 
Default controls are pre-configured security and compliance checks designed to identify and manage risks in your identity environment. These controls help enforce best practices by automatically monitoring for potentially dangerous situations, enabling you to catch issues proactively.  
 
### Examples 

The control "Account with abnormal login attempts" alerts you to accounts with repeated unsuccessful login attempts, flagging them for possible brute force attacks. 

"Contractor with past ending date and active accounts" highlights cases where contractors who have left the company still possess active accounts, which could be exploited for unauthorized access.  
 
Another common default control is "MFA is not registered," which identifies accounts that are missing multi-factor authentication, exposing them to higher risk of takeover. 
 
You can enable or disable any of these default controls. You can also create your own control by following the steps below.  

## Creating a new control 

1. Click the Control menu item on the left navigation.  This will open the Control management interface. 
 

2. Select a category for which you want to build the control. Define one or more criteria for the control. Suppose you want to create a control for Accounts that last logged in before a certain date. To do so, select the “Account” category from the dropdown and set the criteria to a certain date such as “before 05/20/2025.”  

3. Click “Apply” to review and then click “Next” to proceed through the remaining steps. 
 
4. On the Name, Description & Status tab, provide a suitable name and description for the control. Choose the relevant control family from Authentication, Identity Lifecycle, Privilege and Access, or Identity Hygiene. 

Then, choose whether you want the control to be active immediately or saved in a deactivated state for future use.  

5. On the Risk Assessment page, select a risk level that you think is appropriate for this control, provide description for this risk and include clear actionable steps for remediating this risk.  
 
6. On the Breakdown configuration tab, you can add additional filters or criteria for your control by clicking the Configuration Breakdown option and selecting the configurations that you would like to add. This step is optional. Click “Skip for now” to skip this step.  
 
7. On the Alert Configuration Tab, activate and configure alerting as needed by:

* Specifying the triggering events, such as when a new item is detected, when an item is removed from the list, or both.

* Setting grouping parameters, for example grouping notifications every 30 minutes, 1 hour, 4 hours, or daily.

* Activating protection to prevent too many notifications from being sent in a short period of time.

* Selecting the delivery channel, such as email, Slack, Teams, or webhook, and configuring the recipients for notifications.
 

8. On the Setting Visible Attributes tab, select the attributes that you want to display or hide in the Controls table by clicking on the “eye” icon next to the attribute name. If you do not see a desired attribute in the displayed list, click Advanced settings to add additional attribute(s).  Click Next after selecting the attributes. 
 

9. After making all the desired changes, select “Submit”. To view the control you just created, navigate to Controls> My Controls and click the control name.  
 
