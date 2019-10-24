---
title: Get and set internet headers
description: How to get and set internet headers on a message in an Outlook add-in.
ms.topic: article
ms.date: 10/24/2019
localization_priority: Normal
---

# Get and set internet headers (preview)

## Background

It has been a common requirement in Outlook add-ins development to store custom properties associated with an add-in at different levels. At present, you can store custom properties at the item or mailbox level.

- Item level - For properties that apply to a specific item, you can use the [CustomProperties](/javascript/api/outlook/office.customproperties) object. For example, store a customer code associated with the person who sent the email.
- Mailbox level - For properties that apply to the all mail items in the user's mailbox, you can use the [RoamingSettings](/javascript/api/outlook/office.roamingsettings) object. For example, store a user's preference to show the temperature in a particular scale.

Both types of properties are not preserved after the item leaves the Exchange server so the email recipients can't get any properties set on the item. Therefore, developers can't access those settings or other MIME properties to enable better read scenarios.

While there's a way for you to set the internet headers through EWS Requests, in some scenarios, making an EWS request won't work. For example, in Compose mode on Outlook desktop, the item id isn't synced on `saveAsync` in cached mode.

> [!TIP]
> See [Get and set add-in metadata for an Outlook add-in](metadata-for-an-outlook-add-in.md) to learn more about using these options.

## Purpose of the internet headers API

The internet headers APIs enable developers to:

- Stamp information on an email that persists after it leaves Exchange across all clients.
- Read information on an email that persisted after the email left Exchange across all clients in mail read scenarios.
- Access the entire MIME header of the email.

> [!IMPORTANT]
> Internet headers APIs for Outlook are currently in [preview](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview#internet-headers) for Outlook on Windows connected to an Office 365 subscription, and are not yet intended for use in production environments.
>
> [!INCLUDE [Information about using preview APIs](../includes/using-preview-apis.md)]

## Example  

TODO: Set internet headers in compose. Retrieve in read.

### Set internet headers while composing a message

### Get internet headers while reading a message

## See also

- [Get and set add-in metadata for an Outlook add-in](metadata-for-an-outlook-add-in.md)