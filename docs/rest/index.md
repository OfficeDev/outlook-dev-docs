---
title: Outlook REST API Overview | Microsoft Docs
description: Learn about the benefits of the Outlook APIs in the Microsoft Graph.
author: jasonjoh

ms.topic: overview
ms.technology: ms-graph
ms.date: 04/26/2017
ms.author: jasonjoh
---

# One Outlook REST API. Your favorite platform. 400+ million users.

With the simplicity of REST, you can use your favorite language and IDE, write your app once, and capture 400 million monthly active Outlook.com users, and tens of millions active Office 365 users.

Start with choosing a language for your app - Node, Python, Ruby, Swift - just to name a few. Write the code, take advantage of new, streamlined services to register and authorize the app, and access user's mail, calendar, and contacts data on Outlook.com or Office 365. You can use the same Outlook REST API for Android, iOS, Windows, on the web, mobile, and desktop. There's no need for any specialized Exchange knowledge!

### It's this straightforward:

1. [Register](https://apps.dev.microsoft.com/) your app. Registration takes only a minute.
1. [Start](get-started.md) coding and implement REST API calls.

### Streamlined services

There's just one simple process for Outlook.com and Office 365 - register and get dynamic user authorization to access users’ mail, calendar and contacts.

Take an early look at the following services in preview status:

- [Application Registration Portal](https://apps.dev.microsoft.com/)

    Use a Microsoft account or Office 365 subscription account to register your app. It takes only a few steps to identify your app for Outlook.com and Office 365. 

- v2 endpoints:

    [Use these common authentication endpoints](/azure/active-directory/develop/active-directory-appmodel-v2-overview) to sign users in with their personal Outlook.com accounts, or business or school credentials, and request authorization for access. Instantly expand the reach of your app!

    - `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`
    - `https://login.microsoftonline.com/common/oauth2/v2.0/token`

#### Use these preview services when:

- Rewriting existing Outlook.com apps* that use the Windows Live API. Windows Live API has been deprecated for Outlook.com. Plan to accommodate changes to these services over their preview period.
- Creating new Outlook.com* and Office 365 apps that access mailbox data.

Plan to upgrade in-production Office 365 apps once the preview period is over.

### Outlook REST API via Microsoft Graph

Use one common REST endpoint -

```http
https://graph.microsoft.com/{version} 
```

to make all Outlook REST API calls in the following APIs:

- [Outlook Mail REST API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/message)
- [Outlook Calendar REST API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/calendar)
- [Outlook Contacts REST API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/contact)
- [Outlook Notifications REST API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/webhooks)
- [Outlook Photo REST API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/profilephoto)

For more information and a comparison between the Graph endpoints and the Outlook API endpoints, see [Compare the Microsoft Graph and Outlook endpoints](compare-graph-outlook.md).
