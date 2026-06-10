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

# Frequently asked questions for Actionable Messages

Find answers to common questions about Actionable Messages.

## Registration process

### I am unable to open the AM Developer Dashboard. How can I proceed?

If you are encountering errors when opening the [AM Developer Dashboard](https://aka.ms/ActionableMessagesPortal), follow these steps to resolve the issue.

1. Open [Outlook Web Email](https://outlook.office.com/mail/)
2. In the next tab open the [AM Developer Dashboard](https://aka.ms/ActionableMessagesPortal)

*This is a known issue and is being fixed*

### I am trying to fill in the registration form but getting an error while submitting. How can I proceed?

This might happen if some fields contain unexpected values.

1. Check all the fields again.
1. Try using a smaller payload and confirm that HTML tags are not used.
1. If you don't find any discrepancy, remove the card payload JSON and submit again. If the form submits successfully, send the card payload separately to [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com) along with your originator ID.

### I have submitted a registration request with organization scope. Who will approve my registration?

For an organization scope registration of Actionable Messages, the approval of the registration depends on the policies of your organization. Typically, the person or team responsible for managing the M365 tenant will need to approve the registration of Actionable Messages. This could be an IT administrator, a security team, or another group within the organization that is responsible for managing Office 365.

To check who your organization's IT admins are, follow these steps:

1. Go to [Graph Explorer](https://developer.microsoft.com/graph/graph-explorer).
1. Make a GET request using `https://graph.microsoft.com/v1.0/directoryRoles` request URL. This will show the list of roles in your tenant.
1. In the response you can search for **Exchange Administrator** and **Global Administrator**.
1. In some tenants **Exchange Administrator** might not be present. If the **Exchange Administrator** role is not listed then you can use the `id` from the **Global Administrator** role to fetch the list of tenant admins.

### How can I update details for an approved globally scoped Actionable Message registration?

To update details for an approved globally scoped Actionable Message provider, reach out to [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com) with the updated details. Our team will acknowledge and update it in the backend. Note that any update will take 2 weeks after the team acknowledges your request.

## Actionable Message Rendering

### Actionable card does not show up in Outlook desktop but works fine in Outlook on the web.

If you don't see the Actionable Message in Outlook desktop, please confirm the following:

1. Your download preferences are set to "Download full items". It should look like the following image.

    :::image type="content" source="images/download-preference.png" alt-text="Download preferences to see actionable messages":::

1. You are not using any screen reader.
1. Also check if the following registry key is set to 0 - `HKEY_CURRENT_USER\Control Panel\Accessibility\Blind Access\On`.

### Actionable messages work as expected with user mailboxes but not with Group or shared mailboxes.

This is the expected behavior. Actionable Messages are only supported with single user mailboxes. Group and shared mailboxes are not supported.

### Users from my organization are not able to use Actionable Messages and the action redirects them to a webpage.

Please check if the organization uses Mimecast or other similar services. Mimecast changes emails in ways that prevent Actionable Messages workflow.

To verify that this is the cause:

1. Disable Mimecast temporarily and send a new Actionable Message.
1. Add "schema.org" domain or `http://schema.org/extensions` to Mimecast exception list.

### Is there a limit to the number of Actionable Messages that can be opened in Outlook at one time?

To maintain optimal performance, we allow a maximum of 10 actionable message emails to be opened at one time. Trying to open more than that at the same time will show an error.

### Is there a time limit on taking actions on Actionable Messages?

Actionable Messages are typically designed for quick actions on emails. As a result, you cannot take actions on an Actionable Message that is more than a month old.

## Upgrading to Adaptive Card 1.4 and above

### I have upgraded the Adaptive Card version from 1.0 to 1.4, and my Action buttons have disappeared.

The action execution paradigm has changed in Adaptive Card version 1.4. We have started supporting [Action.Execute](https://adaptivecards.io/explorer/Action.Execute.html) in place of `Action.Http`. For more information, see [code sample using Adaptive cards 1.4+](./adaptive-card-expense-approval-sample.md).