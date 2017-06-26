---
title: Understanding Outlook add-in permissions | Microsoft Docs
description: Learn about the permissions model for Outlook add-ins.
author: jasonjoh

ms.topic: article
ms.technology: office-add-ins
ms.date: 06/13/2017
ms.author: jasonjoh
---

# Understanding Outlook add-in permissions

Outlook add-ins specify the required permission level in their manifest. The available levels are **Restricted**, **ReadItem**, **ReadWriteItem**, or **ReadWriteMailbox**. These levels of permissions are cumulative: **Restricted** is the lowest level, and each higher level includes the permissions of all the lower levels. **ReadWriteMailbox** includes all the supported permissions.

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

- [CopyItem](http://msdn.microsoft.com/en-us/library/bcc68f9e-d511-4c29-bba6-ed535524624a%28Office.15%29.aspx)
- [CreateFolder](http://msdn.microsoft.com/en-us/library/6f6c334c-b190-4e55-8f0a-38f2a018d1b3%28Office.15%29.aspx)
- [CreateItem](http://msdn.microsoft.com/en-us/library/78a52120-f1d0-4ed7-8748-436e554f75b6%28Office.15%29.aspx)
- [FindConversation](http://msdn.microsoft.com/en-us/library/2384908a-c203-45b6-98aa-efd6a4c23aac%28Office.15%29.aspx)
- [FindFolder](http://msdn.microsoft.com/en-us/library/7a9855aa-06cc-45ba-ad2a-645c15b7d031%28Office.15%29.aspx)
- [FindItem](http://msdn.microsoft.com/en-us/library/ebad6aae-16e7-44de-ae63-a95b24539729%28Office.15%29.aspx)
- [GetConversationItems](http://msdn.microsoft.com/en-us/library/8ae00a99-b37b-4194-829c-fe300db6ab99%28Office.15%29.aspx)
- [GetFolder](http://msdn.microsoft.com/en-us/library/355bcf93-dc71-4493-b177-622afac5fdb9%28Office.15%29.aspx)
- [GetItem](http://msdn.microsoft.com/en-us/library/e3590b8b-c2a7-4dad-a014-6360197b68e4%28Office.15%29.aspx)
- [MarkAsJunk](http://msdn.microsoft.com/en-us/library/1f71f04d-56a9-4fee-a4e7-d1034438329e%28Office.15%29.aspx)
- [MoveItem](http://msdn.microsoft.com/en-us/library/dcf40fa7-7796-4a5c-bf5b-7a509a18d208%28Office.15%29.aspx)
- [SendItem](http://msdn.microsoft.com/en-us/library/337b89ef-e1b7-45ed-92f3-8abe4200e4c7%28Office.15%29.aspx)
- [UpdateFolder](http://msdn.microsoft.com/en-us/library/3494c996-b834-4813-b1ca-d99642d8b4e7%28Office.15%29.aspx)
- [UpdateItem](http://msdn.microsoft.com/en-us/library/5d027523-e0bc-4da2-b60b-0cb9fc1fdfe4%28Office.15%29.aspx)

Attempting to use an unsupported operation will result in an error response.

## Additional resources

- [Privacy, permissions, and security for Outlook add-ins](https://dev.office.com/docs/add-ins/develop/privacy-and-security?product=outlook)
- [Match strings in an Outlook item as well-known entities](match-strings-in-an-item-as-well-known-entities.md)