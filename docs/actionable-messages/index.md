---
title: What are actionable messages in Office 365?
description: Actionable Messages enable you to take quick actions right from within Outlook.
author: jasonjoh
ms.topic: article
ms.service: outlook
ms.author: jasonjoh
ms.date: 08/01/2019
nodes_to_expand: get-started
ms.localizationpriority: high
ms.subservice: o365-connectors
---

# Actionable messages in Outlook and Office 365 Groups

[!INCLUDE [global-onboarding-paused-notice](../includes/actionable-messages/global-onboarding-paused-notice.md)]

Whether you are filling out a survey, approving an expense report, or updating a CRM sales opportunity, Actionable Messages enable you to take quick actions right from within Outlook. Developers can now embed actions in their emails or notifications, elevating user engagement with their services and increasing organizational productivity.

Office 365 provides two solutions to enhance productivity with Outlook Actionable Messages: actionable messages via email, and actionable messages via Office connectors.

> [!NOTE]
> Actionable messages are also available in Microsoft Teams. See [Office connectors for Teams](/microsoftteams/platform/concepts/connectors/connectors) for more information.

## User experience

Let's take a look at the end-to-end user experience for both an email-based and a connectors-based actionable message scenario.

### Actionable messages via email: group membership request approval scenario

A Contoso employee submits a request to join a private Office 365 group. Office 365 sends an Actionable Message to the person who owns the group to approve or decline the request. The card included in the message contains all the information the approver might need to quickly understand who submitted the request and any message they included to explain their request. It also includes **Approve** and **Decline** actions that can be taken right from Outlook. The owner approves the request, and the card updates to indicate the outcome.

![A join group request message card rendered in Outlook on iOS.](images/group-join-request-ios.png)

The new member of the group submits a second request to add her team members to the group. Office 365 sends an Actionable Message to the owner with clear information about who submitted the request and the new members to add. The recipient can approve all, some, or none of the proposed new members. The owner approves one new member, and the card updates to indicate the outcome. The approved member is no longer selectable, while the remaining member remains selectable.

![An add group members message card showing the first requested member being approved as rendered in Outlook on iOS.](images/group-add-members-request-ios-1.png)

The owner declines the other requested new member, and the card updates to indicate the outcome. Both members are no longer selectable, and the action buttons are removed.

![An add group members message card showing the second requested member being declined as rendered in Outlook on iOS.](images/group-add-members-request-ios-2.png)

### Actionable messages via Office connectors: task management scenario

Adele Vance and her team use Trello as their task management system. Adele has configured the Trello connector in her account, and will receive granular notifications as activity occurs in the Trello boards she is interested in.

Shiva, in Adele's team, creates a new Trello card in the "Hiring" board. He needs the latest job postings to be published. Adele receives an actionable message that tells her all about the new card and the task it represents: who created it, in which list, what the due date is, and more.

![A Trello connector card with actions.](images/trello-card-actions.png)

Adele has a few notes she recently took on a piece of paper with important things that should be mentioned in the job postings. She decides to add these as a comment to the Trello card. She clicks the **Add a comment** action, and is presented with a text input field in which she can type her notes:

![The "Add a comment" action UI with a text input.](images/trello-card-add-comment.png)

Adele then clicks the **Save** button, and the notes are immediately saved to the Trello card. A confirmation appears at the bottom of the message:

![The confirmation message.](images/trello-card-infobar.png)

## Office connectors

Office connectors are a great way to get useful information and content into your Office 365 Groups in Outlook or Microsoft Teams. Any user can connect their group or team to services like Trello, Bing News, Twitter, etc., and get notified of activity from that service. From tracking a team's progress in Trello, to following important hashtags in Twitter, Office connectors make it easier for an Office 365 group in Outlook, Microsoft Teams, or Yammer to stay in sync and get more done.

## Accessing Office connectors from Outlook

Office connectors are available for both the inbox and Groups for any Office 365 Mail user. Connectors can be managed in either Outlook on the web, or Outlook 2016 or later on Windows.

### Accessing connectors in Outlook on the web

#### Inbox connectors

Users access inbox connectors from the **Settings** menu, accessed from the gear icon in the top-right corner.

![A screenshot of the Settings menu in Outlook on the web.](images/web-inbox-connector-menu.png)

#### Groups connectors

Users access group connectors from the **Group settings** menu, accessed from the gear icon in the group title bar.

![A screenshot of the Group settings icon in Outlook on the web.](images/web-group-settings-icon.png)

![A screenshot of the Group settings menu in Outlook on the web.](images/web-group-connector-menu.png)

#### Accessing connectors in Outlook 2016 on Windows

Users access both inbox and group connectors from the **Store** button. The Store button loads connectors for the inbox if Outlook is currently displaying a mail folder in the user's mailbox, and loads connectors for the currently selected group if displaying a group.

![A screenshot of the Store button in Outlook 2016 on Windows.](images/win-store-icon.png)

In the inbox case, the Store dialog displays the **Add-ins** tab by default. Connectors are available in the **Connectors** tab.

![A screenshot of the Connectors tab in the Store dialog in Outlook 2016 on Windows.](images/win-store-connectors-tab.png)

## Release Notes

Currently, you can only configure connectors from Outlook on the web, or Outlook 2016 or later on Windows, but you can view information posted by Connectors to your Group or inbox in multiple clients such as Outlook on the web, Outlook 2016 or later, and the Office365 Groups Mobile app.

For information on admin controls, look at the FAQ section of this [support article](https://support.office.com/article/Connect-apps-to-your-groups-ed0ce547-038f-4902-b9b3-9e518ae6fbab).

### Outlook version requirements for actionable messages

Actionable messages are available to all customer mailboxes on Exchange Online in Office 365 or Outlook.com with a supported client. The following table lists the availability of actionable messages for current Outlook clients. For information on the Office 365 release channels, see [Overview of update channels for Microsoft 365 apps](/deployoffice/updates/overview-update-channels). For current supported versions of Microsoft 365 Apps, see [supported versions](/officeupdates/update-history-microsoft365-apps-by-date#supported-versions).

> [!NOTE]
> Currently actionable message cards do not change the way that they render when Outlook is in dark mode. Support for dark mode for actionable messages is coming soon.

| Client                                                     | Actionable messages supported? | Adaptive card supported? |
|------------------------------------------------------------|--------------------------------|--------------------------|
| Outlook on the web for Microsoft 365                       | Yes                            | Yes |
| Microsoft 365 Apps Current Channel                         | Yes                            | Yes |
| Microsoft 365 Apps Monthly Enterprise Channel              | Yes                            | Yes |
| Microsoft 365 Apps Semi-Annual Enterprise Channel          | Yes                            | Yes |
| Outlook on Mac                                             | Yes, version 16.0.12817.10000  | Yes (Legacy MessageCard format is not supported) |
| Outlook on iOS                                             | Yes, version 4.34.0            | Yes (Legacy MessageCard format is not supported) |
| Outlook on Android                                         | Yes, version 4.1.71            | Yes (Legacy MessageCard format is not supported) |
| Office Professional Plus (Click-to-Run only), all versions | No                             | No |
| Exchange On-Premises Outlook on the web, all versions      | No                             | No |

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
