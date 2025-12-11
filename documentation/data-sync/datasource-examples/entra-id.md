---
title: Entra ID Data Source
description: Guide that shows how to create Entra ID Data Source 
---

## Overview 

This document provides steps to configure Entra ID as a data source for Identity Observability. 


### Creating Data Source 

1. Log in to the **Identity Observability Data Sync Config** portal. From the left navigation pane, select **Data Catalog > Data Sources**. Click **NEW SOURCE**.  

    ![Image of New Source Button](../media/new-source.png "Image of New Source Button")


2. In the **Data Source Types** gallery, select the **Microsoft Entra ID** tile. Click **SELECT**.

     ![Image of Data Source Type](../media/entra-source.png "Image of Data Source Type")


3. Provide a unique name for the data source. Optionally, enter a description. Confirm that the **ACTIVE** toggle is enabled. Certain characters including dashes (-) are not permitted in the data source name. If invalid characters are used, the system will display an error message.  
Leave **Secure Data Connector** set to **None**, as it is not required for this configuration.  
    
![Image of Data Source Details](../media/entra-details.png "Image of Data Source Details")


4. Scroll to the **Connection Info** section. Enter the required connection details for the target Entra ID tenant:  

- The **URL** and **SCOPE** fields contain default values that typically do not require modification but adjust as necessary based on organizational requirements.  
- Update the **OAUTHURL** field using the domain of the target Entra ID tenant.  
- Enter the appropriate **USERNAME** and **PASSWORD** for an account with sufficient permission to manage identity objects through the Microsoft Graph API.  

     ![Image of Connection Info](../media/entra-connection.png "Image of Data Source Connection Details")


5. Click **TEST CONNECTION** to verify credentials and connectivity. A pop-up will display the results. If the test fails, additional diagnostic information will be provided. When configuration is complete and the connection test is successful, click **CREATE**.  

After creation, the system returns to the main **Data Sources** page, where the new data source will appear with status **ACTIVE**.  



## Configure the Namespace for Entra ID 

Unlike direct Active Directory integration, which requires only a data source, Entra ID integration requires a **Namespace**. This Namespace will later be referenced by a “proxy” data source as part of the IDO configuration. To create a namespace, follow these steps: 

1. Navigate to **Directory Namespace > Namespace Design**. 

2. Click **NEW NAMING CONTEXT**.

3. Select a label from the drop-down menu (e.g., `cn`, `o`, or `dc`). For this example, select **o** as the root for the Namespace. Enter the label value and click **CONFIRM**. 

    ![Image of naming context label](../media/label.png "Image of naming context label")


4. Select the newly created root level, then click **MOUNT BACKEND**. 

    ![Image showing mount backend](../media/mount-backend.png "Image showing mount backend")


5. Choose the **Virtual Tree** tile and click **SELECT**, then click **MOUNT**. This mounts the Entra ID View. RadiantOne automatically generates views during Namespace creation. Use the drop-down to review available virtual directory views.  


## Add Namespace Levels for Users 

1. Click **NEW LEVEL** and select **Label**.  

    ![Image of new level button](../media/level-users.png "Image of new level button")


2. Create an **ou** label with the value `users`. This structure is customizable to organizational requirements. Click **CONFIRM**.  

    ![Image of user label](../media/users-ou.png "Image of user label")


3. The next step is to create a level that makes the Entra ID data accessible through the Namespace view. This is created under the `users` level we created in the previous step.  
Select the new **users** level and click **NEW LEVEL** and select **CONTENT** as the level type. This level will contain the Entra ID identity data to be synchronized.  

   ![Image of user content](../media/user-content.png "Image of user content")


4. Choose the **Entra ID data source** (example: ido_entra_backend) from the drop-down, then select the **Entra ID schema** (example: default-ido_entra_backend) from the dropdown and click **NEXT**.

   ![Image of data source and schema](../media/datasource.png "Image of data source and schema")


5. You can see Entra ID Schema objects we extracted, which included all the available objects. For this example, select the Entra ID **user** Schema object. A pop-up will indicate the success of the operation or provide troubleshooting information if issues occur.  

   ![Image of user object being mounted](../media/userobj.png "Image of user object being mounted")


6. Select the **users content** level. Open the **OBJECT BUILDER** tab. Enable attribute selection by selecting the highlighted icon and choose desired attributes. All attributes may be selected, but this is optional. Once you select the attributes, click **DONE**.  

   ![Image of object builder screen](../media/objbuilder.png "Image of object builder screen")


7. Review selected attributes in the right pane. Remove any undesired attributes by reopening the selector. Click **SAVE** to finalize.  


## Add Namespace Levels for Groups 

1. Return to the root of the Namespace and click **NEW LEVEL**.  

   ![Image of new level](../media/group-level.png "Image of new level")


2. Select **Label**.  

3. Create an **ou** label with the value `groups` and click **CONFIRM**.  

   ![Image of group naming context](../media/groupcontext.png "Image of group naming context")


4. Select the new **groups** level and click **NEW LEVEL**.  

5. Select **CONTENT** as the level type. Since the data source and schema were previously configured, they will already be selected. Click **NEXT**.  

   ![Image of new level](../media/groupscontent.png "Image of new level")


6. Select the **group schema** object, then click **SELECT**.  

   ![Image of group being mounted](../media/groupmount.png "Image of group being mounted")


7. After confirmation, select the **group content** level and open the **OBJECT BUILDER** tab.  

   ![Image of group object builder](../media/groupobject-builder.png "Image of group object builder")

8. Select the desired attributes as needed and click **DONE**. Review the attribute list, adjust if required, and click **SAVE**.  

   ![Image of mapped attributes](../media/mapped-attr.png "Image of mapped attributes")



## Configure and Initialize the Namespace Cache 

1. From the root level of the Namespace, select **CACHE** and click **CREATE NEW CACHE**.

   ![Image of Cache](../media/cache.png "Image of Cache")

2. Select the top-level Namespace (e.g., `o=entraid`) for cache creation and click **CREATE**.  

2. Choose **Real-time change detection**. RadiantOne’s Entra ID connector supports real-time monitoring, which is required for Identity Observability synchronization. Click **NEXT**. 

   ![Image of Realtime Cache option](../media/realtime-cache.png "Image of Realtime Cache option")

3. Click **INITIALIZE**. Initial cache creation may take several minutes to several hours, depending on the volume of data. This will generate an LDIF file and load it into the HDAP store.  

4. Select the option to create a new LDIF and click **DONE**. A pop-up will display operation status. Use provided diagnostics and system logs to troubleshoot as needed. 

   ![Image of new LDIF option](../media/new-ldif.png "Image of new LDIF option")


5. Click **SAVE** to complete the cache configuration. After successful creation, you will see the details of the namespace cache.  


## Create a Proxy Data Source for Entra ID 

To reference the Entra ID Data Source, we need to create a **“proxy”**. This is an additional Data Source that will connect to the Entra ID backend, through the previously created Data Source. This is required due to a current limitation on support for the `mGraph` API directly from IDO. This may change in a future release, and this document will be updated as needed.  

To create this proxy, follow these steps (similar to the original data source creation steps):  

1. From the left navigation pane, select **Data Catalog > Data Sources**.  

2. Click **NEW SOURCE**.  

3. In the **Data Source Types** gallery, select **RadiantOne**, then click **SELECT**. 

   ![Image of RadiantOne Data Source](../media/radiantone.png "Image of Realtime Cache option")


4. Enter a unique name for the data source. Optionally enter a description. Ensure the **ACTIVE** toggle is enabled.  

5. In the **Connection Info** section, enter the required details for the target RadiantOne environment. Use the **localhost URL** for this configuration, as this data source will be referenced in the Identity Observability Pipeline.

   ![Image of Connection Details](../media/connectioninfo-2.png "Image of Connection Details")


6. Click **TEST CONNECTION** to validate credentials and connectivity. Troubleshooting details will be provided if the test fails.  

7. Click the folder icon to view available Namespaces. Select the root of the previously created **Entra ID Namespace**, then click **SELECT**.  

   ![Image of Base DN](../media/base-dn-folder.png "Image of Base DN folder")


9. Click **CREATE** to finalize the configuration. The new data source will appear with a status of **ACTIVE**.  

The next step is to create a configuration pipeline. Refer to [this example](../pipeline-configuration/#configuration-example) for details. 

