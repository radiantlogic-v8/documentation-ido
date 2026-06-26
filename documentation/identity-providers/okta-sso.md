---
title: Configure Okta as an Identity Provider
description: How to configure Okta as an identity provider for Identity Observability using Keycloak OpenID Connect
---

## Overview

This guide explains how to configure Okta as an identity provider for Identity Observability using Keycloak's OpenID Connect integration. By the end, users will be able to log in to the Identity Observability portal using their Okta credentials.

### Prerequisites
  
- Admin access to the Okta Admin Console
- Admin access to Keycloak
- An existing Identity Observability deployment


###  1. Create an Okta App Integration

Follow the steps listed below to create an [OpenID connect app integration](https://help.okta.com/en-us/content/topics/apps/apps_app_integration_wizard_oidc.htm)): 

1. In the Okta Admin Console, go to **Applications > Applications**.
2. Click **Create App Integration**.
3. Select **OIDC - OpenID Connect** as the sign-in method.
4. Select **Web Application** as the application type.
6. Click **Next**.
7. Enter a name for your application (e.g. `IDO`).

   ![Okta general settings](Media/okta-general.png)


###  2. Configure the Redirect URI in IDP Console

Before saving the Okta app, you need to retrieve the redirect URI from your [IDP console](../installation/installation-steps/#log-in-to-identity-observability-endpoints). To retrieve this,  

1. Login to your IDP console and navigate to **Identity Providers**.
2. Click **OpenID Connect v1.0**.
3. Copy the **Redirect URI** displayed on the page.

   ![Keycloak Identity Provider page with Redirect URI field highlighted](Media/keycloak-01-redirect-uri.png)

4. Return to Okta and paste the copied URI into the **Sign-in redirect URIs** field.

   ![Okta app settings with Sign-in redirect URIs field populated](Media/okta-03-sign-in-redirect-uri.png)

5. Scroll down and click **Save**.

###  3. Create an Okta Group and Assign Users

1. In the Okta Admin Console, go to **Directory > Groups**.
2. Click **Add group**.
3. Enter a name for the group (e.g. `Identity Observability`) and click **Save**.

   ![Add group dialog with group name filled in](Media/okta-04-add-group.png)

4. Click the group you just created, then click **Assign people**.
5. Find and assign the appropriate users, then click **Done**.
   
####  Assign the Group to the Identity Observability Application

1. Go to **Applications > Applications** and click **Identity Observability**.
2. Click the **Assignments** tab.
3. Click **Assign > Assign to Groups**.
4. Find the group you created and click **Assign**, then click **Done**.

   ![Identity Observability Assignments tab showing the group assigned](Media/okta-06-assign-group-to-app.png)


###  4. Copy Client Credentials to Keycloak

1. In the Identity Observability application in Okta, click the **General** tab.
2. Copy the **Client ID**.

   ![Okta Identity Observability app General tab with Client ID highlighted](Media/okta-07-client-id.png)

3. In Keycloak, paste the Client ID into the **Client ID** field of the OpenID Connect identity provider.
4. Back in Okta, copy the **Client Secret**.
5. In Keycloak, paste the Client Secret into the **Client Secret** field.

   ![Keycloak identity provider form with Client ID and Client Secret filled in](Media/keycloak-02-client-credentials.png)

#### Copy the Discovery Endpoint

1. In Okta, go to **Security > API** and click the **default** authorization server.
2. Copy the **Metadata URI**.

   ![Okta default authorization server settings with Metadata URI highlighted](Media/okta-08-metadata-uri.png)

3. In Keycloak, paste the Metadata URI into the **Discovery endpoint** field.
4. Click **Show metadata** and review the retrieved metadata to confirm the connection is valid.

   ![Keycloak identity provider with Discovery endpoint populated and metadata displayed](Media/keycloak-03-discovery-endpoint.png)

###  5. Configure the Keycloak Identity Provider Settings

1. In Keycloak, click **Advanced** on the identity provider configuration page.
2. Scroll down and enable **Trust Email**.

   ![Keycloak Advanced settings with Trust Email toggle enabled](Media/keycloak-04-trust-email.png)

3. Click **Scopes**.
4. Enter the following scopes: `openid email profile groups`.
5. Click **Save**.

   ![Keycloak Scopes field with openid email profile groups entered](Media/keycloak-05-scopes.png)


###  6. Configure Scopes in Okta

1. In Okta, navigate to your **default** authorization server and click **Scopes**.
2. Click **Add Scope**.
3. Fill in the following fields:
   - **Name**: `groups`
   - **Display phrase**: a short label visible to users during consent
   - **Description**: a description of what this scope provides
4. Click **Create**.

   ![Okta Add Scope dialog with groups scope configured](Media/okta-09-add-scope.png)


###  7. Configure a Groups Claim in Okta

1. Click **Claims**, then click **Add Claim**.
2. Fill in the following fields:
   - **Name**: `groups`
   - **Include in token type**: Select **ID Token** and set to **Always**
   - **Value type**: Select **Groups**
   - **Filter**: Set to **Starts with** (or **Equals**) and enter `Identity Observability`
   - **Include in**: Select **The following scopes** and enter `groups`
3. Click **Create**.

   ![Okta Add Claim dialog with groups claim configured](Media/okta-10-add-claim.png)


###  8. Configure an Access Policy in Okta

1. Click **Access Policies**, then click **Add Policy**.
2. Fill in the following fields:
   - **Name**: a descriptive name for the policy
   - **Description**: a brief description
   - **Assign to**: select **The following clients**, search for and select `Identity Observability`
3. Click **Create Policy**.

   ![Okta Add Policy dialog with Identity Observability client selected](Media/okta-11-add-policy.png)

#### Add a Policy Rule

1. Click **Add rule**.
2. Enter a name for the rule.
3. Uncheck **Client Credentials** and **Device Authorization** grant types.
4. Under **The following scopes**, add the **OIDC default scopes** and the `groups` scope.
5. Click **Create rule**.

   ![Okta Add Rule dialog with grant types and scopes configured](Media/okta-12-add-rule.png)


###  9. Verify the Configuration with Token Preview

1. Click **Token Preview**.
2. Select `Identity Observability` as the application.
3. Set the grant type to **Authorization Code**.
4. Select a test user.
5. Add the following scopes: `openid`, `email`, `groups`.
6. Click **Preview Token**.
7. Scroll down in the token preview and confirm that the `groups` claim contains `Identity Observability` as a value.

   ![Token Preview result showing groups claim with Identity Observability value](Media/okta-13-token-preview.png)


###  10. Configure a Mapper in Keycloak

1. In Keycloak, open the OpenID Connect identity provider and click **Mappers**.
2. Click **Add mapper** and fill in the following fields:
   - **Name**: a descriptive name for the mapper
   - **Sync mode override**: **Force**
   - **Mapper type**: **Claim to Role**
   - **Claim**: `groups`
   - **Claim Value**: `Identity Observability`

   ![Keycloak Add Mapper dialog with Claim to Role settings configured](Media/keycloak-06-add-mapper.png)

3. Click **Select Role**, then select **Client roles**.
4. Search for `technical` and select the **technicaladmin** role.
5. Click **Assign**, then click **Save**.

   ![Keycloak role selection with technicaladmin role highlighted](Media/keycloak-07-select-role.png)


###  11. Assign Roles to Users

1. In Keycloak, go to **Users**.
2. Create a new user, or locate an existing user and update their role mapping as needed.
3. To remove the `technicaladmin` role from an existing user, open the user's **Role mapping** tab, click the kebab menu (⋮) next to the role, select **Unassign**, and confirm by clicking **Remove**.

   ![Keycloak User Role mapping tab with Unassign option in kebab menu](Media/keycloak-08-role-mapping.png)

###  12. Log In to Identity Observability via Okta Credentials

1. Navigate to the Identity Observability portal.
2. Click **oidc** on the login page.

   ![Identity Observability portal login page with oidc option highlighted](Media/Identity Observability-01-login-oidc.png)

3. If your account already exists, click **Add to existing account** and verify your email address.

   ![Identity Observability account linking prompt with Add to existing account option](Media/Identity Observability-02-add-to-existing-account.png)

4. Once verified, you will be logged in to Identity Observability using your Okta credentials.
