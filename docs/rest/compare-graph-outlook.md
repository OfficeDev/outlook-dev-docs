---
title: Compare Microsoft Graph and Outlook endpoints | Microsoft Docs
description: Compare the differences between Graph and Outlook APIs. Learn which endpoint is the best fit for your scenario and how to translate between them.
author: jasonjoh

ms.topic: article
ms.technology: ms-graph
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Compare Microsoft Graph and Outlook REST API endpoints

The Outlook REST APIs are available in both [Microsoft Graph](https://developer.microsoft.com/en-us/graph/) and the Outlook API endpoint (`https://outlook.office.com/api`). The APIs generally provide the same functionality and use the same resource types.

## Which endpoint should I use?

Use Microsoft Graph whenever possible. The Microsoft Graph endpoint lets you access Outlook and many more [services and features](https://developer.microsoft.com/graph/docs/concepts/overview-major-services), including other Office 365 services, Enterprise Mobility + Security, and Windows 10. Choosing the Microsoft Graph endpoint allows your app to get an access token that can provide access to both Outlook data and other resources, without having to make multiple token requests.

## Feature differences

There are some features that are currently either only available on the Outlook endpoint, or are only in beta in Microsoft Graph. If your app needs these features, you should access them via the Outlook endpoint.

> **Note:** We are constantly working to incorporate all of the features currently available on the Outlook endpoint into Microsoft Graph. Be sure to check back periodically as this list is updated.

| Feature | Difference between endpoints |
|---------|-------------|
| [Outlook tasks](/previous-versions/office/office-365-api/api/version-2.0/task-rest-operations) | The Outlook API provides access to user's tasks. This feature is currently only available in beta in Microsoft Graph. |
| Attachments over 4MB in size | Microsoft Graph cannot create attachments over 4MB in size. Attempts to create an attachment larger than 4MB results in a `413 Request Entity Too Large` error. |
| [Rich notifications](/previous-versions/office/office-365-api/api/version-2.0/notify-rest-operations#get-instance-properties-by-subscribing-to-rich-notifications) | The Outlook API allows developers to request specific fields to be included with the notification payload by using the `$select` parameter. Microsoft Graph does not support this feature. |
| [Streaming notifications](/previous-versions/office/office-365-api/api/beta/notify-streaming-rest-operations) | The Outlook API supports streaming notifications in preview on the beta endpoint. Microsoft Graph does not support this feature. |

## Moving from Outlook endpoint to Microsoft Graph

The APIs are very similar on the Microsoft Graph endpoint and the Outlook endpoint. However, there are some differences to be aware of, especially if you are migrating to Microsoft Graph or using both endpoints in the same application.

### API versions

The Microsoft Graph API offers two versions: `v1.0` and `beta`, while Outlook offers `v1.0`, `v2.0`, and `beta`. Microsoft Graph `v1.0` matches Outlook `v2.0`, and Microsoft Graph `beta` matches Outlook `beta`. The Outlook `v1.0` version is being deprecated.

### OAuth scopes

While the Microsoft Graph and Outlook endpoints both rely on Azure AD-issued tokens, and the [permissions](https://developer.microsoft.com/graph/docs/concepts/permissions_reference) used are the same, the way that your application requests those permissions is slightly different for each endpoint.

#### Azure v2 OAuth2 endpoint

Apps that use the [Azure AD v2.0 endpoint](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview) for OAuth2 request permission scopes in the `scope` parameter in an authentication or token request.

- For Microsoft Graph, apps specify permissions prefixed with `https://graph.microsoft.com/`. For example, an app can request the `Mail.Read` permission by including `https://graph.microsoft.com/Mail.Read` in the `scope` parameter.
- For the Outlook endpoint, apps specify permissions prefixed with `https://outlook.office.com/`. For example, an app can request the `Mail.Read` permission by including `https://outlook.office.com/Mail.Read` in the `scope` parameter.

> **Note:** If a domain is not included in a scope, Microsoft Graph is assumed.

Tokens issued for one endpoint are not valid for the other. Additionally, you cannot mix permissions for one endpoint with permissions for the other in a single request.

The Azure AD v2.0 endpoint only supports the [client credentials flow](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-protocols-oauth-client-creds) for the Microsoft Graph endpoint. Apps that need to use an app-only token with the Outlook endpoint must use the Azure AD v1.0 endpoint.

#### Azure v1 OAuth2 endpoint

Apps that use the [Azure AD v1.0 endpoint](https://docs.microsoft.com/azure/active-directory/develop/active-directory-developers-guide) specify their required permissions during app registration in the Azure Portal.

- For Microsoft Graph, choose the **Microsoft Graph** API when adding required permissions.
- For the Outlook endpoint, choose the **Office 365 Exchange Online** API when adding required permissions.

### Resource property names

The resources are the same between Microsoft Graph and Outlook. However, the two endpoints handle casing of the property names differently. Microsoft Graph uses camelCase for property names, while Outlook uses PascalCase. Translating between the two simply requires converting the case.

For example, the Microsoft Graph [message resource](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/message) defines properties such as `subject`, `from`, and `receivedDateTime`. On the Outlook endpoint, these properties are named `Subject`, `From`, and `ReceivedDateTime`.

### Tracking changes (synchronization)

Both endpoints support querying collections for changes relative to a synchronization state. While the functionality is the same, the methods are slightly different.

On the Microsoft Graph endpoint, changes are queried by using [delta queries](https://developer.microsoft.com/graph/docs/concepts/delta_query_overview). This is implemented as a `delta` function on the collection.

On the Outlook endpoint, changes are queried by [adding a header](/previous-versions/office/office-365-api/api/version-2.0/mail-rest-operations#synchronize-messages) to normal resource collection queries.

### Batching

Both endpoints support batching up to 20 separate requests into one HTTP request.

[Microsoft Graph batching](https://developer.microsoft.com/graph/docs/concepts/json_batching) encodes multiple API requests into a JSON body with a content type of `application/json`.

In addition to the JSON body format, [Outlook endpoint batching](/previous-versions/office/office-365-api/api/version-2.0/batch-outlook-rest-requests) also supports a multi-part body format with a content type of `multipart/mixed`.

### Example: retrieving a message

Let's take a look at a simple example. In this scenario, a web app requests a list of messages in the user's inbox.

#### Microsoft Graph

First, the app has the user sign in to authorize the application. Because the app uses the Microsoft Graph scope `Mail.Read`, the authorization URL looks like the following:

```http
https://login.microsoftonline.com/common/oauth2/v2.0/authorize?scope=openid+Mail.Read&response_type=code&client_id=<SOME GUID>&redirect_uri=<REDIRECT URL>
```

Once the app has an access token, it sends the following request:

```http
GET https://graph.microsoft.com/v1.0/me/mailfolders/inbox/messages?$top=1&$select=subject,from,receivedDateTime,isRead
Accept: application/json
Authorization: Bearer <token>
```

The server returns the following response:

```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('b63d5fb9-4f43-44c4-8f9d-fd0727842876')/mailFolders('inbox')/messages(subject,from,receivedDateTime,isRead)",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/mailfolders/inbox/messages?$top=1&$select=subject%2cfrom%2creceivedDateTime%2cisRead&$skip=1",
  "value": [
    {
      "@odata.etag": "W/\"CwAAABYAAACd9nJ/tVysQos2hTfspaWRAAD8ujHV\"",
      "id": "AAMkAGI2...",
      "receivedDateTime": "2015-11-03T03:21:04Z",
      "subject": "Scrum",
      "isRead": false,
      "from": {
        "emailAddress": {
          "name": "user0TestUser",
          "address": "user0@contoso.com"
        }
      }
    }
  ]
}
```

#### Outlook

First, the app has the user sign in to authorize the application. Because the app uses the Outlook scope `https://outlook.office.com/Mail.Read`, the authorization URL looks like the following:

```http
https://login.microsoftonline.com/common/oauth2/v2.0/authorize?scope=openid+https%3A%2F%2Foutlook.office.com%2Fmail.read&response_type=code&client_id=<SOME GUID>&redirect_uri=<REDIRECT URL>
```

Once the app has an access token, it sends the following request:

```http
GET https://outlook.office.com/api/v2.0/me/mailfolders/inbox/messages?$top=1&$select=Subject,From,ReceivedDateTime,IsRead
Accept: application/json
Authorization: Bearer <token>
```

The server returns the following response:

```json
{
  "@odata.context": "https://outlook.office.com/api/v2.0/$metadata#Me/MailFolders('inbox')/Messages(Subject,From,ReceivedDateTime,IsRead)",
  "@odata.nextLink": "https://outlook.office.com/api/v2.0/$metadata#Me/MailFolders('inbox')/Messages(Subject,From,ReceivedDateTime,IsRead)",
  "value": [
    {
      "@odata.etag": "W/\"CwAAABYAAACd9nJ/tVysQos2hTfspaWRAAD8ujHV\"",
      "Id": "AAMkAGI2...",
      "ReceivedDateTime": "2015-11-03T03:21:04Z",
      "Subject": "Scrum",
      "IsRead": false,
      "From": {
        "EmailAddress": {
          "Name": "user0TestUser",
          "Address": "user0@contoso.com"
        }
      }
    }
  ]
}
```