---
title: Get and set item data in a compose form in Outlook | Microsoft Docs
description: Learn what data is available in an Outlook compose form add-in.
author: jasonjoh
ms.topic: article
ms.technology: office-add-ins
ms.date: 08/09/2017
ms.author: jasonjoh
---

# Get and set item data in a compose form in Outlook

Learn how to get or set various properties of an item in an Outlook Add-in in a compose scenario, including its recipients, subject, body, and appointment location and time.

## Getting and setting item properties for a compose add-in

In a compose form, you can get most of the properties that are exposed on the same kind of item as in a read form (such as attendees, recipients, subject, and body), and you can get a few extra properties that are relevant in only a compose form but not a read form (body, bcc). 

For most of these properties, because it's possible that an Outlook Add-in and the user can be modifying the same property in the user interface at the same time, the methods to get and set them are asynchronous. Table 1 lists the item-level properties and corresponding asynchronous methods to get and set them in a compose form. The  [item.itemType](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) and [item.conversationId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) properties are exceptions because users cannot modify them. You can programmatically get them the same way in a compose form as in a read form, directly from the parent object.

Other than accessing item properties in the JavaScript API for Office, you can access item-level properties using Exchange Web Services (EWS). With the **ReadWriteMailbox** permission, you can use the [mailbox.makeEwsRequestAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5) method to access EWS operations, [GetItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/getitem-operation) and [UpdateItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/updateitem-operation), to get and set more properties of an item or items in the user's mailbox. 

The `makeEwsRequestAsync` function is available in both compose and read forms. For more information about the **ReadWriteMailbox** permission, and accessing EWS through the Office Add-ins platform, see [Understanding Outlook Add-in permissions](understanding-outlook-add-in-permissions.md) and [Call web services from an Outlook Add-in](web-services.md).

**Table 1. Asynchronous methods to get or set item properties in a compose form**

<br/>

| Property | Property type | Asynchronous method to get | Asynchronous method(s) to set |
|:-----|:-----|:-----|:-----|
|[bcc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[Recipients](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)|[Recipients.getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)|[Recipients.addAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5), [Recipients.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)|
|[body](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[Body](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)|[Body.getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)|[Body.prependAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5), [Body.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5), [Body.setSelectedDataAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)|
|[cc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Recipients|Recipients.getAsync|Recipients.addAsync Recipients.setAsync|
|[end](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[Time](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5)|[Time.getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5)|[Time.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5)|
|[location](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[Location](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5)|[Location.getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5)|[Location.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5)|
|[optionalAttendees](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Recipients|Recipients.getAsync|Recipients.addAsync Recipients.setAsync|
|[requiredAttendees](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Recipients|Recipients.getAsync|Recipients.addAsync Recipients.setAsync|
|[start](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Time|Time.getAsync|Time.setAsync|
|[subject](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[Subject](https://dev.office.com/reference/add-ins/outlook/1.5/Subject?product=outlook&version=v1.5)|[Subject.getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Subject?product=outlook&version=v1.5)|[Subject.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Subject?product=outlook&version=v1.5)|
|[to](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Recipients|Recipients.getAsync|Recipients.addAsync Recipients.setAsync|

## See also

- [Create Outlook Add-ins for compose forms](compose-scenario.md)
- [Understanding Outlook Add-in permissions](understanding-outlook-add-in-permissions.md)
- [Call web services from an Outlook Add-in](web-services.md)
- [Get and set Outlook item data in read or compose forms](item-data.md)