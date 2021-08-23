---
title: Get Started with the Outlook REST APIs
description: Learn how to use Microsoft Graph via REST requests and responses to access the Outlook API.
author: jasonjoh

ms.topic: article
ms.technology: ms-graph
ms.devlang: rest-api
ms.date: 02/19/2020
ms.author: jasonjoh
ms.localizationpriority: high
---
# Overview of using the Outlook REST APIs

> [!TIP]
> Try out sample REST calls in the [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer). You can use your own account, or one of our test accounts. Once you're done exploring the API, come back here and select your favorite platform on the left. We'll guide you through the steps to write a simple application to retrieve messages from your inbox.
>
> If your preferred platform isn't listed yet, continue reading on this page. We'll go through the same set of steps using raw HTTP requests.

The purpose of this guide is to walk through the process of calling the [Outlook Mail API](/graph/api/resources/message?view=graph-rest-1.0) to retrieve messages in Office 365 and Outlook.com. Unlike the platform-specific getting started guides, this guide focuses on the OAuth and REST requests and responses. It will cover the sequence of requests and responses that an app uses to authenticate and retrieve messages.

This tutorial will use [Microsoft Graph](/graph/overview) to call the Mail API. Microsoft recommends using Microsoft Graph to access Outlook mail, calendar, and contacts. You should use the Outlook APIs directly (via `https://outlook.office.com/api`) only if you require a feature that is not available on the Graph endpoints.

With the information in this guide, you can implement this in any language or platform capable of sending HTTP requests.

## Use OAuth2 to authenticate

In order to call the Mail API, the app requires an access token from the Microsoft identity platform. Use one of the [supported OAuth 2.0 flows](/azure/active-directory/develop/active-directory-v2-protocols) to obtain an access token.

## Calling the Mail API

Once the app has an access token, it's ready to call the Mail API. The [Mail API Reference](/graph/api/resources/message?view=graph-rest-1.0) has all of the details. Since the app is retrieving messages, it will use an HTTP GET request to the `https://graph.microsoft.com/v1.0/me/mailfolders/inbox/messages` URL. This will retrieve messages from the inbox.

### Refining the request

Apps can control the behavior of GET requests by using [OData query parameters](/graph/query-parameters). It is recommended that apps use these parameters to limit the number of results that are returned and to limit the fields that are returned for each item. Let's look at an example.

Consider an app that displays messages in a table. The table only displays the subject, sender, and the date and time the message was received. The table displays a maximum of 25 rows, and should be sorted so that the most recently received message is at the top.

To achieve this, the app uses the following query parameters:

- The `$select` parameter is used to specify only the `subject`, `from`, and `receivedDateTime` fields.
- The `$top` parameter is used to specify a maximum of 25 items.
- The `$orderby` parameter is used to sort the results by the `receivedDateTime` field.

This results in the following request.

#### Mail API request for messages in the inbox

```http
GET https://graph.microsoft.com/v1.0/me/mailfolders/inbox/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC

Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

#### Mail API Response

```http
HTTP/1.1 200 OK
Content-Type: application/json;odata.metadata=minimal;odata.streaming=true;IEEE754Compatible=false;charset=utf-8

{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(...)/mailfolders('inbox')messages(subject,from,receivedDateTime)",
  "value": [
    {
      "@odata.etag": "W/\"CQAAABYAAAAoPBSqxXQOT6tuE0pxCMrtAABufX4i\"",
      "id": "AAMkADRmMDExYzhjLWYyNGMtNDZmMC1iZDU4LTRkMjk4YTdjMjU5OABGAAAAAABp4MZ-5xP3TJnNAPmjsRslBwAoPBSqxXQOT6tuE0pxCMrtAAAAAAEMAAAoPBSqxXQOT6tuE0pxCMrtAABufW1UAAA=",
      "subject": "Ruby on Rails tutorial",
      "from": {
        "emailAddress": {
          "address": "jason@contoso.onmicrosoft.com",
          "name": "Jason Johnston"
        }
      },
      "receivedDateTime": "2015-01-29T20:44:53Z"
    },
    {
      "@odata.etag": "W/\"CQAAABYAAAAoPBSqxXQOT6tuE0pxCMrtAABSzmz4\"",
      "id": "AAMkADRmMDExYzhjLWYyNGMtNDZmMC1iZDU4LTRkMjk4YTdjMjU5OABGAAAAAABp4MZ-5xP3TJnNAPmjsRslBwAoPBSqxXQOT6tuE0pxCMrtAAAAAAEMAAAoPBSqxXQOT6tuE0pxCMrtAABMirSeAAA=",
      "subject": "Trip Information",
      "from": {
        "emailAddress": {
          "address": "jason@contoso.onmicrosoft.com",
          "name": "Jason Johnston"
        }
      },
      "receivedDateTime": "2014-12-09T21:55:41Z"
    },
    {
      "@odata.etag": "W/\"CQAAABYAAAAoPBSqxXQOT6tuE0pxCMrtAABzxiLG\"",
      "id": "AAMkADRmMDExYzhjLWYyNGMtNDZmMC1iZDU4LTRkMjk4YTdjMjU5OABGAAAAAABp4MZ-5xP3TJnNAPmjsRslBwAoPBSqxXQOT6tuE0pxCMrtAAAAAAEMAAAoPBSqxXQOT6tuE0pxCMrtAABAblZoAAA=",
      "subject": "Multiple attachments",
      "from": {
        "emailAddress": {
          "address": "jason@contoso.onmicrosoft.com",
          "name": "Jason Johnston"
        }
      },
      "receivedDateTime": "2014-11-19T20:35:59Z"
    },
    {
      "@odata.etag": "W/\"CQAAABYAAAAoPBSqxXQOT6tuE0pxCMrtAAA9yBBa\"",
      "id": "AAMkADRmMDExYzhjLWYyNGMtNDZmMC1iZDU4LTRkMjk4YTdjMjU5OABGAAAAAABp4MZ-5xP3TJnNAPmjsRslBwAoPBSqxXQOT6tuE0pxCMrtAAAAAAEMAAAoPBSqxXQOT6tuE0pxCMrtAAA9x_8YAAA=",
      "subject": "Attachments",
      "from": {
        "emailAddress": {
          "address": "jason@contoso.onmicrosoft.com",
          "name": "Jason Johnston"
        }
      },
      "receivedDateTime": "2014-11-18T20:38:43Z"
    }
  ]
}
```

Now that you've seen how to make calls to the Mail API, you can use the API reference to construct any other kinds of calls your app needs to make. However, bear in mind that your app needs to have the appropriate permissions configured on the app registration for the calls it makes.
