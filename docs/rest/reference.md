---
title: Outlook API reference
description: The Outlook REST APIs are a part of Microsoft Graph. Microsoft recommends using Microsoft Graph to access Outlook mail, calendar, and contacts.
author: jasonjoh
ms.topic: reference
ms.technology: ms-graph
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Outlook API reference documentation

The Outlook REST APIs are a part of [Microsoft Graph](https://developer.microsoft.com/graph/). Microsoft recommends using Microsoft Graph to access Outlook mail, calendar, and contacts. You should use the Outlook API endpoints directly (via `https://outlook.office.com/api`) only if you require a feature that is not available on the Graph endpoints.

For more information about the differences between Graph and the Outlook endpoints, see [Compare Microsoft Graph and Outlook endpoints](compare-graph.md).

## Released APIs

The following APIs are released and ready for production use in both the Graph and Outlook endpoints.

- [Mail API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/message)
- [Calendar API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/calendar)
- [Contacts API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/contact)
- [Groups API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/group)
- [Push Notifications API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/webhooks)
- [User Photo API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/profilephoto)
- [Open Extensions API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/opentypeextension)
- [Extended Properties API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/extended-properties-overview)
- [FindMeetingTimes API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_findmeetingtimes)

<!--for the FindMeetingTimes API, you must include "en-us" in the URL or it will 404-->

## Transitioning APIs

The following APIs are released and ready for production in the Outlook endpoint, but are in preview status in the Graph endpoint.

- [Sync messages, mail folders, events, contacts, and contact folders API](https://developer.microsoft.com/graph/docs/concepts/delta_query_overview)

## Beta APIs

The following APIs are in beta and should not be used in production.

- [People API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/person)

## APIs only available in Outlook endpoint

The following APIs are only available on the Outlook API endpoint.

### Released

- [Tasks API](https://docs.microsoft.com/previous-versions/office/office-365-api/api/version-2.0/task-rest-operations)

### Beta

- [Batch REST Requests](https://docs.microsoft.com/previous-versions/office/office-365-api/api/version-2.0/batch-outlook-rest-requests)
