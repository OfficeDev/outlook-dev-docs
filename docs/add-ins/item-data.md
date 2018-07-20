---
title: Get or set item data in an Outlook Add-in | Microsoft Docs
description: Learn how to get or set item data in an Outlook Add-in.
author: jasonjoh
ms.topic: article
ms.technology: office-add-ins
ms.date: 06/13/2017
ms.author: jasonjoh
---

# Get and set Outlook item data in read or compose forms

Starting in version 1.1 of the Office Add-ins manifests schema, Outlook can activate add-ins when the user is viewing or composing an item. Depending on whether an add-in is activated in a read or compose form, the properties that are available to the add-in on the item differ as well. 

For example, the [dateTimeCreated](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) and [dateTimeModified](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) properties are defined only for an item that has already been sent (item is subsequently viewed in a read form) but not when the item is being created (in a compose form). Another example is the [bcc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) property, which is only meaningful when a message is being authored (in a compose form), and is not accessible to the user in a read form.

## Item properties available in compose and read forms

Table 1 shows the item-level properties in the JavaScript API for Office that are available in each of read and compose modes of mail add-ins. Typically, those properties available in read forms are read-only, and those available in compose forms are read/write, with the exception of the [itemId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) and [conversationId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) properties, which are always read-only regardless. 

For the remaining item-level properties available in compose forms, because the add-in and user can possibly be reading or writing the same property at the same time, the methods to get or set them in compose mode are asynchronous, and hence the type of the objects returned by these properties are also different in compose forms than in read forms. For more information about using asynchronous methods to get or set item-level properties in compose mode, see [Get and set item data in a compose form in Outlook](get-and-set-item-data-in-a-compose-form.md).


**Table 1. Item properties available in compose and read forms**

<br/>

|**Item type**|**Property**|**Property type in read forms**|**Property type in compose forms**|
|:-----|:-----|:-----|:-----|
|Appointments and messages|[dateTimeCreated](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|JavaScript **Date** object|Property not available|
|Appointments and messages|[dateTimeModified](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|JavaScript **Date** object|Property not available|
|Appointments and messages|[itemClass](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|Property not available|
|Appointments and messages|[itemId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|Property not available|
|Appointments and messages|[itemType](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String in [ItemType](https://dev.office.com/reference/add-ins/outlook/1.5/Office.MailboxEnums?product=outlook&version=v1.5) enumeration|Property not available|
|Appointments and messages|[attachments](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[AttachmentDetails](https://dev.office.com/reference/add-ins/outlook/1.5/simple-types?product=outlook&version=v1.5)|Property not available|
|Appointments and messages|[body](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[Body](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)|[Body](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)|
|Appointments|[end](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|JavaScript **Date** object|[Time](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5)|
|Appointments|[location](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|[Location](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5)|
|Appointments and messages|[normalizedSubject](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|Property not available|
|Appointments|[optionalAttendees](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|[EmailAddressDetails](https://dev.office.com/reference/add-ins/outlook/1.5/simple-types?product=outlook&version=v1.5)|[Recipients](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)|
|Appointments|[organizer](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|EmailAddressDetails|Property not available|
|Appointments|[requiredAttendees](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|EmailAddressDetails|Recipients|
|Appointments|[resources](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|Property not available|
|Appointments|[start](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|JavaScript **Date** object|Time|
|Appointments and messages|[subject](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|[Subject](https://dev.office.com/reference/add-ins/outlook/1.5/Subject?product=outlook&version=v1.5)|
|Messages|[bcc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Property not available|Recipients|
|Messages|[cc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|EmailAddressDetails|Recipients|
|Messages|[conversationId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|String|String (read only)|
|Messages|[from](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|EmailAddressDetails|Property not available|
|Messages|[internetMessageId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|Integer|Property not available|
|Messages|[sender](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|EmailAddressDetails|Property not available|
|Messages|[to](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)|EmailAddressDetails|Recipients|

## Use Exchange Server callback tokens from a read add-in

If your Outlook Add-in is activated in read forms, you can get an Exchange callback token. This token can be used in server-side code to access the full item via Exchange Web Services (EWS). 

By specifying the **ReadItem** permission in the add-in manifest, you can use the [mailbox.getCallbackTokenAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5) method to get an Exchange callback token, the [mailbox.ewsUrl](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5) property to get the URL of the EWS endpoint for the user's mailbox, and [item.itemId](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) to get the EWS ID for the selected item. You can then pass the callback token, EWS endpoint URL, and the EWS item ID to server-side code to access the [GetItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/getitem-operation) operation, to get more properties of the item.


## Access EWS from a read or compose add-in

You can also use the [mailbox.makeEwsRequestAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5) method to access the Exchange Web Services (EWS) operations [GetItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/getitem-operation) and [UpdateItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/updateitem-operation) directly from the add-in. You can use these operations to get and set many properties of a specified item. This method is available to Outlook Add-ins regardless of whether the add-in has been activated in a read or compose form, as long as you specify the **ReadWriteMailbox** permission in the add-in manifest. 

For more information about using **makeEwsRequestAsync** to access EWS operations, see [Call web services from an Outlook Add-in](web-services.md).


## See also

- [Get and set item data in a compose form in Outlook](get-and-set-item-data-in-a-compose-form.md)
- [Call web services from an Outlook Add-in](web-services.md)
    


