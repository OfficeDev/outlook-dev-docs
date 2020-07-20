---
title: Outlook REST API Overview
description: With the simplicity of REST, you can use your favorite language and IDE and write your app once to capture millions of users.
author: jasonjoh
ms.topic: overview
ms.technology: ms-graph
ms.date: 02/19/2020
ms.author: jasonjoh
localization_priority: Priority
---

# One Outlook REST API - your favorite platform - 400+ million users

With the simplicity of REST, you can use your favorite language and IDE, write your app once, and capture 400 million monthly active Outlook.com users, and tens of millions active Office 365 users.

Start with choosing a language for your app&mdash;Node, Python, Ruby, Swift&mdash;just to name a few. Write the code, take advantage of new, streamlined services to register and authorize the app, and access user's mail, calendar, and contacts data on Outlook.com or Office 365. You can use the same Outlook REST API for Android, iOS, Windows, on the web, mobile, and desktop. There's no need for any specialized Exchange knowledge!

## Get started

1. [Register](https://aad.portal.azure.com) your app. Registration takes only a minute.

2. [Start](get-started.md) coding and implement REST API calls.

## Streamlined services

There's just one simple process for Outlook.com and Office 365: register and get dynamic user authorization to access users’ mail, calendar, and contacts.

Take an early look at the following services in preview status:

- [Azure Active Directory admin center](https://aad.portal.azure.com)

  Use a Microsoft account or Microsoft 365 subscription account to register your app. It takes only a few steps to identify your app for Outlook.com and Office 365.

- v2 endpoints:

  [Use these common authentication endpoints](/azure/active-directory/develop/v2-overview) to sign users in with their personal Outlook.com accounts, or business or school credentials, and request authorization for access. Instantly expand the reach of your app!

  - `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`
  - `https://login.microsoftonline.com/common/oauth2/v2.0/token`

<!-- markdownlint-disable MD026 -->
### Use these preview services when:
<!-- markdownlint-enable MD026 -->

- Rewriting existing Outlook.com apps that use the Windows Live API. Windows Live API has been deprecated for Outlook.com. Plan to accommodate changes to these services over their preview period.
- Creating new Outlook.com and Office 365 apps that access mailbox data.

Plan to upgrade in-production Office 365 apps after the preview period is over.

## Outlook REST API via Microsoft Graph

Use one common REST endpoint...

```http
https://graph.microsoft.com/{version}
```

...to make all Outlook REST API calls in the following APIs:

- [Outlook Mail REST API](/graph/api/resources/message?view=graph-rest-1.0)
- [Outlook Calendar REST API](/graph/api/resources/calendar?view=graph-rest-1.0)
- [Outlook Contacts REST API](/graph/api/resources/contact?view=graph-rest-1.0)
- [Outlook Notifications REST API](/graph/api/resources/webhooks?view=graph-rest-1.0)
- [Outlook Photo REST API](/graph/api/resources/profilephoto?view=graph-rest-1.0)
- [Outlook Settings REST API](/graph/api/resources/outlookuser?view=graph-rest-1.0)

For more information and a comparison between the Graph endpoints and the Outlook API endpoints, see [Compare Microsoft Graph and Outlook endpoints](compare-graph.md).
