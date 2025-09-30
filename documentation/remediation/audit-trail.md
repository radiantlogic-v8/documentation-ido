---
title: Remediation Audit Trail 
description: Audit who performed remediation actions. 
---

## Remediation Audit Trail  

 Remediation Audit trail records all end-user activities in Identity Observability related to remediation, such as attribute updates, accounts disabled etc., providing traceability for data modification. 

The Audit Trail page can be accessed through the admin settings page, control detail page as well as object detail page.  

It provides details such as the action taken, who performed the action, the affected entity, the time of occurrence, and the action’s status.  

 
1. Technical administrators can access this page by clicking on “Admin > Settings > Audit Trail”. This will list all actions performed through Identity Observability.  

  ![Image showing navigation to audit trail](Media/audittrail-menu.png "Image showing navigation to audit trail")

 
2. Non admins users and technical administrators can view this page by navigating to a control from the Controls page and clicking the “View Audit Trail” link in the top left corner. 

  ![Image showing navigation to audit trail](Media/audit-trail-controls.png "Image showing navigation to audit trail from Controls page")

This lists the audit trail related to remediation of a specific control.


3. Non admin users and technical administrators can also view the Audit Trail by:
    * Clicking on an object to open its detail page. Example:

     ![Image showing a clickable object](Media/objname.png "Image showing a clickable object")
    
    * Then, clicking on the (…) menu and selecting “View Audit Trail”. Example:

     ![Image showing the View Audit Trail link](Media/obj-audittrail.png "Image showing the View Audit Trail link")


### Understanding Audit Trail  

| Field            | Description                                                                                                                                  | Examples / Possible Values                                                                                                                                                                                                 |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Remediation Value | Type of action performed, displayed in one of the following formats:<br>• entity/remediation action (for control issue remediation)<br>• entity/attribute (for attribute updates)<br><br>Refer to the remediation code listed in the remediation table above. | Account/disable<br>Identity/updateEmail                                                                                                                                                                                     |
| Entity Type      | The classification of the entity subject to remediation.                                                                                     | Accounts, Identities, Departments, Resources, Repositories                                                                                                                                                                  |
| Affected Entity  | The name and unique identifier (ID) of the impacted entity.                                                                                   | Account: 12345<br>Identity: jdoe                                                                                                                                                                                            |
| Action Status    | Indicates the progress or result of the remediation action (distinct from issue status).                                                     | error – Action failed due to an execution error<br>cancel – Action was canceled before completion<br>running – Action is in progress<br>submitted – Action was submitted but not yet started<br>completed – Action finished successfully |



### Filtering Audit Trails  

An audit trail can be filtered by dates so you can specify the date range for which the trail should be displayed. 

  ![Image showing dates based audit trail filtering](Media/filter-audittrail.png "Image showing audit trail filtering based on dates")

You can also filter the audit trail by:  

* actor (user who performed the remediation action)
* modified object ( the account, the identity …) 
* remediation action type 
* entity type 
* affected entity 
* status 

  ![Image showing audit trail filtering](Media/filter-search.png "Image showing audit trail filtering")

Optionally, you can enable the "Group by Remediation Bulk Actions" option to reveal which remediation operations were performed in bulk. 

  ![Image showing grouping option for remediation actions](Media/groupby-bulk.png "Image showing grouping option for remediation actions")


By clicking the “View Details” option under Action, you can find additional technical details about that action (such as details about a timeout status or an error status). 