# Key Concepts of Identity Observability  

This document describes the key concepts related to **Identity Observability.  

## Data Model of Identity Observability  

The data model serves as the foundation for comprehensive identity visibility, organizing all identity-related entities and their relationships within a unified framework.  

Data from different backend systems are mapped to this data model and updated in real time.  

Here is a simplified schema of the Identity Observability data model:  

<to-add: A diagram of data model>  

This model is agnostic, which allows Identity Observability to ingest any type of data from multiple heterogeneous systems about identities, accounts, and access. It enables real-time observation in a centralized location through dashboards, custom queries, observations, and controls.  

### Objects (Accounts, Identities, Departments, and More)  

Identity Observability encompasses multiple types of objects that represent the complete identity landscape. These objects may be stored in various repositories (data sources).  

### Accounts and Groups Repositories  

In the Identity Observability data model, a **repository** represents the source that hosts objects such as accounts and groups. A repository could be a directory or identity provider (IDP), such as Active Directory, Entra ID, or LDAP directories. It may also originate from servers or applications that maintain local accounts and groups. In essence, the repository serves as the backend system where account and group data is stored.  

#### Accounts  

Accounts are used to authenticate resources such as applications, servers, and services, and enable authorization. Accounts can be of the following types:  

- **User accounts**: Accounts used by individual users.  
- **Service/technical accounts or machine identities**: Accounts used by services or applications, or shared by multiple identities.  
- **Orphaned accounts**: Accounts whose user, technical, or service association is unknown.  

#### Groups  

Groups serve as containers that bundle multiple accounts together. They allow administrators to efficiently manage permissions, access rights, and policies for multiple entities simultaneously, rather than handling each account individually.  

### Human Resources Repositories  

HR systems are modeled as “repositories” of type HR within the data structure. They contain information about identities and departments, including internal employees and contractors. Multiple HR repositories may exist, such as one for internal employees and another for contractors.  

#### Identities  

Identities represent the people who are using user accounts or shared accounts to access company resources. This concept is broader than an account—it represents **who the user is**, including all attributes and characteristics.  

#### Departments  

Departments refer to the subgroups within an organization where an identity works. An identity is linked to a department through a job title. This is represented as: “An identity is working in the department as job title.”  


### Resource Repositories  

#### Resources  

Resources include any type of asset that accounts can access, such as servers, applications, services, unstructured data, PAM safes, and more.  

#### Permissions  

Permissions define the authorizations granted to accounts, typically assigned through groups or profiles. Access rights are determined by the combination of account, permission, and resource.  


### Advanced Items  

The data model includes additional items that are not currently accessible through the user interface, but will be available in future use cases.  

- **Geographical Location**: Linked to identities; represents the location where the identity is working.  
- **Work Teams**: Linked to identities; represents the projects in which the identity is involved.  
- **Cost Center**: Linked to identities; represents the cost center to which the identity imputes.  
- **People**: Associated with identities; represents a physical person.  

Example: “Dr. Fox” may be represented as a service manager at Hospital X and a surgeon at Clinic Y. The “Dr. Fox” person is linked to two distinct identities: “Dr. Fox – Service Manager” and “Dr. Fox – Surgeon.” Each identity has its own attributes and relationships and is connected to different accounts that provide access to different resources.  

## Observations 

Identity Observability provides continuous monitoring capabilities, enabling real-time visibility into identity access to resources and changes across the entire identity infrastructure. Users can enable relevant observations for their specific context or create custom observations.  

Observations are based on rules that define what needs to be monitored in real time. The results (lists of items) are maintained in real time. Observations can be used in out-of-the-box or custom dashboards. Notifications may also be activated to alert users when changes occur, such as new items being added or removed based on the rule.  

### Built-in Observation List  

The platform includes pre-configured observations that automatically track critical identity events:  

#### Authentication and Access Observations  

- Orphaned accounts with never-expiring passwords  
- Active accounts with MFA enforced  
- Sensitive groups with new members in the last 7 days  
- Contractors' active accounts  
- Privileged accounts created in the last 7 days  

#### Identity Lifecycle Observations  
- Employees who arrived in the last 7 days  
- Employees who left in the last 7 days  
- Contractors who left in the last 7 days  
- Technical and service accounts created in the last 7 days  
- Contractors who arrived in the last 7 days  

#### Identity Hygiene Observations  
- Identity without an email address  
- Team leader without an email address  
- No sensitivity level set for a permission  
- No permission description  

## Controls to Detect Issues  

The control framework provides automated detection mechanisms to identify potential security risks and compliance violations within the identity infrastructure. Users can enable relevant controls for their specific context or create their own custom controls.  

The key difference between an **observation** and a **control** is that a control represents an issue with a defined risk level, risk description, and remediation plan. Control risk levels are used to compute the risk score of items (Accounts, Identities, Departments, and more).  

Like observations, controls are rule-based and maintained in real time. Controls can be used in out-of-the-box or custom dashboards, and notifications can be activated when changes occur (e.g., new items appear or are removed).  

### Control Families  

Controls are organized into families to streamline investigations. There are four main families:  

#### Authentication Controls  
Ensure only validated users or service accounts can access systems. They focus on secure authentication mechanisms, such as strong passwords, multi-factor authentication (MFA), lockouts after abnormal login attempts, and prevention of weak or passwordless methods.  

**Example Controls:**  
- Accounts with passwords set to never expire  
- MFA not allowed, not registered, or disabled  
- Abnormal login attempts  
- Password not required or cannot be changed  

These controls remediate vulnerabilities that could lead to account takeovers or credential abuse.  

#### Identity Lifecycle Controls  

Monitor and enforce processes for onboarding, offboarding, and role changes (joiners, movers, leavers). Ensure access is properly granted, modified, or revoked.  

**Example Controls:**  
- Active accounts for identities who have left the organization  
- Technical accounts without an assigned manager  
- Missing contractor end dates  
- Critical/sensitive resources without owners after organizational changes  

These controls prevent risks such as lingering access, insider threats, and compliance gaps.  

#### Privileged & Access Controls  

Govern the granting and review of sensitive permissions. Enforce least-privilege principles and detect excessive or unusual access.  

**Example Controls:**  
- Users with excessive permissions/groups  
- Sensitive resources without owners or descriptions  
- High-risk access not reviewed in the last 90 days  
- Direct permission assignments outside policy  

These controls protect against privilege escalation and misconfigurations while supporting compliance standards (e.g., ISO 27001, NIST).  

#### Hygiene Controls  
Focus on data quality and elimination of unnecessary, anomalous, or misconfigured entities.  

**Example Controls:**  
- Dormant or orphaned accounts/groups/resources  
- No manager/owner assigned to critical assets  
- Missing sensitivity level or descriptions for resources/groups/permissions  
- Phantom identities (no accounts correlate to an identity)  
- Data normalization issues (e.g., missing employee number, invalid email)  

Maintaining hygiene increases observability accuracy and reduces identity sprawl.  

### Control Risk Level Concept  

Controls are categorized by risk assessment and potential impact:  

- **Critical**: High-impact vulnerabilities that could cause significant breaches  
- **High**: Substantial risks requiring prompt remediation  
- **Medium**: Moderate risks, remediated within normal timeframes  
- **Low**: Minor risks, addressed during routine maintenance  

Risk levels can be used to determine prioritization and resource allocation.  

### Control Defect Status  

Each control tracks its remediation lifecycle through a finite state machine:  

<to-add: An image of control defect status>  


**Remediation Status Categories:**  
- **New**: Newly identified control violations awaiting remediation  
- **Renew**: Issues previously fixed but now reappeared  
- **False Positive**: Marked false positive by users; excluded from risk scoring  
- **Exception**: Temporarily excluded by users; excluded from risk scoring  
- **To Remediate**: Identified issues pending remediation decisions  
- **In Progress**: Remediation underway (manual or via ITSM ticket)  
- **Processed**: Marked as remediated or closed in ITSM, though still present in the backend  
- **Error**: Remediation failed; issue still present  
- **Done**: Issue resolved and no longer present  

This process ensures efficient workflow management and accountability.  


### Built-in Control List  

The platform provides comprehensive pre-configured controls covering key domains:  

- Authentication Controls  
- Identity Lifecycle Controls  
- Privileged & Access Controls  
- Hygiene Controls  

<to-add: An image of built in control list>  


## Risk Scoring to Prioritize Items  

Risk scoring provides a quantitative framework for prioritizing risks and focusing remediation efforts.  

### Items Priority Column Based on Risk Score  

- Risk scores are computed for identities and associated items.  
- Scores consider the risk levels of control defects and the sensitivity levels of items.  
- Items are ranked and labeled as *Urgent, Essential, Medium, or Low* (instead of raw numbers).  
- Prioritization adjusts dynamically in real time.  

### Risk Score Computation and Aggregation  

Risk scoring considers both **likelihood** and **impact**:  

- **Control Risk Level**: Severity of control defects  
- **Sensitivity Level**: Data sensitivity (1 = Low, 4 = Critical)  
- **Aggregated Risk Score**: Includes related items (e.g., an identity’s accounts)  

**Calculation Methodology:**  
- *Intrinsic Score*: Based on item’s defects and sensitivity  
- *Aggregated Score*: Weighted sum including related items  
- *Normalization*: Scores normalized for comparability  
- *Continuous Updates*: Adjust dynamically with new data  

Relationships are aggregated as follows:  

<to-add: An image of relationships>  

**Examples:**  
- A group’s score includes risks from its member accounts and sub-groups.  
- A department’s score includes risks from its member identities and their accounts.  


## Identity Observability User Roles  

The platform implements **role-based access control (RBAC)** with the **visibility cone principle**—users only access identity data and controls relevant to their role and clearance.  

### Role Definitions  

- **Technical Administrator (Tech Admin)**  
  - Full configuration and system management  
  - Complete visibility across IDO  
  - Can modify controls, observations, risk scoring, and user roles  

- **Functional Administrator (Functional Admin)**  
  - Business-focused administration  
  - Manages identity governance workflows  
  - Oversees compliance and reporting  

- **Auditor**  
  - Read-only dashboards and compliance reports  
  - Can export documentation and attestations  
  - No configuration rights  

- **Line Manager**  
  - Limited to direct reports and departments  
  - Department-specific dashboards  

- **Resource Owner**  
  - Manages specific apps, systems, or data  
  - Controls access and policies for those resources  

- **Repository Owner**  
  - Manages identity data repositories/directories  
  - Oversees data quality and repository monitoring  

Each role operates within its visibility cone, ensuring sensitive data remains restricted while enabling collaboration across the identity security ecosystem.  
