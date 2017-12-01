---
title: Authenticate a user with an single-sign-on token in an Outlook add-in | Microsoft Docs
description: Learn about using the single-sign-on token provided by an Outlook add-in to implement SSO with your service.
author: jasonjoh

ms.topic: article
ms.technology: office-add-ins
ms.date: 09/13/2017
ms.author: jasonjoh
---

# Authenticate a user with an single-sign-on token in an Outlook add-in

Single sign-on access tokens provide a seamless way for your add-in to authenticate and obtain access tokens to call the [Microsoft Graph API](https://developer.microsoft.com/en-us/graph/docs/concepts/overview). Consider using SSO access tokens if your add-in:

- Is used primarily by Office 365 users
- Needs access to:
    - Microsoft services that are exposed as part of the Microsoft Graph
    - A non-Microsoft service that you control

The SSO authentication method uses the [OAuth2 On-Behalf-Of flow provided by Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-on-behalf-of). It requires that the add-in register in the [Application Registration Portal](https://apps.dev.microsoft.com/) and specify any required Microsoft Graph scopes in its manifest.

Using this method, your add-in can obtain an access token scoped to your server back-end API. The add-in uses this as a bearer token in the `Authorization` header to authenticate a call back to your API. At that point your server can:

- Complete the On-Behalf-Of flow to obtain an access token scoped to the Microsoft Graph API
- Use the identity information in the token to establish the user's identity and authenticate to your own back-end services

 >**Note** If your application needs to use Exchange Web Services for operations beyond what is supported with the [mailbox.makeEwsRequestAsync method] (https://docs.microsoft.com/en-us/outlook/add-ins/web-services) (such as using the EWS Managed API), you cannot configure the required Exchange Online permissions in the Application Registration Portal. You will need to register your application in your [Azure Active Directory tenant] (https://docs.microsoft.com/en-us/azure/active-directory/active-directory-app-registration) in order to configure the necessary permissions.

## Registering your add-in

In order to participate in the Azure OAuth flows, the add-in must be registered in the Application Registration Portal. Minimally the add-in must register a Web API in order for the add-in to obtain an SSO token. If the add-in requires access to the Microsoft Graph API, then the add-in must also register a web app.

The registration process starts by creating an application in the portal. This generates an application ID. After the app is created, then components are added as platforms.

### Registering a Web API

The Web API component of the add-in is the endpoint that will receive the SSO token.

1. In the details of the application in the portal, select the **Add Platform** button.
1. Select **Web API**.
1. Enter the following details:
    - **Application ID URI**: Use the format `api://{add-in server host}/{application id}`
    - **Scopes defined by this API**: Leave default value
    - **Pre-authorized applications**: Add the application ID for Office: `d3590ed6-52b3-4102-aeff-aad2292ab01c`, with the scope `api://{add-in server host}/{application id}/access_as_user`

For example, for an addin that is hosted at `https://addin.contoso.com` with an application ID of `5661fed9-f33d-4e95-b6cf-624a34a2f51d`, the **Web API** section of the application registration looks like the following.

![A screenshot of the Web API section of an application registration for an add-in that uses single-sign-on](images/single-sign-on-web-api-details.PNG)

### Registering a Web app

The Web app component of the add-in allows your server back-end to complete the On-Behalf-Of flow to exchange the initial token for an access token that allows access to the Microsoft Graph API.

1. In the details of the application in the portal, select the **Add Platform** button.
1. Select **Web**.
1. Enter the following details:
    - **Redirect URLs**: Enter the base URL of the add-in.

For example, for the add-in described above, hosted at `https://addin.contoso.com`, the **Web** section of the application registration looks like the following.

![A screenshot of the Web section of an application registration for an add-in that uses single-sign-on](images/single-sign-on-web-details.PNG)

### Generate an application password

In order to request a Microsoft Graph token, the Web app registration needs a shared secret to include in the `client_secret` parameter of the token request.

1. Choose the **Generate New Password** button under **Application Secrets**.
1. Copy the generated password when it is displayed and save it in a safe place.

### Configure necessary Microsoft Graph permissions

The final step in the registration process is to configure the required permissions to the Microsoft Graph. These permissions should be added as delegated permissions, **NOT** application permissions.

Keep in mind the following guidelines:

- The Office JS API requires that you add both `User.Read` and `profile` to the delegated permissions.
- If you are using an OAuth library to handle the token requests from your back-end, it may request additional scopes (in addition to the ones you specifically request) that you should also list here. For example, the Microsoft Authentication Library always adds `openid` and `offline_access` to token requests.

For example, if your add-in requires read access to the user's OneDrive files, the **Microsoft Graph Permissions** section looks like the following.

![A screenshot of the Microsoft Graph Permissions section of an application registration for an add-in that uses single-sign-on](images/single-sign-on-graph-permissions.PNG)

### Providing consent when sideloading an add-in

When an add-in that uses SSO is acquired from the Office Store, the store UI handles prompting the user for consent to the requested Graph permissions. However, when sideloading the add-in for testing purposes, the store consent UI is bypassed. In order for the add-in to obtain tokens, you need to provide consent.

You have two choices for providing consent. You can use an administrator account and consent once for all users in your Office 365 organization, or you can use any account to consent for just that user.

#### Provide admin consent for all users

If you have access to a tenant administrator account, this method will allow you to provide consent for all users in your organization, which can be convenient if you have multiple developers that need to develop and test your add-in.

1. Browse to `https://login.microsoftonline.com/common/adminconsent?client_id={application_ID}&state=12345`, where `{application_ID}` is the application ID shown in your app registration.
1. Sign in with your administrator account.
1. Review the permissions and click **Accept**.

The browser will attempt to redirect back to your app, which may not be running. You might see a "this site cannot be reached" error after clicking **Accept**. This is OK, the consent was still recorded.

#### Provide consent for a single user

If you don't have access to a tenant administrator account, or you just want to limit consent to a few users, this method will allow you to provide consent for a single user.

1. Browse to `https://login.microsoftonline.com/common/oauth2/authorize?client_id={application_ID}&state=12345&response_type=code`, where `{application_ID}` is the application ID shown in your app registration.
1. Sign in with your account.
1. Review the permissions and click **Accept**.

The browser will attempt to redirect back to your app, which may not be running. You might see a "this site cannot be reached" error after clicking **Accept**. This is OK, the consent was still recorded.

## Updating the add-in manifest

The next step to enable SSO in the add-in is to add a `WebApplicationInfo` element into the `VersionOverridesV1_1` [VersionOverrides](https://dev.office.com/reference/add-ins/manifest/versionoverrides?product=outlook) element. This element contains the following child elements:

- `Id` - set to the application ID obtained during app registration
- `Resource` - set to the Application ID URI configured in the **Web API** section of the app registration
- `Scopes` - contains a `Scope` element for each Microsoft Graph scope required by the add-in

For example, for the add-in mentioned in the prior examples, the `WebApplicationInfo` element looks like the following.

```xml
<VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides/1.1" xsi:type="VersionOverridesV1_1">

  ...

  <WebApplicationInfo>
    <Id>5661fed9-f33d-4e95-b6cf-624a34a2f51d</Id>
    <Resource>api://addin.contoso.com/5661fed9-f33d-4e95-b6cf-624a34a2f51d</Resource>
    <Scopes>
      <Scope>user.read</Scope>
      <Scope>files.read</Scope>
    </Scopes>
  </WebApplicationInfo>
</VersionOverrides>
```

## Getting the SSO token

The add-in obtains an SSO token by calling `Office.context.auth.getAccessTokenAsync`.

```js
Office.context.auth.getAccessTokenAsync(function (result) {
    if (result.status === "succeeded") {
        // No need to prompt user, use this token to call Web API
        var ssoToken = result.value;
        ...
    } else {
        if (result.error.code === 13003) {
            // SSO is not supported for this user, fallback
            // on another authentication method
        } else {
            // Handle error
        }
    }
});
```

The add-in includes the token in the `Authorization` header when sending a request back to the Web API.

```js
$.ajax({
    type: "POST",
    url: "/api/DoSomething",
    headers: {
        "Authorization": "Bearer " + ssoToken
    },
    data: { /* some JSON payload */ },
    contentType: "application/json; charset=utf-8"
}).done(function (data) {
    // Handle success
}).fail(function (error) {
    // Handle error
}).always(function () {
    // Cleanup
});
```

## Using the SSO token at the back-end

Once the Web API receives the access token, it must validate it before using it. The token is a JSON Web Token (JWT), which means that validation works just like token validation in most standard OAuth flows. There are a number of libraries available that can handle JWT validation, but the basics include:

- Checking that the token is well-formed
- Checking that the token was issued by the intended authority
- Checking that the token is targeted to the Web API

Keep in mind the following guidelines when validating the token:

- Valid SSO tokens will be issued by the Azure authority, `https://login.microsoftonline.com`. The `iss` claim in the token should start with this value.
- The token's `aud` parameter will be set to the application ID of the add-in's registration.
- The token's `scp` parameter will be set to `access_as_user`.

### Using the SSO token as an identity

If your add-in needs to verify the user's identity, the SSO token contains information that can be used to establish the identity. The following claims in the token relate to identity.

- `name` - The user's display name.
- `preferred_username` - The user's login name.
- `oid` - A GUID representing the ID of the user in the Azure Active Directory.
- `tid` - A GUID representing the ID of the user's organization in the Azure Active Directory.

Since the `name` and `preferred_username` values could change, it's recommended that the `oid` and `tid` values be used to correlate the identity with your back-end's authorization service.

For example, your service could format those values together like `{oid-value}@{tid-value}`, then store that as a value on the user's record in your internal user database. Then on subsequent requests, the user could be retrieved by using the same value, and access to specific resources could be determined based on your existing access control mechanisms.

> [!IMPORTANT]
> When using the SSO token as an identity, it is recommended that you also [use the Exchange identity token](authenticate-a-user-with-an-identity-token.md) as an alternate identity. Users of your add-in may use multiple clients, and some may not support providing an SSO token. By using the Exchange identity token as an alternate, you can avoid having to prompt these users for credentials multiple times. For more details, see [](implement-sso-in-outlook-add-in.md).

### Using the SSO token to get a Microsoft Graph token

In order to access the Microsoft Graph API, the add-in sends a [service-to-service request using the On-Behalf-Of flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-on-behalf-of#service-to-service-access-token-request) to the Azure Active Directory v2.0 token endpoint using a shared secret.

The response contains a JSON payload, with the Microsoft Graph token supplied in the `access_token` property. That token is used in the `Authorization` header when making calls to the Microsoft Graph API.

## Resources

For a sample add-in that uses the SSO token to access the Microsoft Graph API, see [AttachmentsDemo Sample Add-in](https://github.com/OfficeDev/outlook-add-in-attachments-demo).
