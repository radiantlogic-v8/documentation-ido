---
title: Repository Details 
description: Navigating and Understanding Repository Details
---

## Overview 

Repositories represent identity, account and group data sources such as Active Directory domains, Entra ID, eDirectory, IDDM or any IDP systems. The repository details page displays information about the repositories that are configured in your account.  

This document provides information about all the attributes presented in the repository detail page. 


## Key Attributes  

The following attributes related to the repository are displayed in the first part of the repository detail interface.  


| Attribute | Description |
|-----------|-------------|
| Code | Unique repository identifier. |
| Repository Name | Display name of the repository. |
| Description | Purpose and function of the repository. |
| Type | Classification of the repository (e.g., Accounts repository or HR repository). |
| Tag | Identity observability tags, added by end-users. |
| Sensitivity Level & Reason | Sensitivity classification of the repository, either defined by a data source or by an end-user, with an explanation for the assigned level. |
| Family | Family of repository (e.g., AD, LDAP, Entra ID). |


## Relationship Tables  

At the bottom of the page, several tables display objects associated with the repository.  


### Accounts  

| Attribute | Description |
|-----------|-------------|
| Login | Login identifier, with link to the account detail page. |
| Account Name | Display name of the account. |
| Account Priority | Account risk score, expressed as a priority level. |
| Last Login Date | Most recent login date (as provided from backend data). |
| Disabled | Indicates whether the account is enabled or disabled. |
| Repository | Source system (e.g., AD, Entra ID), with link to the repository detail page. |

By default, only enabled accounts are shown. Select “Include Disabled Accounts” to display disabled ones. To disable accounts, select the rows with the desired accounts and click the “Disable Accounts” button.  


### Groups  


| Attribute | Description |
|-----------|-------------|
| Group | Display name of the group, with link to the group detail page. |
| DN | Distinguished Name of the group. |
| Group Priority | Group risk score, expressed as a priority level. |
| Description | Purpose and function of the group. |
| Last Modification Date | Date of the most recent modification to the group. |
| Repository | Source repository, with link to the repository detail page. |

By default, only direct groups are shown. To include subgroups, clear the Direct Group Membership Only option.  



### Repository Managers  

The repository managers table includes a list of identities managing the repository. The attributes of this table are described below: 

| Attribute | Description |
|-----------|-------------|
| Full Name | First and last name of the identity, with link to the identity detail page. |
| Identity Priority | Identity risk score, expressed as a priority level. |
| Email | Contact email address. |
| Active | Employment status: active if currently employed, inactive if no longer with the organization. |
| Identity Type | Identity classification: true for employee, false for contractor. |
| HR Repository | HR repository display name, with link to the HR repository detail page. |
| Expertise Domain | Ownership role (e.g., business owner, technical owner, reviewer). |
| Actions | Option to remove the listed manager. |



## Available Actions  

### For Repository Owners  
- Add or remove tags  
- Add or remove repository managers  
- Update sensitivity level and reason attributes  
- Fix detected issues through the “Remediate Issues” side panel  

### For Technical Administrators  
- Add or remove tags  
- Add or remove repository managers  
- Update sensitivity level and reason attributes  
- Fix detected issues through the “Remediate Issues” side panel  

### For Auditors  
- View only  

### For Other Roles  
- Not accessible  
