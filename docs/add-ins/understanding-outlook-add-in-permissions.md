---
title: Understanding Outlook Add-in permissions | Microsoft Docs
description: Learn about the permissions model for Outlook Add-ins.
author: jasonjoh
ms.topic: article
ms.technology: office-add-ins
ms.date: 06/13/2017
ms.author: jasonjoh
---

# Understanding Outlook Add-in permissions

Outlook Add-ins specify the required permission level in their manifest. The available levels are **Restricted**, **ReadItem**, **ReadWriteItem**, or **ReadWriteMailbox**. These levels of permissions are cumulative: **Restricted** is the lowest level, and each higher level includes the permissions of all the lower levels. **ReadWriteMailbox** includes all the supported permissions.

You can see the permissions requested by a mail add-in before installing it from the Office Store. You can also see the required permissions of installed add-ins in the Exchange Admin Center.

## Restricted permission

The **Restricted** permission is the most basic level of permission. Specify **Restricted** in the [Permissions](https://dev.office.com/reference/add-ins/manifest/permissions?product=outlook&version=v1.5) element in the manifest to request this permission. Outlook assigns this permission to a mail add-in by default if the add-in does not request a specific permission in its manifest.

### Can do

- [Get only specific entities](match-strings-in-an-item-as-well-known-entities.md) (phone number, address, URL) from the item's subject or body.
- Specify an [ItemIs activation rule](activation-rules.md#itemis-rule) that requires the current item in a read or compose form to be a specific item type, or [ItemHasKnownEntity rule](match-strings-in-an-item-as-well-known-entities.md) that matches any of a smaller subset of supported well-known entities (phone number, address, URL) in the selected item.
- Access any properties and methods that do **not** pertain to specific information about the user or item. (See the next section for the list of members that do.)

### Can't do

- Use an [ItemHasKnownEntity](https://dev.office.com/reference/add-ins/manifest/rule?product=outlook&version=v1.5#itemhasknownentity-rule) rule on the contact, email address, meeting suggestion, or task suggestion entitiy.
- Use the [ItemHasAttachment](https://dev.office.com/reference/add-ins/manifest/rule?product=outlook&version=v1.5#itemhasattachment-rule) or [ItemHasRegularExpressionMatch](https://dev.office.com/reference/add-ins/manifest/rule?product=outlook&version=v1.5#itemhasregularexpressionmatch-rule) rule.
- Access the members in the following list that pertain to the information of the user or item. Attempting to access members in this list will return **null** and result in an error message which states that Outlook requires the mail add-in to have elevated permission.

- [item.addFileAttachmentAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.addItemAttachmentAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.attachments](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.bcc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.body](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.cc](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.from](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.getRegExMatches](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.getRegExMatchesByName](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.optionalAttendees](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.organizer](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.removeAttachmentAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.requiredAttendees](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.resources](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.sender](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [item.to](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
- [mailbox.getCallbackTokenAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5)
- [mailbox.getUserIdentityTokenAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5)
- [mailbox.makeEwsRequestAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5)
- [mailbox.userProfile](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.userProfile?product=outlook&version=v1.5)
- [Body](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5) and all its child members
- [Location](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5) and all its child members
- [Recipients](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5) and all its child members
- [Subject](https://dev.office.com/reference/add-ins/outlook/1.5/Subject?product=outlook&version=v1.5) and all its child members
- [Time](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5) and all its child members

## ReadItem permission

The **ReadItem** permission is the next level of permission in the permissions model. Specify **ReadItem** in the **Permissions** element in the manifest to request this permission.

### Can do

- [Read all the properties](item-data.md) of the current item in a read or [compose form](get-and-set-item-data-in-a-compose-form.md), for example, [item.to](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) in a read form and [item.to.getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5) in a compose form.
- [Get a callback token to get item attachments](get-attachments-of-an-outlook-item.md) or the full item with Exchange Web Services (EWS) or [Outlook REST APIs](use-rest-api.md).
- [Write custom properties](https://dev.office.com/reference/add-ins/outlook/1.5/CustomProperties?product=outlook&version=v1.5) set by the add-in on that item.
- [Get all existing well-known entities](match-strings-in-an-item-as-well-known-entities.md), not just a subset, from the item's subject or body.
- Use all the [well-known entities](activation-rules.md#itemhasknownentity-rule) in [ItemHasKnownEntity](https://dev.office.com/reference/add-ins/manifest/rule?product=outlook&version=v1.5#itemhasknownentity-rule) rules, or [regular expressions](activation-rules.md#itemhasregularexpressionmatch-rule) in [ItemHasRegularExpressionMatch](https://dev.office.com/reference/add-ins/manifest/rule?product=outlook&version=v1.5#itemhasregularexpressionmatch-rule) rules. The following example follows schema v1.1. It shows a rule that activates the add-in if one or more of the well-known entities are found in the subject or body of the selected message:

```XML
<Permissions>ReadItem</Permissions>
    <Rule xsi:type="RuleCollection" Mode="And">
    <Rule xsi:type="ItemIs" FormType = "Read" ItemType="Message" />
    <Rule xsi:type="RuleCollection" Mode="Or">
        <Rule xsi:type="ItemHasKnownEntity" 
            EntityType="PhoneNumber" />
        <Rule xsi:type="ItemHasKnownEntity" EntityType="Address" />
        <Rule xsi:type="ItemHasKnownEntity" EntityType="Url" />
        <Rule xsi:type="ItemHasKnownEntity" 
            EntityType="MeetingSuggestion" />
        <Rule xsi:type="ItemHasKnownEntity" 
            EntityType="TaskSuggestion" />
        <Rule xsi:type="ItemHasKnownEntity" 
            EntityType="EmailAddress" />
        <Rule xsi:type="ItemHasKnownEntity" EntityType="Contact" />
</Rule>
```

### Can't do

- Use the token provided by **mailbox.getCallbackTokenAsync** to update or delete the current item using the Outlook REST API or access any other items in the user's mailbox.
- Use any of the following APIs:
    - [mailbox.makeEwsRequestAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox?product=outlook&version=v1.5)
    - [item.addFileAttachmentAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
    - [item.addItemAttachmentAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
    - [item.bcc.addAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.bcc.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.body.prependAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)
    - [item.body.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)
    - [item.body.setSelectedDataAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Body?product=outlook&version=v1.5)
    - [item.cc.addAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.cc.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.end.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5)
    - [item.location.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5)
    - [item.optionalAttendees.addAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.optionalAttendees.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.removeAttachmentAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5)
    - [item.requiredAttendees.addAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.requiredAttendees.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.start.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Time?product=outlook&version=v1.5)
    - [item.subject.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Subject?product=outlook&version=v1.5)
    - [item.to.addAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)
    - [item.to.setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Recipients?product=outlook&version=v1.5)

## ReadWriteItem permission

Specify **ReadWriteItem** in the **Permissions** element in the manifest to request this permission. Mail add-ins activated in compose forms that use write methods ( **Message.to.addAsync** or **Message.to.setAsync**) must use at least this level of permission.

### Can do

- [Read and write all item-level properties](item-data.md) of the item that is being viewed or composed in Outlook.
- [Add or remove attachments](add-and-remove-attachments-to-an-item-in-a-compose-form.md) of that item.
- Use all other members of the JavaScript API for Office that are applicable to mail add-ins, except **Mailbox.makeEWSRequestAsync**.

### Can't do

- Use the token provided by **mailbox.getCallbackTokenAsync** to update or delete the current item using the Outlook REST API or access any other items in the user's mailbox.
- Use **mailbox.makeEWSRequestAsync**.

## ReadWriteMailbox permission

The **ReadWriteMailbox** permission is the highest level of permission. Specify **ReadWriteMailbox** in the **Permissions** element in the manifest to request this permission.

In addition to what the **ReadWriteItem** permission supports, the token provided by **mailbox.getCallbackTokenAsync** provides access to use Exchange Web Services (EWS) operations or Outlook REST APIs to do the following:

- Read and write all properties of any item in the user's mailbox.
- Create, read, and write to any folder or item in that mailbox.
- Send an item from that mailbox

Through **mailbox.makeEWSRequestAsync**, you can access the following EWS operations:

- [CopyItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/copyitem-operation)
- [CreateFolder](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/createfolder-operation)
- [CreateItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/createitem-operation)
- [FindConversation](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/findconversation-operation)
- [FindFolder](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/findfolder-operation)
- [FindItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/finditem-operation)
- [GetConversationItems](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/getconversationitems-operation)
- [GetFolder](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/getfolder-operation)
- [GetItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/getitem-operation)
- [MarkAsJunk](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/markasjunk-operation)
- [MoveItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/moveitem-operation)
- [SendItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/senditem-operation)
- [UpdateFolder](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/updatefolder-operation)
- [UpdateItem](https://docs.microsoft.com/en-us/exchange/client-developer/web-service-reference/updateitem-operation)

Attempting to use an unsupported operation will result in an error response.

## See also

- [Privacy, permissions, and security for Outlook Add-ins](https://docs.microsoft.com/office/dev/add-ins/develop/privacy-and-security)
- [Match strings in an Outlook item as well-known entities](match-strings-in-an-item-as-well-known-entities.md)