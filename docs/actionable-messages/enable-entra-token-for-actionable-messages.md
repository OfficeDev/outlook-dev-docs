---
title: Enable Microsoft Entra ID token for Actionable Messages
description: Learn how to enable Microsoft Entra ID tokens for Actionable Messages
author: vermaanimesh
ms.topic: how-to
ms.service: outlook
ms.subservice: o365-connectors
ms.date: 11/17/2025
ms.author: vermaanimesh
ms.localizationpriority: high
---

# Enable Microsoft Entra ID token for Actionable Messages

<!-- cSpell:ignore vermaanimesh -->

> [!IMPORTANT]
> Actionable Messages (AM) are moving from EAT (External Access Token) to Microsoft Entra ID token authentication. Partners using EAT tokens must update their integration to support AAD tokens for requests from the AM service. Starting November 21st 2025 we will **stop** accepting new registrations with EAT (External Access Token).

## Register an app in Azure

> [!NOTE]
> If you already have an app registration in Azure, skip to the next step.

1. Sign in to the Microsoft Entra admin center.
1. If you have access to multiple tenants, use the Settings icon to switch to the desired tenant via **Directories + subscriptions**.
1. Go to **Identity** > **Applications** > **App registrations** and select **New registration**.
1. Enter a display name for your application.
1. Specify who can use the application in the **Supported account types** section:
    - **Accounts in any organizational directory** (Any Microsoft Entra ID tenant - Multitenant): For partners doing a Global scope AM registration.
    - **Accounts in this organizational directory only:** For Single Tenant App (Org and test scope registration).
1. Leave **Redirect URI (optional)** blank.
1. Select **Register** to complete the registration.

## Register a new AM provider

- Register a new provider using Actionable Messages (office.com), or use the **Migrate to MSEntra** button on your existing registration to create a copy.
- Fill in the **MsEntra Auth** section with:
  - **MsEntra Application ID**
  - **AppIdUri** (auto-generated; must be allowlisted in your app as shown in the next section).

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/new-provider.jpg" alt-text="A screenshot of the new provider form fields":::

- Approval and onboarding of the AM registration remain unchanged.

> [!TIP]
> Use this new registration to test the AAD token scenario end-to-end. Gradually move traffic to the new registration once validated.

## Expose an API and pre-authorize the Actions app

1. Select the **Expose an API** option from left navigation pane of the registered app
1. Add URI under the **Application ID URI** option. Use the **AppIdUri** generated in the provider registration. Example format:
`api://auth-am-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/expose-an-api.jpg" alt-text="A screenshot of the 'Expose an API' form in the Entra admin portal":::

1. Add the scope for this app under **Add a scope section** (e.g., Global.Test).

1. Choose a value for **Who can consent?**.

    - **Admins and users:** Consent from either works.
    - **Admins only (Recommended):** Only admin approval works.

    Once the Admin has authorized, consent is for the whole tenant and won't be prompted again.

1. Go to **Add a client application** and authorize Action app ID `48af08dc-f6d2-435f-b2a7-069abd99c086` to the scopes created above.

## Validate the AAD token

Upon receiving the token in the request from Actions service, partners should perform validation. For details on validating tokens, see [Access tokens in the Microsoft identity platform](/entra/identity-platform/access-tokens).

There are also [code samples for Microsoft identity platform authentication and authorization](/entra/identity-platform/sample-v2-code?tabs=framework) for validation in your preferred language/framework.

### Sample token

```json
{
  "alg": "RS256",
  "kid": "27643737-6767-4678-9714-96485a53e23b",
  "typ": "JWT"
}.{
  "aud": "https://graph.microsoft.com/",
  "iss": "https://login.microsoftonline.com/1234567890",
  "iat": 1673495600,
  "nbf": 1673495600,
  "exp": 1673499200,
  "aio": "AWQAm/8TAAAAbIRXVv66AlGAbTpvmfbtyMHZVpuhGjjasLVHf73tIlZI6dtwBFJQFCXUTDLxNnopKxopumbIJAMd3LqIQ==",
  "azp": "1234567890-abcdefghijklmnopqrstuv",
  "amr": [
    "pwd"
  ],
  "family_name": "Doe",
  "given_name": "John",
  "groups": [
    "Admins",
    "Users"
  ],
  "preferred_username": "john.doe@contoso.com",
  "sub": "AUCeKGQXBnSqpWfTYEk0li8TyNul1QSuSxcPplBAwaQ",
  "tid": "1234567890",
  "uti": "yvEyycOza9zpyjmgkdDqA",
  "ver": "2.0"
}.[Signature]
```

## Get approval from admins

For a Global scope actionable message registration to work in any tenant, the tenant admin must consent to the app hosting the target URL. Admins can grant consent using the [Actionable Email Developer Dashboard](https://outlook.office.com/connectors/oam/admin) page.

1. Go to the Actionable Email Developer Dashboard and select the **Consent 3P Apps** button (top right).

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/consent-third-party-apps.png" alt-text="A screenshot of the Actionable Email Developer Dashboard showing the 'Consent 3P Apps' button":::

1. The Admin Consent Dashboard will open, listing all 3P providers. Apps that need consent show an **Approve** button.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/actionable-message-dashboard.png" alt-text="AM Email Dashboard":::

1. Select a provider row to review details.
1. Select **Approve** to trigger the consent flow. Sign in and review the requested permissions.
1. Ensure **Consent on behalf of your organization** is selected for tenant-wide consent.
1. Select **Accept** to grant consent. The Microsoft Entra app is now authorized in your tenant. The browser redirects back to the dashboard where the app status is **Approved**.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/permission.png" alt-text="User permission screen":::

1. If status remains **Approving**, use the **Refresh** button to update.

1. Use the search bar to find a provider by **Name, Provider ID,** or **Microsoft Entra ID**.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/search-bar.png" alt-text="Search bar in AM portal":::

1. To remove consent, open Azure Portal and select **Enterprise Applications**. Search for the app's service principal, and delete it in **Properties**.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/service-principal-azure-portal.png" alt-text="Azure portal screen":::
