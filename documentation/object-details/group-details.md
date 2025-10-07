---
title: Group Details 
description: Navigating and Understanding Group Details
---

## Overview

Groups organize collections of accounts for access control and security policy enforcement. The group detail page displays details on group membership, assigned permissions, and management settings.  


## Key Attributes  

The following attributes related to a group are displayed in the first part of the group detail interface.

| Attribute | Description |
|-----------|-------------|
| Code | Unique group identifier. |
| Group Name | Display name of the group. |
| DN | Distinguished Name of the group. |
| Description | Purpose and function of the group. |
| Last Modified | Date of the most recent modification to the group. |
| Tag | Identity observability tags, added by end-users. |
| Sensitivity Level & Reason | Sensitivity classification of the object, either defined by a data source or by an end-user, with an explanation for the assigned level. |
| Repository | Source repository name, with link to the HR repository detail page. |



## Status Attributes  

In the Status section, you can view following information about a Group.


| Attribute | Description |
|-----------|-------------|
| Group Type | Classification such as role, profile, or other type. |
| Dynamic | Indicates whether the group is dynamic or static. |
| Created | Group creation date. |
| Last Modified | Date of the most recent modification to the group. |

## Relationship Tables  

### Identities  

The Identity table shows identities that include accounts as members of a group. The attributes of the Identity table are described below.  

| Attribute | Description |
|-----------|-------------|
| Full Name | First and last name of the identity. |
| Identity Priority | Identity risk score, expressed as a priority level. |
| Email | Contact email address. |
| Active | Employment status: active if currently employed, inactive if no longer with the organization. |
| Identity Type | Identity classification: true for employee, false for contractor. |
| Job Title | Current job/position title. |
| Department | Affiliated department, with link to the department detail page. |
| HR Repository | HR repository display name, with link to the repository detail page. |

By default, the table displays only identities that are directly assigned to the group. To include identities from subgroups, clear the “Direct Assignment Only” checkbox above the table.

### Accounts  

| Attribute | Description |
|-----------|-------------|
| Login | Login identifier. |
| Account Name | Display name of the account, with link to the account detail page. |
| Account Priority | Account risk score, expressed as a priority level. |
| Last Login Date | Most recent login date. |
| Disabled | Indicates whether the account is enabled or disabled. |
| Repository | Source system (e.g., AD, Entra ID). |
| Account Type | Classification: orphaned, service, or user account. |
| Identity Full Name | First and last name of the associated identity, with link to the identity detail page. |
| Identity Email | Email address of the associated identity. |
| Identity Active | Employment status of the associated identity. |
| Department | Affiliated department, with link to the department detail page. |

By default, the table displays only enabled accounts that are directly part of the group.

To include accounts from subgroups, clear the “Direct Group Membership Only” checkbox.
To display disabled accounts, clear the “Include Disabled Accounts” checkbox. 

### Parent and Child Groups  

| Attribute | Description |
|-----------|-------------|
| Group | Display name of the group. |
| DN | Distinguished Name of the group. |
| Group Priority | Group risk score, expressed as a priority level. |
| Description | Purpose and function of the group. |
| Last Modification Date | Date of the most recent modification. |
| Repository | Source repository, with link to the HR repository detail page. |

When the “Direct Group Membership Only” checkbox is unchecked, you can view all child groups in the Child Groups tab, or all parent groups in the Parent Groups tab, based on the group hierarchy.  

### Resources  

| Attribute | Description |
|-----------|-------------|
| Resource | Display name of the resource, with link to the resource detail page. |
| Resource Priority | Resource risk score, expressed as a priority level. |
| Description | Purpose and function of the resource. |
| Resource Type | Type of resource (e.g., file share, server, application). |
| Resource Family | Family of the resource (e.g., SharePoint, AD, LDAP). |


### Permissions  

| Attribute | Description |
|-----------|-------------|
| Permission | Access right name, with link to the permission detail page. |
| Permission Priority | Permission risk score, expressed as a priority level. |
| Description | Purpose and function of the permission. |
| Permission Type | Type of permission (e.g., role, profile, right). |
| Permission Family | Family of the permission (e.g., NTFS, OneDrive, SharePoint). |
| Resource | Target system or application, with link to the resource detail page. |

The permissions table shows all permissions from the group and its subgroups by default. To see only the permissions assigned directly to this group, check the 'Direct Assignment Only' box.  


## Group Owners  

| Attribute | Description |
|-----------|-------------|
| Full Name | First and last name of the owner, with link to the identity detail page. |
| Identity Priority | Identity risk score, expressed as a priority level. |
| Email | Contact email of the owner. |
| Active | Employment status: active if currently employed, inactive if no longer with the organization. |
| Identity Type | Identity classification: true for employee, false for contractor. |
| HR Repository | HR repository display name, with link to the HR repository detail page. |
| Expertise Domain | Type of ownership (e.g., business owner, technical owner, reviewer). |
| Actions | Option to remove the listed owner. |


## Available Actions  

### For Repository Owners  
- Add or remove tags.
- Add or remove group owners associated with the repositories they own.
- Update sensitivity level and reason attributes.  
- Fix detected issues through the “Remediate Issues” side panel.

### For Technical Administrators  
- Add or remove tags.
- Add or remove group owners.
- Update sensitivity level and reason attributes.
- Fix detected issues through the “Remediate Issues” side panel.

### For Auditors  
- View only  

### For Other Roles  
- Not accessible  

