---
title: What are actionable messages in Office 365?
description: Actionable Messages enable you to take quick actions right from within Outlook.
author: jasonjoh
ms.topic: article
ms.service: outlook
ms.author: jasonjoh
ms.date: 11/17/2025
nodes_to_expand: get-started
ms.localizationpriority: high
ms.subservice: o365-connectors
---

# Actionable messages in Outlook and Office 365 Groups

> [!IMPORTANT]
> Actionable Messages (AM) are moving from EAT (External Access Token) to Microsoft Entra ID token authentication. Partners using EAT tokens must update their integration to support AAD tokens for requests from the AM service. For more information, see [Enabling AAD token of Actionable Messages](enable-entra-token-for-actionable-messages.md).

Whether you are filling out a survey, approving an expense report, or updating a CRM sales opportunity, Actionable Messages enable you to take quick actions right from within Outlook. Developers can now embed actions in their emails or notifications, elevating user engagement with their services and increasing organizational productivity.

## User experience

Let's take a look at the end-to-end user experience.

### Group membership request approval scenario

A Contoso employee submits a request to join a private Office 365 group. Office 365 sends an Actionable Message to the person who owns the group to approve or decline the request. The card included in the message contains all the information the approver might need to quickly understand who submitted the request and any message they included to explain their request. It also includes **Approve** and **Decline** actions that can be taken right from Outlook. The owner approves the request, and the card updates to indicate the outcome.

![A join group request message card rendered in Outlook on iOS.](images/group-join-request-ios.png)

The new member of the group submits a second request to add her team members to the group. Office 365 sends an Actionable Message to the owner with clear information about who submitted the request and the new members to add. The recipient can approve all, some, or none of the proposed new members. The owner approves one new member, and the card updates to indicate the outcome. The approved member is no longer selectable, while the remaining member remains selectable.

![An add group members message card showing the first requested member being approved as rendered in Outlook on iOS.](images/group-add-members-request-ios-1.png)

The owner declines the other requested new member, and the card updates to indicate the outcome. Both members are no longer selectable, and the action buttons are removed.

![An add group members message card showing the second requested member being declined as rendered in Outlook on iOS.](images/group-add-members-request-ios-2.png)

## Outlook version requirements for actionable messages

Actionable messages are available to all customer mailboxes on Exchange Online in Office 365 or Outlook.com with a supported client. The following table lists the availability of actionable messages for current Outlook clients. For information on the Office 365 release channels, see [Overview of update channels for Microsoft 365 apps](/deployoffice/updates/overview-update-channels). For current supported versions of Microsoft 365 Apps, see [supported versions](/officeupdates/update-history-microsoft365-apps-by-date#supported-versions).

> [!NOTE]
> Currently actionable message cards do not change the way that they render when Outlook is in dark mode. Support for dark mode for actionable messages is coming soon. Actionable messages are not supported when you use Outlook on the web in a browser on a mobile device.

| Client                                                     | Actionable messages supported? | Adaptive card supported? |
|------------------------------------------------------------|--------------------------------|--------------------------|
| Outlook on the web for Microsoft 365                       | Yes                            | Yes |
| Microsoft 365 Apps Current Channel                         | Yes                            | Yes |
| Microsoft 365 Apps Monthly Enterprise Channel              | Yes                            | Yes |
| Microsoft 365 Apps Semi-Annual Enterprise Channel          | Yes                            | Yes |
| Outlook on Mac                                             | Yes'*'                         | Yes'*' (Legacy MessageCard format is not supported) |
| Outlook on iOS                                             | Yes'*'                         | Yes'*' (Legacy MessageCard format is not supported) |
| Outlook on Android                                         | Yes'*'                         | Yes'*' (Legacy MessageCard format is not supported) |
| Office Professional Plus (Click-to-Run only), all versions | No                             | No |
| Exchange On-Premises Outlook on the web, all versions      | No                             | No |

> '*' For Developers: We support Actionable Messages version 1.4 and above, and Adaptive Cards version 1.0 and below.

> [!NOTE]
> Actionable messages may not render correctly in Outlook for Windows if you have any of the following options enabled in **Download Preferences** on the **Send / Receive** tab.
>
> - Download Headers and then Full Items
> - Download Headers
> - On Slow Connections Download Only Headers
>
> To resolve the problem, select **Download Full Items**. Depending on your connection type, you may want to uncheck **On Slow Connections Download Only Headers**.

## Submit feedback

There are multiple ways you can send us feedback.

- The in-product **Send Feedback** link (preferred)
- If you have a question, need help, or are experiencing an issue with your code, ask the developer community on [Microsoft Q&A](/answers/topics/office-addins-dev.html).
- You can also post questions to [Stack Overflow](https://stackoverflow.com/questions/tagged/Office365Connectors?sort=newest) - Tag your questions with `Office365Connectors`.
- If you have a feature suggestion, please post your idea on our [**Microsoft 365 Developer Platform Ideas**](https://aka.ms/m365dev-suggestions) forum, and vote for your suggestions there.
