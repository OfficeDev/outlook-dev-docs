---
title: Get started with actionable messages
description: Get started with actionable messages. Learn about the available delivery mechanisms and applicable scenarios.
author: jasonjoh
ms.topic: conceptual
ms.service: outlook
ms.date: 05/09/2017
ms.author: jasonjoh
ms.localizationpriority: high
ms.subservice: o365-connectors
---

# Get started with actionable messages in Office 365

[!INCLUDE [global-onboarding-paused-notice](../includes/actionable-messages/global-onboarding-paused-notice.md)]

Actionable messages can be posted via a group or inbox connector, or can be sent directly over email. Choosing the right delivery mechanism depends on your scenario.

> [!NOTE]
> Office 365 administrators can disable actionable messages via the [Set-OrganizationConfig cmdlet](/powershell/module/exchange/organization/set-organizationconfig). If actionable messages do not render, check with your administrator to make sure the feature is enabled in your organization.

## Universal Actions in Outlook

[Adaptive Card](./adaptive-card.md) have evolved from the developer feedback that even though layout and rendering for Adaptive Cards was universal, action handling was not.

This is resolved with Universal Actions and above that brings the bot as the common backend for handling actions and introduces a new action type, [Action.Execute](https://adaptivecards.io/explorer/Action.Execute.html), which works across apps, such as Teams and Outlook. Universal Actions is available with adaptive card version 1.4 and higher. See Overview of [Universal Action Model](./universal-action-model.md) to learn more.

## Connectors vs Email: Choosing a delivery mechanism

With Office connectors, any user can choose to connect to services like Trello, Bing News, Twitter, etc., from Outlook and get notified of activity from that service into their Office 365 inbox or Group. With actionable messages for Office connectors, user can now act on these notifications to complete routine tasks without the hassle of context switching or signing in. For example the Trello connector allow users to subscribe to the boards and notifications they care about and lets them take actions such as set a due date or add a comment without ever leaving Outlook.

On the other hand, several line-of-business solutions send system messages to users that are business critical and requires the user to complete a task. Today, users complete these tasks by visiting a website or switching to another application. This hinders user productivity due to context switching and the extra steps required to complete that task. Examples include expense approvals, bill pay, etc. These are great candidates for enabling actions within the email itself.

The two approaches may not be necessarily mutually exclusive, as you may choose to use both solutions for enabling actionable messages from your service. For example, a CRM system may generate an approval email for authorizing a sales discount by a manager, thus enabling an actionable message via email. Consider other scenarios where users or teams want to be notified of important changes to opportunities, leads or accounts they manage. In these cases, a user may update the stage of an opportunity or lead, or set the close date depending on the notification they get. Office connectors are a great way to accomplish these scenarios.

Once you've decided which delivery method is best for your scenario, you're ready to start trying some examples.

- [Designing Actions and Inputs](./adaptive-card.md)
- [Actionable Messages via Email](send-via-email.md)
- [Actionable Messages via Connectors](send-via-connectors.md)

## Questions

For questions prior to signing up or for help, please send email to [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com).
