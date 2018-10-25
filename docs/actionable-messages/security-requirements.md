---
title: Security requirements for actionable messages | Microsoft Docs
description: Learn about security requirements for actionable messages and how to validate the bearer token sent by Office 365.
author: jasonjoh
ms.topic: article
ms.technology: o365-connectors
ms.date: 10/25/2018
ms.author: jasonjoh
---

# Security requirements for actionable messages in Office 365

Securing actionable email is simple and easy. There are two phases within the end-to-end experience that impose security requirements on your service when supporting actionable messages with Office 365. The phases and their corresponding requirements are as follows.

1. Send phase: The pre-requisites for your service to send actionable messages are as follows:
    - If you're using actionable email, you'll need to enable [sender verification](#sender-verification). Note that this does not apply to connector messages.
    - Your service must be registered with Microsoft.
    - The Action URL must support HTTPS.
1. Action processing phase: When processing an action, your service should:
    - Verify the bearer token (a JSON Web token) included in the header of the HTTP POST request. Verification can also be done leveraging the sample libraries provided by Microsoft.
    - Include Limited Purpose Token from your service as part of the target URL, which can be used by your service to correlate the service URL with the intended request & user. This is optional, but highly recommended.

## Sender verification

Office 365 requires sender verification in order to enable actionable messages via email. Your actionable message emails must either originate from servers that implement DomainKeys Identified Mail (DKIM) and Sender Policy Framework (SPF), or you must implement [signed cards](#signed-card-payloads).

While DKIM and SPF are sufficient for some scenarios, that solution will not work in some situations where emails are sent via an external provider, which can lead to recipients not experiencing the enhanced actionable message. For this reason, we recommend always implementing signed cards which work in all cases and are fundamentally more secure since they do not rely on DNS records.

### Implementing DKIM and SPF

DKIM and SPF are industry standard ways to prove a sender's identity when sending emails over SMTP. Many companies already implement these standards to secure the emails they are already sending. To learn more about SPF/DKIM and how to implement them, see:

- [DomainKeys Identified Mail (DKIM)](http://www.dkim.org/)
- [Sender Policy Framework](http://www.openspf.org/)

### Signed card payloads

Actionable messages [sent via email](actionable-messages-via-email.md) support an alternative verification method: signing the card payload with and RSA key or X509 certificate. This method is required in the following scenarios:

- The sending servers do not support DKIM or SPF verification.
- You scenario for actionable messages requires sending from multiple email accounts.

Using signed card payloads requires onboarding with Microsoft. Please contact [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com) for more information.

#### SignedCard

Signed actionable message cards are available when sending via email. Use this format to include a signed card in the HTML body of an email. This payload is serialized as JSON inside of a `<script>` tag of type `application/ld+json` in the `<head>` of the HTML body.

| Field | Type | Description |
|-------|------|-------------|
| `@type` | String | Required. MUST be set to either `SignedAdaptiveCard` when using the [Adaptive card format](adaptive-card.md), or `SignedMessageCard` when using the [MessageCard format](message-card-reference.md). |
| `@context` | String | Required. MUST be set to `http://schema.org/extensions` |
| `signedAdaptiveCard` | String | Required if `@type` is set to `SignedAdaptiveCard`. MUST be set to a signed [SignedCardPayload](#signedcardpayload), signed using the [JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515) standard. |
| `signedMessageCard` | String | Required if `@type` is set to `SignedMessageCard`. MUST be set to a signed [SignedCardPayload](#signedcardpayload), signed using the [JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515) standard. |

##### SignedCardPayload

| Field | Type | Description |
|-------|------|-------------|
| `originator` | String | Required. MUST be set to the ID provided by Microsoft during onboarding. |
| `iat` | TimeStamp (Unix epoch) | Required. The time that the payload was signed. |
| `sender` | String | Required. The email address used to send this actionable message. |
| `recipientsSerialized` | String | A serialized [RecipientList](#recipentlist) containing all of the To and CC recipients of the message. |
| `sub` | String | Required. A unique identifier in your backend system for the recipient. |
| `adaptiveCardSerialized` | String | Required if using the Adaptive card format, otherwise MUST NOT be present. The serialized content of the Adaptive card JSON. |
| `messageCardSerialized` | String | Required if using the MessageCard format, otherwise MUST NOT be present. The serialized content of the Adaptive card JSON. |

###### RecipientList

| Field | Type | Description |
|-------|------|-------------|
| `recipients` | Array of String | Required. Contains the email addresses of the To and CC recipients for the actionable message. |

## Verifying that requests come from Microsoft

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

## Verifying the identity of the user

The bearer token included with all requests includes the Azure AD identity of the Office 365 user who took the action. If necessary, you can include your own token, specific to your service, in the URLs of your HTTP actions to represent the identity of the user in your system. Microsoft does not prescribe how the limited-access tokens should be designed or used by the service. This token is opaque to Microsoft, and is simply echoed back to the service.

Service-specific tokens can be used to correlate service URLs with specific messages and users. Microsoft recommends that if you use your own service-specific token, youinclude it as part of the action target URL, or in the body of the request coming back to the service. For example:

```http
https://contoso.com/approve?requestId=abc&serviceToken=a1b2c3d4e5...
```

Service-specific tokens act as correlation IDs (for e.g. a hashed token using the userID, requestID, and salt). This allows the service provider to keep track of the action URLs it generates and sends out and match it with action requests coming in. In addition to correlation, the service provider may use the service-specific token to protect itself from replay attacks. For example, the service provider may choose to reject the request, if the action was already performed previously with the same token.