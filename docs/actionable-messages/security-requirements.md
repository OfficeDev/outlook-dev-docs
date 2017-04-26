---
title: Security requirements for actionable messages | Microsoft Docs
description: Learn about security requirements for actionable messages and how to validate the bearer token sent by Office 365.
author: jasonjoh

ms.topic: article
ms.technology: office-365-connectors
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Security requirements for actionable messages

Securing actionable email is simple and easy. There are two phases within the end-to-end experience that impose security requirements on your service when supporting actionable messages with Office 365. The phases and their corresponding requirements are as follows.

1. Send phase: The pre-requisites for your service to send actionable messages are as follows:
    - If you're using actionable email, check that your mail servers are leveraging SPF and DKIM as an additional safeguard against sender email spoofing (majority of mail systems do that already). Note that this does not apply to connector messages.
    - Your service must be registered with Microsoft.
    - The Action URL must support HTTPS.
2.	Action processing phase: When processing an action, your service should:
    - Verify the bearer token (a JSON Web token) included in the header of the HTTP POST request. Verification can also be done leveraging the sample libraries provided by Microsoft.
    - Include Limited Purpose Token from your service as part of the target URL, which can be used by your service to correlate the service URL with the intended request & user. This is optional, but highly recommended.

## Bearer Token

All action requests from Microsoft have a bearer token (JWT) in the HTTP authorization header. This token is a JWT token signed by Microsoft. The token also provides information about the user who took the action. It is strongly recommended that services handling these requests verify the bearer token.

The description of important claims available in this token is given below.

| Claim Type | Description | Value |
|------------|-------------|-------|
| 'aud' | The audience claim identifies the target service that this token is intended for. | The domain of the service API URL, as specified by target URL in the actions markup. |
| 'sub' | Subject | The email address of the user who took the action **Note:** for connectors, the `sub` claim will have the objectID of the user who took the action. |
| 'sender ' | Sender of the message containing the action. | The email address of the sender of the message containing the action. |


The token also provides important information (claims) about the user who took the action and the sender who sent the action request.

Service providers handling these requests must verify the bearer token and its claims.

1. The token is in JWT format and is signed by Azure Active Directory.
1. Verify the 'aud' claim and make sure that it is what the service API is expecting.
1. Verify the 'sender' and 'sub' claims are what the service API expects.

Please refer to the Microsoft code samples provided below, which show how to do these validations on the JWT token.

- [.NET Sample](https://github.com/OfficeDev/outlook-actionable-messages-csharp-token-validation)
- [Node.js Sample](https://github.com/OfficeDev/outlook-actionable-messages-node-token-validation)
- [Java Sample](https://github.com/OfficeDev/outlook-actionable-messages-java-token-validation)
- [Python Sample](https://github.com/OfficeDev/outlook-actionable-messages-python-token-validation)

### C# Sample

```csharp
public async Task<HttpResponseMessage> Post([FromBody]string value)
{
    HttpRequestMessage request = this.ActionContext.Request;
    
    // Validate that we have a bearer token.
    if (request.Headers.Authorization == null ||
        !string.Equals(request.Headers.Authorization.Scheme, "bearer", StringComparison.OrdinalIgnoreCase) ||
        string.IsNullOrEmpty(request.Headers.Authorization.Parameter))
    {
        return request.CreateErrorResponse(HttpStatusCode.Unauthorized, "Bearer token not found.");
    }

    // Validate that the bearer token is valid.
    string bearerToken = request.Headers.Authorization.Parameter;
    ActionableMessageTokenValidator validator = new ActionableMessageTokenValidator();
    ActionableMessageTokenValidationResult result = await validator.ValidateTokenAsync(bearerToken, "https://api.contoso.com");
    if (!result.ValidationSucceeded)
    {
        if (result.Exception != null)
        {
            Trace.TraceError(result.Exception.ToString());
        }

        return request.CreateErrorResponse(HttpStatusCode.Unauthorized, "Invalid bearer token");
    }

    // We have a valid token. We will next verify the sender and the action performer.
    // In this example, we verify that the email is sent by Contoso LOB system
    // and the action performer is john@contoso.com.
    if (!string.Equals(result.Sender, @"lob@contoso.com") ||
        !string.Equals(result.ActionPerformer, "john@contoso.com")
    {
        return request.CreateErrorResponse(HttpStatusCode.Forbidden, string.Empty);
    }

    // Process the request.
    
    return Request.CreateResponse(HttpStatusCode.OK);
}
```

## Limited-Purpose Tokens

Limited purpose tokens can be used to correlate service URLs with specific messages and users. Microsoft recommends that service providers use "limited-purpose tokens" as part of the action target URL, or in the body of the request coming back to the service. For example:

```
https://contoso.com/approve?requestId=abc&limitedPurposeToken=a1b2c3d4e5...
```

Limited-purpose tokens act as correlation IDs (for e.g. a hashed token using the userID, requestID, and salt). This allows the service provider to keep track of the action URLs it generates and sends out and match it with action requests coming in. In addition to correlation, the service provider may use the limited purpose token to protect itself from replay attacks. For example, the service provider may choose to reject the request, if the action was already performed previously with the same token. 

Microsoft does not prescribe how the limited-access tokens should be designed or used by the service. This token is opaque to Microsoft, and is simply echoed back to the service.