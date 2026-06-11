---
title: Frequently asked questions for Actionable Messages
description: Frequently asked questions for Actionable Messages
author: avijityadav
ms.topic: reference
ms.service: outlook
ms.subservice: o365-connectors
ms.date: 06/10/2026
ms.author: vermaanimesh
ms.localizationpriority: high
---

<!-- markdownlint-disable MD026 -->
<!-- cSpell:ignore avijityadav vermaanimesh Mimecast -->

# Frequently asked questions for Actionable Messages

Find answers to common questions about Actionable Messages.

## Registration process

### I can't open the AM Developer Dashboard. How can I proceed?

If you encounter errors when opening the [AM Developer Dashboard](https://aka.ms/ActionableMessagesPortal), follow these steps to resolve the issue.

1. Open [Outlook Web Email](https://outlook.office.com/mail/).
1. In the next tab, open the [AM Developer Dashboard](https://aka.ms/ActionableMessagesPortal).

*This is a known issue and is being fixed.*

### I try to fill in the registration form but get an error while submitting. How can I proceed?

This error might happen if some fields contain unexpected values.

1. Check all the fields again.
1. Try using a smaller payload and confirm that HTML tags aren't used.
1. If you don't find any discrepancy, remove the card payload JSON and submit again. If the form submits successfully, send the card payload separately to [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com) along with your originator ID.

### I submitted a registration request with organization scope. Who approves my registration?

For an organization scope registration of Actionable Messages, the approval process depends on your organization's policies. Typically, the person or team responsible for managing the M365 tenant needs to approve the registration of Actionable Messages. This role could be an IT administrator, a security team, or another group within the organization that's responsible for managing Office 365.

To check who your organization's IT admins are, follow these steps:

1. Go to [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer).
1. Make a GET request with `https://graph.microsoft.com/v1.0/directoryRoles` as the request URL. This request returns the list of roles in your tenant.
1. In the response, search for **Exchange Administrator** and **Global Administrator**.
1. In some tenants, **Exchange Administrator** might not be present. If the **Exchange Administrator** role isn't listed, use the `id` from the **Global Administrator** role to fetch the list of tenant admins.

### How can I update details for an approved globally scoped Actionable Message registration?

To update details for an approved globally scoped Actionable Message provider, contact [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com) with the updated details. The team acknowledges your request and updates it in the backend. Any update takes two weeks after the team acknowledges your request.

## Actionable Message Rendering

### Actionable card doesn't show up in Outlook desktop but works fine in Outlook on the web.

If you don't see the Actionable Message in Outlook desktop, confirm the following conditions:

1. Your download preferences are set to **Download full items**. It should look like the following image.

    :::image type="content" source="images/download-preference.png" alt-text="Download preferences to see actionable messages":::

1. You're not using any screen reader.
1. The following registry key is set to 0 - `HKEY_CURRENT_USER\Control Panel\Accessibility\Blind Access\On`.

### Actionable messages work as expected with user mailboxes but not with group or shared mailboxes

This behavior is expected. Actionable Messages only support single user mailboxes. Group and shared mailboxes aren't supported.

### Users from my organization can't use Actionable Messages and the action redirects them to a webpage

Check if your organization uses Mimecast or other similar services. Mimecast changes emails in ways that prevent Actionable Messages workflow.

To verify that this is the cause:

1. Temporarily disable Mimecast and send a new Actionable Message.
1. Add the "schema.org" domain or `http://schema.org/extensions` to the Mimecast exception list.

### Is there a limit to the number of Actionable Messages that can be opened in Outlook at one time?

To maintain optimal performance, you can open up to 10 actionable message emails at one time. If you try to open more than that number at the same time, an error appears.

### Is there a time limit on taking actions on Actionable Messages?

Actionable Messages are typically designed for quick actions on emails. As a result, you can't take actions on an Actionable Message that is more than a month old.

## Upgrading to Adaptive Card 1.4 and later

### I upgraded the Adaptive Card version from 1.0 to 1.4, and my action buttons disappeared

The action execution paradigm changed in Adaptive Card version 1.4. The platform now supports [Action.Execute](https://adaptivecards.io/explorer/Action.Execute.html) in place of `Action.Http`. For more information, see [code sample using Adaptive cards 1.4+](./adaptive-card-expense-approval-sample.md).
