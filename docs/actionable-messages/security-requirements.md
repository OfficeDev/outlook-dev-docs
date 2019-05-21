---
title: Security requirements for actionable messages
description: Learn about security requirements for actionable messages and how to validate the bearer token sent by Office 365.
author: jasonjoh
ms.topic: article
ms.technology: o365-connectors
ms.date: 10/25/2018
ms.author: jasonjoh
localization_priority: Priority
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
- [Sender Policy Framework](https://en.wikipedia.org/wiki/Sender_Policy_Framework)

### Signed card payloads

Actionable messages [sent via email](send-via-email.md) support an alternative verification method: signing the card payload with and RSA key or X509 certificate. This method is required in the following scenarios:

- SPF/DKIM failure caused by sender setup or recipient tenant set custom security services in front of Office 365 services.
- You scenario for actionable messages requires sending from multiple email accounts.

Using signed card payloads requires onboarding with Microsoft. Please contact [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com) for more information.

#### SignedCard

Signed actionable message cards are available when sending via email. Use this format to include a signed card in the HTML body of an email. This payload is serialized in Microdata format appended in the end of HTML body.

```html
<section itemscope itemtype="http://schema.org/SignedAdaptiveCard">
    <meta itemprop="@context" content="http://schema.org/extensions" />
    <meta itemprop="@type" content="SignedAdaptiveCard" />
    <div itemprop="signedAdaptiveCard" style="mso-hide:all;display:none;max-height:0px;overflow:hidden;">[SignedCardPayload]</div>
</section>
```
> Note: Partners who prefer to use the legacy MessageCard entity may create a SignedMessageCard entity in place of a SignedAdaptiveCard.

##### SignedCardPayload

SignedCardPayload is a string encoded by JSON Web Signature (JWS) standard. [RFC7515](https://tools.ietf.org/html/rfc7515) describes JWS, and [RFC7519](https://tools.ietf.org/html/rfc7519) describes JSON Web Token (JWT). Given no claim is required in JWT, JWT libraries can be used to build JWS signature.

> Note: The term "JWT" can be used interchangeably in practice. However, we prefer the term "JWS" here. 

Here is an example of SignedCardPayload. The encoded Adaptive Card appears in the form of [header].[payload].[signature] as per JWS specification.

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzZW5kZXIiOiJzZXJ2aWNlLWFjY291bnRAY29udG9zby5jb20iLCJvcmlnaW5hdG9yIjoiNjVjNjgwZWYtMzZhNi00YTFiLWI4NGMtYTdiNWM2MTk4NzkyIiwicmVjaXBpZW50c1NlcmlhbGl6ZWQiOiJbXCJqb2huQGNvbnRvc28uY29tXCIsXCJqYW5lQGNvbnRvc28uY29tXCJdIiwiYWRhcHRpdmVDYXJkU2VyaWFsaXplZCI6IntcIiRzY2hlbWFcIjpcImh0dHA6Ly9hZGFwdGl2ZWNhcmRzLmlvL3NjaGVtYXMvYWRhcHRpdmUtY2FyZC5qc29uXCIsXCJ0eXBlXCI6XCJBZGFwdGl2ZUNhcmRcIixcInZlcnNpb25cIjpcIjEuMFwiLFwiYm9keVwiOlt7XCJzaXplXCI6XCJsYXJnZVwiLFwidGV4dFwiOlwiSGVsbG8gQWN0aW9uYWJsZSBtZXNzYWdlXCIsXCJ3cmFwXCI6dHJ1ZSxcInR5cGVcIjpcIlRleHRCbG9ja1wifV0sXCJhY3Rpb25zXCI6W3tcInR5cGVcIjpcIkFjdGlvbi5JbnZva2VBZGRJbkNvbW1hbmRcIixcInRpdGxlXCI6XCJPcGVuIEFjdGlvbmFibGUgTWVzc2FnZXMgRGVidWdnZXJcIixcImFkZEluSWRcIjpcIjNkMTQwOGY2LWFmYjMtNGJhZi1hYWNkLTU1Y2Q4NjdiYjBmYVwiLFwiZGVza3RvcENvbW1hbmRJZFwiOlwiYW1EZWJ1Z2dlck9wZW5QYW5lQnV0dG9uXCJ9XX0iLCJpYXQiOjE1NDUzNDgxNTN9.BP9mK33S1VZyjtWZd-lNTTjvueyeeoitygw9bl17TeQFTUDh9Kg5cB3fB7BeZYQs6IiWa1VGRdiiR4q9EkAB1qDsmIcJnw6aYwDUZ1KY4lNoYgQCH__FxEPHViGidNGtq1vAC6ODw0oIfaTUWTa5cF5MfiRBIhpQ530mbRNnA0QSrBYtyB54EDJxjBF1vNSKOeVHAl2d4gqcGxsytQA0PA7XMbrZ8B7fEU2uNjSiLQpoh6A1tevpla2C7W6h-Wekgsmjpw2YToAOX67VZ1TcS5oZAHmjv2RhqsfX5DlN-ZsTRErU4Hs5d92NY9ijJPDunSLyUFNCw7HLNPFqqPmZsw
```

The header in above JWS is:

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```
 
The payload in above JWS is:

```json
{
  "sender": "service-account@contoso.com",
  "originator": "65c680ef-36a6-4a1b-b84c-a7b5c6198792",
  "recipientsSerialized": "[\"john@contoso.com\",\"jane@contoso.com\"]",
  "adaptiveCardSerialized": "{\"$schema\":\"http://adaptivecards.io/schemas/adaptive-card.json\",\"type\":\"AdaptiveCard\",\"version\":\"1.0\",\"body\":[{\"size\":\"large\",\"text\":\"Hello Actionable message\",\"wrap\":true,\"type\":\"TextBlock\"}],\"actions\":[{\"type\":\"Action.InvokeAddInCommand\",\"title\":\"Open Actionable Messages Debugger\",\"addInId\":\"3d1408f6-afb3-4baf-aacd-55cd867bb0fa\",\"desktopCommandId\":\"amDebuggerOpenPaneButton\"}]}",
  "iat": 1545348153
}
```

###### Required Claims

| Claim | Description |
|-------|-------------|
| `originator` | MUST be set to the ID provided by Microsoft during onboarding. |
| `iat` | The time that the payload was signed. |
| `sender` | The email address used to send this actionable message. |
| `recipientsSerialized` | The stringified list of the recipients of the email. This should include all the To/CC recipients of the email. |
| `adaptiveCardSerialized` | The stringfied Adaptive Card. |

Sample code generating signed card:

- [.NET Sample](https://github.com/tony-zhu/SignedAdaptiveCardSample-dotnet)
- [Node.js Sample](https://github.com/tony-zhu/SignedAdaptiveCardSample-node)

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
