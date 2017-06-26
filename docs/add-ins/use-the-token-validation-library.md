---
title: Use the .NET token validation library to validate Outlook add-in tokens | Microsoft Docs
description: Learn how to use the .NET token validation library to validate Outlook add-in identity tokens.
author: jasonjoh

ms.topic: article
ms.technology: office-add-ins
ms.date: 06/13/2017
ms.author: jasonjoh
---

# Use the Exchange Web Services Managed API token validation library

You can identify the clients of your Outlook add-in by using an identity token that your add-in requests from a server running Exchange Server 2013 or Exchange Online. The token, formatted as a JSON Web token, provides a unique identifier for an email account on an Exchange server. The Exchange Web Services (EWS) Managed API provides helper classes to simplify the use of the identity token.

## Prerequisites for using the validation library

To validate an Exchange identity token, you must install the [EWS Managed API library](https://www.nuget.org/packages/Microsoft.Exchange.WebServices).

## Validate the Exchange identity token

The EWS Managed API validation library provides the  **AppIdentityToken** class to manage the Exchange identity tokens. The following method shows how to create an **AppIdentityToken** instance and call the **Validate** method to verify that the token is valid. The method takes the following parameters:

- *rawToken*: The string representation of the token returned in your Outlook add-in from the [**Office.context.mailbox.getUserIdentityTokenAsync**](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5) method.
- *hostUri*: The fully qualified URI to the page in your Outlook add-in that called **getUserIdentityTokenAsync**.

```C#
// Required to use the validation library.
using Microsoft.Exchange.WebServices.Auth.Validate;

private AppIdentityToken CreateAndValidateIdentityToken(string rawToken, string hostUri)
{
    try
    {
        AppIdentityToken token = (AppIdentityToken)AuthToken.Parse(rawToken);
        token.Validate(new Uri(hostUri));

        return token;
    }
    catch (TokenValidationException ex)
    {
        throw new ApplicationException("A client identity token validation error occurred.", ex);
    }
}
```

## Additional resources

- [Authenticate an Outlook add-in by using Exchange identity tokens](authentication.md)  
- [Inside the Exchange identity token](inside-the-identity-token.md)
- [Validate an Exchange identity token](validate-an-identity-token.md)
    
