---
title: Data Configuration Starter Guide
description: Data Configuration Starter Guide
---

## Overview

This document provides a high-level overview of steps to configure, deploy, and monitor data sources and pipelines in Identity Observability.

## **Install Identity Observability**

Identity Observability is offered as a SaaS service via Radiant Logic's Environment Operations Center. Refer to the [installation steps guide](../installation/installation-steps/) to learn how to install and access Identity Observability.

You can access Identity Observability's services (main Identity Observability Portal, IDP console to manage users and roles and Data Config Sync to connect your data) by [logging into the application
endpoints](../installation/installation-steps/#log-in-to-identity-observability-endpoints) that appear on your application's overview page.

![Image of Data Sync Config URL](Media/config-url.png "Image of Data Sync Config URL")


## Connect to a Data Source

Login to data sync config URL using "cn=Directory Manager" as the username and the password you set during the application installation.

Once you login to the data sync config URL, you will see an interface for data configuration. This interface includes various features such as Graph Viewer, Pipeline Configuration, Connector
Configuration, and Template Configuration.

![Image of Left Navigation Pane](Media/nav-icons.png "Image of Left Navigation Pane")


The Graph Viewer visualizes data schemas with vertices, edges, and attributes, and includes a filter panel for focused analysis. Template Management uses predefined, generic templates like ad_template_v1 for AD, which can be overloaded in the pipeline but not modified directly. <br>

Pipeline Configuration manages template use, mapping, and reconciliation of data sources.

After logging into the config sync URL, navigate to Data Sources and click New to create a new data source. You can use various types of data sources:

- **LDAP or Active Directory (AD):** Easiest options for getting started.

- **Custom data sources:** Supported via **virtualization** and **periodic cache refresh** in Identity Data Management which acts as an intermediary layer.

Refer to the [Creating Entra ID Data Source](to-add) and [Creating AD Data Source]() examples to learn how to create data sources. 

## **Configure Data Pipeline** 

The next step is to configure the data pipeline to map your organization's identity data into the platform.

Keep in mind that the data will follow a specific flow as represented the schema below:

![Image of Schema](Media/data-schema.png "Image of Schema")


The following sections cover the prerequisites and steps for pipeline configuration.

### **Identify Entities**

Begin by identifying the entities you will work with. Once you've
identified your entities, familiarize yourself with the data model.
Focus on **core objects** such as accounts and groups first.

The concepts related to **JSON model representation are described
below.**

  -----------------------------------------------------------------------------
  **Concept**               **Description**
  ------------------------- ---------------------------------------------------
  **Concepts (Vertices)**   Logical groupings of tags with a main tag and
                            optional additional tags. The schema defines
                            incoming and outgoing edges.

  **Tags**                  Logical grouping of properties.

  **Edges**                 Relationships between concepts/vertices. They can
                            also hold properties.

  **Properties**            Key-value pairs associated with concepts or edges.

  **external_identifier**   The ID used to join the entity with other data
                            sources. In LDAP, this is typically the dn.

  **displayname**           Mandatory. Used as the entity's display name in the
                            Identity Observability UI.

  **dn**                    Automatically populated by the pipeline engine.
  -----------------------------------------------------------------------------

####  Identifiers

Every entity mapped in *Identity Observability* requires an **invariant ID**.

Avoid using dn (Distinguished Name) since it can change through modifyRdn operations.

Instead, use one of the following:

- **objectGUID** (for AD)

- **UUID** or **EntryUUID** (for LDAP)

- A **custom composite key** (as a last resort)

> Some attributes are mandatory and must be included in your data mapping. These attributes are listed in the tables below. 

### Identities

  -----------------------------------------------------------------------
  **attribute**                 **default value**
  ----------------------------- -----------------------------------------
  Active                        true

  Internal                      true
  -----------------------------------------------------------------------

Active and Internal flags must always be set. Default is true if no value is provided by the backend.

When identities leave the company or are no longer present in the backend, the Active status should be set to false. All identities must belong to an identity type repository.

> There is currently no standardized way to retrieve the manager, whether interpersonal or departmental. It is recommended to collect management information.

###  2. Accounts

+---------------------+-----------------+-----------+---------+-------+
| **RealTime          | **Audit         | **default | **u     | **    |
| Observability**     | &Compliance**   | value**   | nique** | non-n |
|                     |                 |           |         | ull** |
| **attribute**       | **attribute**   |           |         |       |
+=====================+=================+===========+=========+=======+
| disabled            |                 | false     |         | **X** |
+---------------------+-----------------+-----------+---------+-------+
| identifier          | Identifier      |           | **X**   | **X** |
+---------------------+-----------------+-----------+---------+-------+

The Disabled flag must always be set. Default is false if no value is provided by the backend.

The Identifier must be unique within a repository. It is recommended to use the attribute mapped to the vertex-id for the identifier.

If the Reconciliation type is set to "service", the account is considered a technical account. The attribute "reconciliation_service_reason" can be used to store the purpose of the technical account (e.g., training, testing).

The account is considered a user account if there is an "is_owned_by" edge to an identity. To verify if the identity has left the company, check the Active flag of the identity, not the account.

All accounts must belong to an account type repository.

### 3. Groups

+---------------------------+----------------------+-------+---------+
| **RealTime                | **Audit &            | **uni | **non   |
| Observability**           | Compliance**         | que** | -null** |
|                           |                      |       |         |
| **attribute**             | **attribute**        |       |         |
+===========================+======================+=======+=========+
| Name                      | Code                 | **X** | **X**   |
+---------------------------+----------------------+-------+---------+

The **Name** must be unique within a repository. It is recommended to use the attribute mapped to the **vertex-id** for the identifier. The **group type** must be populated in the pipeline.

### 4. Permissions

  --------------------------------------------------------------------------
  **IDO attribute**  **IDA attribute**  **Possible values**    **Default**
  ------------------ ------------------ ---------------------- -------------
  Type               Type               Role, Activity         Role

  --------------------------------------------------------------------------

Permission must be unique for each application. Each permission must be linked to only one application.

The permission, resource type, and family must be defined in the pipeline.

 

### 5. Resources

  -------------------------------------------------------------------------
  **IDO            **IDA            **Possible values**       **Default**
  attribute**      attribute**                                
  ---------------- ---------------- ------------------------- -------------
  Type             Type             \'Profile\', \'Server\',  Profile
                                    \'Filesystem\'            

  -------------------------------------------------------------------------

All resources must belong to a repository. The repository must be the same one that contains the accounts and groups associated with the resource.

Example: if the accounts and groups of a resource are mapped in an AD repository, the repository value must be the AD repository, not the resource.

### Repository

  -----------------------------------------------------------------------
  **IDO attribute**              **values**
  ------------------------------ ----------------------------------------
  Type                           Accounts \| Identities

  -----------------------------------------------------------------------

To leverage **MFA attributes** in accounts, the attribute mfa_enabled must be set on the repository containing the accounts.

## **3. Deploy configuration using configuration templates**

Configuration templates define **reusable components** for the orchestrator.

Refer to the [graph pipeline template configuration](./configuration/template-configuration/) and the [pipeline configuration quickstart guide](./pipeline-configuration-quickstart) for details on how to build and apply them effectively.

## **4. Monitor Configuration**

After mapping your data to Identity Observability, login to your Identity Observability portal and enable controls and observations. Ensure that you upload data updates on a regular basis by using the Real Time Audit and Compliance page under Settings. From there, the "Load Data" option can be used to manually initiate data loading. You can define the schedule and frequency for data loading. You can also define settings for purging of the old data to optimize performance. 


Next, use the Query Builder page to confirm that objects have been uploaded successfully. If the tables appear empty, it indicates that the data upload has not been completed.

## **Things to know**

Connectors continuously collect identity data from backend sources (LDAP, Entra ID, Active Directory) and send it to the Identity Data Management (IDM) service.

The **Pipeline Orchestrator** maps and transforms this data into the **Identity Observability (IDO)** model, reconciling user and HR identities, identifying service accounts, and normalizing attributes
such as AD UAC values.

Processed data is stored in the Identity Observability graph and visualized in the **IDO
Portal**. When **observations and controls** are active, the Observation Supervisor monitors related events, updates observed objects, and triggers alerts as needed.

Real-time data can be periodically loaded into the **Audit & Compliance database** for historical analysis.

Attribute updates or remediations made in the IDO interface are routed back to source systems through the **write-back service or** applied directly to the real-time database if not mapped.

The **Remediation Notification Service** supports automated remediation through third-party systems such as ITSM tools, orchestrators, or email.

### **When Applying a Modified Configuration**

The orchestrator will:

1.  Validate the configuration against the graph schema.

2.  Verify that all referenced data sources exist.

3.  Confirm attributes and object names match the data source schema.

4.  Create and add **flat views** in the identity catalog naming context for each configured object.

5.  Generate a **default connector configuration** for Identity Observability.

6.  Automatically trigger an **initial upload** if the configuration requires no manual input.

    a.  If manual input is needed, the pipeline state is marked as CONFIGURATION_INCOMPLETE.

7.  After uploading, automatically restart the pipeline connector.

8.  Create and deploy **graph pipelines** that push normalized data into the graph database.

9.  Continuously monitor the state of pipelines and connectors through **Prometheus metrics**.

### **When Applying a Modified Configuration**

The orchestrator will:

1.  Validate that changes are **non-structural** (e.g., new objects, edges, or properties).

    a.  **Structural changes** (like renaming objects or changing data sources) are **not supported**.

2.  Ensure the configuration aligns with the graph and data source schemas.

3.  Deploy or undeploy identity catalog views as needed.

4.  Identify and re-upload **dependent components** when new vertices or edges rely on existing objects.

5.  Re-upload dependencies and pipelines.

6.  Backup, redeploy, and synchronize graph pipelines with the new configuration.

### **When Applying an Empty Configuration**

The orchestrator will remove all data from the graph database and undeploy:

1.  All **identity catalog views**.

2.  C**onnectors**.

3.  G**raph pipelines**.

4.  Clear all data from the **graph database**.
