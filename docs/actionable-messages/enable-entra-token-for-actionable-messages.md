---
title: Enable Microsoft Entra ID token for Actionable Messages
description: Learn how to enable Microsoft Entra ID tokens for Actionable Messages
author: vermaanimesh
ms.topic: how-to
ms.service: outlook
ms.subservice: o365-connectors
ms.date: 06/10/2026
ms.author: vermaanimesh
ms.localizationpriority: high
---

# Enable Microsoft Entra ID token for Actionable Messages

<!-- cSpell:ignore vermaanimesh -->

[!INCLUDE [legacy-token-deprecation](../includes/actionable-messages/legacy-token-deprecation.md)]

## Admin Guide: View providers with Auth type

Admins can download the list of all approved providers in their organization along with the token type being used. The data is exported in a .csv format for easy analysis and reporting.

### How to download the provider list

1. Navigate to the [Actionable Email Developer Dashboard](https://aka.ms/ActionableMessagesPortal).
2. In the top right corner, select the **Export Approved Providers** button, to export the list of approved providers in .csv format

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/get-provider.png" alt-text="A screenshot of the Actionable Email Developer Dashboard showing the position of the Get Provider List button":::

#### Important notes

- The download button is visible only after filtering by Approved providers.
- The downloaded file contains provider details along with their token type.
- If a timeout error occurs, the error message is displayed in the UI and disappears automatically after 5 seconds.

## Register an app in Azure

> [!NOTE]
> If you already have an app registration in Azure, skip to the next step.

1. Sign in to the Microsoft Entra admin center.
2. If you have access to multiple tenants, use the Settings icon to switch to the desired tenant via **Directories + subscriptions**.
3. Go to **Identity** > **Applications** > **App registrations** and select **New registration**.
4. Enter a display name for your application.
5. Specify who can use the application in the **Supported account types** section:
    - **Accounts in any organizational directory** (Any Microsoft Entra ID tenant - Multitenant): For partners doing a Global scope AM registration.
    - **Accounts in this organizational directory only:** For Single Tenant App (Org and test scope registration).
6. Leave **Redirect URI (optional)** blank.
7. Select **Register** to complete the registration.

## Register a new AM provider

- Register a new provider using Actionable Messages (office.com), or use the **Migrate to MSEntra** button on your existing registration to create a copy.
- Fill in the **MsEntra Auth** section with:
  - **MsEntra Application ID**
  - **AppIdUri** (auto-generated; must be allowlisted in your app as shown in the next section).

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/new-provider.jpg" alt-text="A screenshot of the new provider form fields":::

- Approval and onboarding of the AM registration remain unchanged.

> [!TIP]
> Use this new registration to test the Microsoft Entra ID token scenario end-to-end. Gradually move traffic to the new registration once validated.

## Expose an API and pre-authorize the Actions app

1. Select the **Expose an API** option from the left navigation pane of the registered app.
2. Add a URI under the **Application ID URI** option. Use the **AppIdUri** generated in the provider registration. Example format:
`api://auth-am-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/expose-an-api.jpg" alt-text="A screenshot of the 'Expose an API' form in the Entra admin portal":::

3. Add the scope for this app under **Add a scope section** (e.g., Global.Test).

4. Choose a value for **Who can consent?**.

    - **Admins and users:** Consent from either works.
    - **Admins only (Recommended):** Only admin approval works.

    Once an admin has authorized, consent applies to the whole tenant and users won't be prompted again.

5. Go to **Add a client application** and authorize Action app ID `48af08dc-f6d2-435f-b2a7-069abd99c086` to the scopes created above.

## Validate the Microsoft Entra ID token

When your service receives the token in the request from the Actions service, validate it. For details on validating tokens, see [Access tokens in the Microsoft identity platform](/entra/identity-platform/access-tokens).

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

For a Global scope actionable message registration to work in any tenant, the tenant admin must consent to the app hosting the target URL. Admins can grant consent using the [Actionable Email Developer Dashboard](https://aka.ms/ActionableMessagesPortal) page.

1. Go to the Actionable Email Developer Dashboard and select the **AAD Consent** button (left side panel).

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/aad-consent.png" alt-text="A screenshot of the Actionable Email Developer Dashboard showing the 'Consent 3P Apps' button":::

2. The Admin Consent Dashboard will open, listing all third-party providers. Apps that need consent show a **Grant Consent** button.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/actionable-message-dashboard.png" alt-text="AM Email Dashboard":::

3. Select **Grant Consent** to trigger the consent flow. Sign in and review the requested permissions.
4. Ensure **Consent on behalf of your organization** is selected for tenant-wide consent.
5. Select **Accept** to grant consent. The Microsoft Entra app is now authorized in your tenant. The browser redirects back to the dashboard where the app status is **Consented**.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/permission.png" alt-text="User permission screen":::

6. If status remains **Consenting**, use the **Refresh Consent Status** button to update.

7. Use the search bar to find a provider by **Name, Provider ID,** or **Microsoft Entra ID**.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/search-bar.png" alt-text="Search bar in AM portal":::

8. To remove consent, open the Azure portal and select **Enterprise Applications**. Search for the app's service principal and delete it in **Properties**.

    :::image type="content" source="images/enabling-entra-token-for-actionable-messages/service-principal-azure-portal.png" alt-text="Azure portal screen":::
