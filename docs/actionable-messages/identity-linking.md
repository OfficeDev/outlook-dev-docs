---
title: Link Azure Active Directory identity with your own identity provider (Preview)
description: Learn how to authenticate an actionable message recipient with your service to link their AAD identity.
author: jasonjoh
ms.topic: article
ms.technology: o365-connectors
ms.date: 10/31/2019
ms.author: jasonjoh
localization_priority: Normal
---

# Link Azure Active Directory identity with your own identity provider (Preview)

[Action.Http](adaptive-card.md#actionhttp) actions in actionable messages include an AAD-issued token in the `Authorization` header, which provides information about the user's identity. However, this information may not be sufficient to authenticate the user to your service. With identity linking, you can signal the Outlook client to present UI to allow the user to authenticate with your service. Once the user authenticates, you can associate their AAD identity with your own to allow for seamless authentication for future requests.

## Using identity linking

Your service can trigger authentication on any `Action.Http` action endpoint by returning a `401 Unauthorized` response with a `ACTION-AUTHENTICATE` header. The header contains the authentication URL for your service.

Once authentication is completed, redirect the request to the URL specified in the `Post-Identity-Linking-Redirect-Url` header sent in the original request.

## Identity linking flow

### Initial request to your action endpoint

Microsoft servers send an initial POST request to your action endpoint.

```http
POST https://api.contoso.com/myEndpoint
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6...
Post-Identity-Linking-Redirect-Url: https://outlook.office.com/connectors/adelev@contoso.com/723a1c49-f8dc-4063-843e-d4c2b7180b8b/postAuthenticate
Content-Type: application/json

{
  // action body
}
```

Your service [validates the JWT token](security-requirements.md#verifying-that-requests-come-from-microsoft) and extracts the user's identity from the `sub` claim.

```json
{
  ...
  "sub": "AdeleV@contoso.com",
  "aud": "https://api.contoso.com"
}
```

Your service finds no user with a linked identity of `AdeleV@contoso.com` so it persists `Post-Identity-Linking-Redirect-Url` header value and returns a `401` response.

> [!NOTE]
> The exact method you use to persist the redirect URL from the `Post-Identity-Linking-Redirect-Url` header is dependent on your implementation. If your service uses OAuth, you may save it in the `state` parameter, for example.

```http
HTTP/1.1 401 Unauthorized
ACTION-AUTHENTICATE: https://identity.contoso.com/authenticate?state=https://outlook.office.com/connectors/adelev@contoso.com/723a1c49-f8dc-4063-843e-d4c2b7180b8b/postAuthenticate
```

### Authentication request

After Outlook receives the `401` with the `ACTION-AUTHENTICATE` header, it will open a task pane and navigate to the URL from the header.

```http
GET https://identity.contoso.com/authenticate?state=https://outlook.office.com/connectors/adelev@contoso.com/723a1c49-f8dc-4063-843e-d4c2b7180b8b/postAuthenticate
```

Your service authenticates the user and associates the identity provided by the AAD-issued token with the user in your system. Once complete, the service redirects the request to the URL from the `Post-Identity-Linking-Redirect-Url` header.

```http
HTTP/1.1 302 Found
Location: https://outlook.office.com/connectors/adelev@contoso.com/723a1c49-f8dc-4063-843e-d4c2b7180b8b/postAuthenticate
```

### Retry action

After Outlook receives the redirect back from your authentication server, it immediately retries the original request. This time, because you've associated the AAD identity with your own, your endpoint processes the request normally.

## Example

You can use the following sample card in the [Card Playground](https://messagecardplayground.azurewebsites.net/) to see this in action. The endpoint in this card will prompt you to login to the Microsoft identity platform and (with your consent) will make a Graph request to [get your profile](/graph/api/user-get?view=graph-rest-1.0).

```json
{
  "hideOriginalBody": true,
  "type": "AdaptiveCard",
  "padding": "none",
  "body": [
    {
      "type": "TextBlock",
      "text": "Identity Linking Demo"
    },
    {
      "type": "ActionSet",
      "actions": [
        {
          "type": "Action.Http",
          "method": "POST",
          "url": "https://amidentitylinking.azurewebsites.net/action",
          "body": "{}",
          "title": "Get User Details",
          "isPrimary": true
        }
      ]
    }
  ],
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.0"
}
```

## Client support roadmap

Identity linking is available to a limited set of clients, with support for the feature being added in the future. The following table provides the approximate timeline.

| Client                            | Availability            |
|-----------------------------------|-------------------------|
| Office 365 ProPlus                | Available               |
| Outlook on the web for Office 365 | December 2019 (planned) |
| Outlook on iOS                    | January 2020 (planned)  |
| Outlook on Android                | January 2020 (planned)  |
| Outlook on Mac                    | TBD                     |

## Resources

- [Security requirements for actionable messages in Office 365](security-requirements.md)
- [Designing Outlook Actionable Message cards with the Adaptive Card format](adaptive-card.md)
