---
title: Enabling AAD Token for Actionable Messages
description: Enabling AAD Token for Actionable Messages
author: vermaanimesh
ms.topic: reference
ms.service: outlook
ms.subservice: o365-connectors
ms.date: 11/11/2025
ms.author: vermaanimesh
ms.localizationpriority: high
---

# Enabling AAD Token for Actionable Messages

> [!IMPORTANT]
Actionable Messages (AM) are moving from EAT (External Access Token) to AAD token authentication. Partners using EAT tokens must update their integration to support AAD tokens for requests from the AM service.

> [!IMPORTANT]
Starting November 21st 2025 we will **stop** accepting new registrations with EAT (External Access Token)

## Enabling AA token authentication

### 1. Registering app in Azure

> [!NOTE]
If you already have an app registration in Azure, skip to the next step.

1. Sign in to the Microsoft Entra admin center.
2. If you have access to multiple tenants, use the Settings icon to switch to the desired tenant via **Directories + subscriptions**.
3. Go to Identity > Applications > App registrations and select New registration.
4. Enter a display Name for your application.
5. Specify who can use the application in the Supported account types section:
    - **Accounts in any organizational directory** (Any Microsoft Entra ID tenant - Multitenant): For partners doing a Global scope AM registration.
    - **Accounts in this organizational directory only:** For Single Tenant App (Org and test scope registration).
6. Leave Redirect URI (optional) blank.
7. Select Register to complete the registration.

### 2. Register a New AM Provider for Your AAD Scenario
- Register a new provider using Actionable Messages (office.com), or use the “Migrate to MSEntra” button on your existing registration to create a copy.
- Fill in the **MsEntra Auth** section with:
    - **MsEntra Application ID**
    - **AppIdUri** (auto-generated; must be whitelisted in your app as shown in the next section).

<img src="images/Enabling AAD Token/new-provider(1).png"/>

- Approval and onboarding of the AM registration remain unchanged.

**Tip:** Use this new registration to test the AAD token scenario end-to-end. Gradually move traffic to the new registration once validated.

## 3. Expose an API and Pre-Auth Actions App
Steps to create a URI (this is the uri for which access permission will be asked by Actions from AAD, to call your Target url) and authorize Actions app to call that URI under this app registration for testing

- Open the Expose an API option from left navigation pane of the registered app 
- Add URI under the “Application ID URI” option. Use the AppIdUri generated in the provider registration. 
Example format:
`api://auth-am-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

<img src="images/Enabling AAD Token/test-app(2).png"/>

- Add the scope for this app under “Add a scope section” (e.g., Global.Test).

**Who can consent?**

- **Admins and users:** Consent from either works.
- **Admins only (Recommended):** Only admin approval works.

- Once the Admin has authorized, consent is for the whole tenant and will not be prompted again.
- Go to “Add a client application” and authorize Action app id 48af08dc-f6d2-435f-b2a7-069abd99c086 to the scopes created above.

<img src="images/Enabling AAD Token/Picture13.png"/>

## 4. Validation of AAD token

Upon receiving the AAD OBO token in the request from Actions service, partners should perform validation.
**Guidelines**
- Refer to [Access tokens in the Microsoft identity platform](https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens)
- See [Code samples for Microsoft identity platform authentication and authorization](https://learn.microsoft.com/en-us/entra/identity-platform/sample-v2-code?tabs=framework) for validation in your preferred language/framework.

**Sample AAD OBO token:**
```
{ "alg": "RS256", "kid": "27643737-6767-4678-9714-96485a53e23b", "typ": "JWT" }.{ "aud": "https://graph.microsoft.com/", "iss": "https://login.microsoftonline.com/1234567890", "iat": 1673495600, "nbf": 1673495600, "exp": 1673499200, "aio": "AWQAm/8TAAAAbIRXVv66AlGAbTpvmfbtyMHZVpuhGjjasLVHf73tIlZI6dtwBFJQFCXUTDLxNnopKxopumbIJAMd3LqIQ==", "azp": "1234567890-abcdefghijklmnopqrstuv", "amr": [ "pwd" ], "family_name": "Doe", "given_name": "John", "groups": [ "Admins", "Users" ], "preferred_username": "john.doe@contoso.com", "sub": "AUCeKGQXBnSqpWfTYEk0li8TyNul1QSuSxcPplBAwaQ", "tid": "1234567890", "uti": "yvEyycOza9zpyjmgkdDqA", "ver": "2.0" }.[Signature]
```
## 5.Getting Approval from Tenant Admins 
For a Global scope AM registration to work in any tenant, the Tenant Admin must consent to the App hosting the Target URL.
Admins can grant consent using the [oam/admin](https://outlook.office.com/connectors/oam/admin) page.

**Admin Consent**

1. Go to the oam/admin page and click the **Consent 3P Apps** button (top right).

<img src="images/Enabling AAD Token/consent-3P-apps(4).png"/>

2. The Admin Consent Dashboard will open, listing all 3P providers. Unconsented apps show an “Approve” button.

<img src="images/Enabling AAD Token/approve(5).png"/>

3. Click on a provider row to review details.
4. Click **Approve** to trigger the MSAL consent flow. Sign in and review the requested permissions.
5. Ensure **Consent on behalf of your organization** is selected for tenant-wide consent.



6. Upon clicking **Accept** button the consent will be granted, and the Microsoft Entra app will be authorized in your tenant. You will be redirected back to the oam/admin page where the app will appear as **Approved**

<img src="images/Enabling AAD Token/permission(6).png"/>

7. If status remains **Approving**, use the Refresh button to update.

8. Use the search bar to find a provider by **Name, Provider ID,** or **Microsoft Entra ID**

<img src="images/Enabling AAD Token/search-bar(8).png"/>

9. To remove consent, open Azure Portal > Enterprise Applications, search for the app’s service principal, and delete it in Properties.

<img src="images/Enabling AAD Token/Azure-portal(9).png"/>