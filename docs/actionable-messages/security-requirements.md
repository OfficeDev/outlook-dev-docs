---
title: Security requirements for actionable messages | Microsoft Docs
description: Learn about security requirements for actionable messages and how to validate the bearer token sent by Office 365.
author: jasonjoh

ms.topic: article
ms.technology: o365-connectors
ms.date: 05/22/2018
ms.author: jasonjoh
---
# Security requirements for actionable messages in Office 365

Securing actionable email is simple and easy. There are two phases within the end-to-end experience that impose security requirements on your service when supporting actionable messages with Office 365. The phases and their corresponding requirements are as follows.

1. Send phase: The pre-requisites for your service to send actionable messages are as follows:
    - If you're using actionable email, check that your mail servers are leveraging SPF and DKIM as an additional safeguard against sender email spoofing (majority of mail systems do that already). Note that this does not apply to connector messages.
    - Your service must be registered with Microsoft.
    - The Action URL must support HTTPS.
1. Action processing phase: When processing an action, your service should:
    - Verify the bearer token (a JSON Web token) included in the header of the HTTP POST request. Verification can also be done leveraging the sample libraries provided by Microsoft.
    - Include Limited Purpose Token from your service as part of the target URL, which can be used by your service to correlate the service URL with the intended request & user. This is optional, but highly recommended.

## Bearer Token

All action requests from Microsoft have a bearer token in the HTTP `Authorization` header. This token is a [JSON Web Token](https://jwt.io/) (JWT) token signed by Microsoft, and it includes important claims that we strongly recommend should be verified by the service handling the associated request.

| Claim name | Value |
|------------|-------|
| `aud` | The base URL of the target service, e.g. `https://api.contoso.com` |
| `sub` | The identity of the user who took the action. For Actionable Messages sent over email, `sub` would be the email address of the user. For connectors, `sub` will be the objectID of the user who took the action. |
| `sender` | The identity of sender of the message containing the action. |

Typically, a service will perform the following verifications.

1. The token is signed by Microsoft.
1. The `aud` claim corresponds to the service's base URL.

With all the above verifications done, the service can trust the `sender` and `sub` claims to be the identity of the sender and the user taking the action. The service can validate that the `sender` and `sub` claims match the sender and user it is expecting.

Please refer to the Microsoft code samples provided below, which show how to do these validations on the JWT token.

- [.NET Sample](https://github.com/OfficeDev/outlook-actionable-messages-csharp-token-validation)
- [Node.js Sample](https://github.com/OfficeDev/outlook-actionable-messages-node-token-validation)
- [Java Sample](https://github.com/OfficeDev/outlook-actionable-messages-java-token-validation)
- [Python Sample](https://github.com/OfficeDev/outlook-actionable-messages-python-token-validation)

### Action-Authorization header

The use of `Authorization` header by Actionable messages may interfere with existing authentication/authorization mechanism for the target endpoint. In this case, developers can set the `Authorization` header to `null` or an empty string in the `headers` property of an `Action.Http` action. Actionable messages will then send the same bearer token via `Action-Authorization` header instead of using `Authorization` header.

> [!TIP]
> The Azure Logic App service returns `HTTP 401 Unauthorized` if the `Authorization` header contains the bearer token set by actionable messages.

#### Example Action.Http with Action-Authorization header

```json
{
  "type": "Action.Http",
  "title": "Say hello",
  "method": "POST",
  "url": "https://api.contoso.com/sayhello",
  "body": "{{nameInput.value}}",
  "headers": [
    { "name": "Authorization", "value": "" }
  ]
}
```

## Limited-Purpose Tokens

Limited purpose tokens can be used to correlate service URLs with specific messages and users. Microsoft recommends that service providers use "limited-purpose tokens" as part of the action target URL, or in the body of the request coming back to the service. For example:

```http
https://contoso.com/approve?requestId=abc&limitedPurposeToken=a1b2c3d4e5...
```

Limited-purpose tokens act as correlation IDs (for e.g. a hashed token using the userID, requestID, and salt). This allows the service provider to keep track of the action URLs it generates and sends out and match it with action requests coming in. In addition to correlation, the service provider may use the limited purpose token to protect itself from replay attacks. For example, the service provider may choose to reject the request, if the action was already performed previously with the same token.

Microsoft does not prescribe how the limited-access tokens should be designed or used by the service. This token is opaque to Microsoft, and is simply echoed back to the service.
