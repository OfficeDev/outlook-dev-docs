---
title: Outlook API Reference | Microsoft Docs
description: Get a list of the Outlook API references.
author: jasonjoh

ms.topic: reference
ms.technology: ms-graph
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Outlook API Reference

The Outlook REST APIs are a part of the [Microsoft Graph](https://developer.microsoft.com/en-us/graph/). Microsoft recommends using the Microsoft Graph to access Outlook mail, calendar, and contacts. You should use the Outlook API endpoints directly (via `https://outlook.office.com/api`) only if you require a feature that is not available on the Graph endpoints. For more information on the differences between Graph and the Outlook endpoints, see [Compare the Microsoft Graph and Outlook endpoints](compare-graph-outlook.md).

## Released APIs

The following APIs are released and ready for production use in both the Graph and Outlook endpoints.

- [Mail API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/message)
- [Calendar API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/calendar)
- [Contacts API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/contact)
- [Groups API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/group)
- [Push Notifications API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/webhooks)
- [User Photo API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/profilephoto)
- [Open Extensions API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/opentypeextension)
- [Extended Properties API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/extended-properties-overview)
- [FindMeetingTimes API](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_findmeetingtimes)

## Transitioning APIs

The following APIs are released and ready for production in the Outlook endpoint, but are in preview status in the Graph endpoint.

- [Sync messages, mail folders, events, contacts, and contact folders API](https://developer.microsoft.com/en-us/graph/docs/concepts/delta_query_overview)

## Beta APIs

The following APIs are in beta and should not be used in production.

- [People API](https://developer.microsoft.com/en-us/graph/docs/api-reference/beta/resources/person)

## APIs only available in Outlook endpoint

The following APIs are only available on the Outlook API endpoint.

### Released

- [Tasks API](https://msdn.microsoft.com/en-us/office/office365/api/task-rest-operations)

### Beta

- [Batch REST Requests](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests)