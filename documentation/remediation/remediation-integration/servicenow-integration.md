
## Overview 

This document outlines how you can create a ServiceNow integration that creates new tickets after a remediation action is performed.  

### Requirements 

- You must have a ServiceNow instance with permissions to create Scripted REST APIs and update the target table (e.g., incident).  

- Your remediation policy must also be [configured](../remediation-policies/#configuring-remediation-target-notifications) to use ServiceNow as the target type and assigned to the appropriate repository.  

### Setup Steps 

The following sections offer a high-level overview of the integration configuration process. For comprehensive instructions on ServiceNow integration, refer to the official [ServiceNow documentation site](https://www.servicenow.com/docs/bundle/zurich-api-reference/page/integrate/guides/concept/developer-guides.html).  

#### 1. Create a Scripted REST API 

Open ServiceNow Studio (create an application if needed).  

Click Create New File → select your application scope.  

Choose Scripted REST API and create it.  

#### 2. Configure ACLs 

Open your Scripted REST API and click the lock icon.  

Add ACLs to restrict access (e.g., Scripted REST External Default for admin-only).  

Add additional ACLs for specific resources if needed.  

#### 3. Create a Resource (Endpoint) 

In the Scripted REST API, go to the Resources tab.  

Click New to create a resource for ticket creation.  

Add ACLs or authentication requirements for the resource.  

#### 4. Add Script Logic 

Create and run a script with appropriate logic. The example below creates incidents after receiving notification from Identity Observability in ServiceNow:  


```
    (function process(request, response) { 
        try { 
            var body = request.body.data; 

            if (!body.short_description) { 
                response.setStatus(400); 
                response.setBody({ error: "'short_description' field is mandatory." }); 
                return; 
            } 

            if (!body.description) { 
                response.setStatus(400); 
                response.setBody({ error: "'description' field is mandatory." }); 
                return; 
            } 

            var gr = new GlideRecord('change_request'); 
            gr.initialize(); 
            gr.short_description = body.short_description; 
            gr.description = body.description || ""; 

            /* customize script with other fields in the body */ 

            // Insert the record into the change_request table 
            var rfcSysId = gr.insert(); 
            if (!rfcSysId) { 
                response.setStatus(500); 
                response.setBody({ error: "Error creating RFC." }); 
                return; 
            } 

            // Response with the incident number and sys_id 
            response.setBody({ 
                result: { 
                    number: gr.number, 
                    sys_id: rfcSysId, 
                    message: 'RFC created successfully.' 
                } 
            }); 

        } catch (ex) { 
            response.setStatus(500); 
            response.setBody({ error: ex.message }); 
        } 
    })(request, response); 

```

Note that ServiceNow automatically calculates the ticket’s priority based on the Impact and Urgency fields. Do not set Priority directly; refer to the linked document to understand the resulting priority.
To use this script with other tables, update the GlideRecord table name (for example, use 'problem' for Problem records). Verify which fields are required for the table you’re targeting, and ensure the script properly handles the corresponding payload structure.

#### 6. Define the API Endpoint

Define your API endpoint and ensure that it is the same endpoint that receives the remediation notification. Example format: "https://<instance>.service-now.com/api/<scope_id>/<api_id>/<resource_path>"

You can get the `scope_id` and `api_id` from the Scripted REST API file in ServiceNow studio.

#### 7. Perform remediation in Identity Observability

To test your integration, first ensure that your ServiceNow remediation target is configured and assigned to the right repository. Next, perform remediation in Identity Observability, such as removing an Identity account. This should trigger the remediation notification and send a JSON body matching your script expectations. You can also send a test request using Postman or any other tool to verify that the API endpoint works as expected.

**Example Body:**

{
    "description" : "Please, disable the account ACMEB.DKNIGHT20",
    "short_description" : "Disable account ACMEB.DKNIGHT20",
    "action" : "account/disable"
}

Finally, confirm that the incident is created correctly in your ServiceNow account.

