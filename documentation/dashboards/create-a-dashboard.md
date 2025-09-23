# Create a new Dashboard 

The Dashboard Management Page provides a centralized interface for creating, configuring, and managing dashboards. Users with Technical and functional administrator privileges have full access to all dashboard operations, including dashboard creation, editing, deletion, export, and import.  
 
This document describes the steps to create a new dashboard and provides instructions for managing existing dashboards, including configuration, duplication, deletion, and import and export operations available on the Dashboard Management Page. 

## Steps for Dashboard creation  

Dashboards can only be created from the Dashboard Management Page. To create a new dashboard: 

1. Navigate to the Dashboard Management Page by clicking Admin > Manage Dashboards. 

2. Click on the Create button on the right navigation menu. 

3. Provide the required information: 

* Title: The name of the dashboard. 

* Category: The menu group in which the dashboard will appear. 

* Priority: The position of the dashboard within its category (higher values place it lower in the list). 

* Description (optional): A summary of the dashboard’s purpose. 

Once created, use the View button to open and begin editing the dashboard content. 

 
## Dashboard List 

The main interface presents a table of all dashboards available in the current environment. The following information is displayed for each entry: 

* Title
* Author
* Menu: Indicates whether the dashboard is included in the portal navigation.
* Category
* Priority: Display order within the category.
* Status: Sharing status (Private or Shared).
* Recipients: Sharing target, if applicable.
* Description 

Dashboards can be sorted by clicking on any column header. Use the filter box above the list to narrow results by matching titles or descriptions (e.g., entering "stats" will display dashboards containing that term). 

## Selection Behavior 

The dashboard list supports both single and multiple selection: 

Click: Select a single item. 

Shift + Click / Shift + Arrow Keys: Extend the selection range. 

Ctrl + Click: Toggle selection status of individual items. 

 

## Available Actions 

All dashboard actions are available to administrators, based on the current selection: 

* Create: Launch the creation of a new dashboard. 

* Instantiate: Generate a new dashboard from a selected template. Enabled only if one template is selected. 

* Configure: Modify dashboard properties such as title, category, and sharing options. Does not allow content editing. Enabled only when exactly one dashboard is selected. 

* Assign: Reassign the selected dashboard(s) to another user. 

* View: Open the dashboard in view/edit mode. Enabled only when exactly one dashboard is selected. 

* Duplicate: Create a copy of the selected dashboard. Enabled only when exactly one dashboard is selected. 

* Delete: Permanently remove one or more selected dashboards. 

 

## Export and Import Dashboards 

Administrators can export dashboards to a local file or import dashboards into the current environment. 

### Export 

Use the Export menu to either export all dashboards or export only selected dashboards. 

### Import 

Click the Import button to load dashboards from a previously exported file. 

Import behavior: 

Imported dashboards are added to the environment; no dashboards are deleted. 

If an imported dashboard already exists (identified by internal ID), it will be replaced. 

If the original creator cannot be matched, ownership defaults to the importing administrator. 

 
