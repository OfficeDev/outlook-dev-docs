---
title: Authentication options in Outlook Add-ins | Microsoft Docs
description: Learn about the ways to authenticate a user in an Outlook Add-in.
author: jasonjoh
ms.topic: article
ms.technology: office-add-ins
ms.date: 09/13/2017
ms.author: jasonjoh
---

# Authentication options in Outlook Add-ins

Your Outlook Add-in can access information from anywhere on the internet, whether from the server that hosts the add-in, from your internal network, or from somewhere else in the cloud. If that information is protected, your add-in needs a way to authenticate your user. Outlook Add-ins provide a number of different methods to authenticate, depending on your specific scenario.

## Single sign-on access token

Single sign-on access tokens provide a seamless way for your add-in to authenticate and obtain access tokens to call the [Microsoft Graph API](https://developer.microsoft.com/en-us/graph/docs/concepts/overview). This capability reduces friction since the user is not required to enter their credentials. Consider using SSO access tokens if your add-in:

- Is used primarily by Office 365 users
- Needs access to:
    - Microsoft services that are exposed as part of Microsoft Graph
    - A non-Microsoft service that you control

The SSO authentication method uses the [OAuth2 On-Behalf-Of flow provided by Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols-oauth-on-behalf-of). It requires that the add-in register in the [Application Registration Portal](https://apps.dev.microsoft.com/) and specify any required Microsoft Graph scopes in its manifest.

Using this method, your add-in can obtain an access token scoped to your server back-end API. The add-in uses this as a bearer token in the `Authorization` header to authenticate a call back to your API. At that point your server can:

- Complete the On-Behalf-Of flow to obtain an access token scoped to the Microsoft Graph API
- Use the identity information in the token to establish the user's identity and authenticate to your own back-end services

For a more detailed overview, see the [full overview of the SSO authentication method](https://docs.microsoft.com/office/dev/add-ins/develop/sso-in-office-add-ins).

For details on using the SSO token in an Outlook Add-in, see [Authenticate a user with an single-sign-on token in an Outlook Add-in](authenticate-a-user-with-an-sso-token.md).

For a sample add-in that uses the SSO token, see [AttachmentsDemo Sample Add-in](https://github.com/OfficeDev/outlook-add-in-attachments-demo).

## Exchange user identity token

Exchange user identity tokens provide a way for your add-in to establish the identity of the user. By verifying the user's identity, you can then perform a one-time authentication into your back-end system, then accept the user identity token as an authorization for future requests. Consider using user identity tokens if your add-in:

- Is used primarily by Exchange on-premises users
- Needs access to a non-Microsoft service that you control

## Access tokens obtained via OAuth2 flows

Add-ins can also access third-party services that support OAuth2 for authorization. Consider using OAuth2 tokens if your add-in:

- Needs access to a third-party service outside of your control

Using this method, your add-in prompts the user to sign-in to the service either by using the [displayDialogAsync](https://dev.office.com/reference/add-ins/shared/officeui.displaydialogasync?product=outlook) method to initialize the OAuth2 flow, or by using the [office-js-helpers library](https://github.com/OfficeDev/office-js-helpers) to the OAuth2 Implicit flow.

## Callback tokens

Callback tokens provide access to the user's mailbox from your server back-end, either using [Exchange Web Services (EWS)](https://docs.microsoft.com/en-us/exchange/client-developer/exchange-web-services/explore-the-ews-managed-api-ews-and-web-services-in-exchange), or the [Outlook REST API](https://msdn.microsoft.com/en-us/office/office365/api/use-outlook-rest-api). Consider using callback tokens if your add-in:

- Needs access to the user's mailbox from your server back-end.

Add-ins obtain callback tokens using the [getCallbackTokenAsync method](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook). The level of access is controlled by the permissions specified in the add-in manifest.