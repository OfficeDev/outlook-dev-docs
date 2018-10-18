---
title: Connectors developer dashboard
description: Register your connector in our developer portal, add an integrated configuration experience to your application, and implement your connector.
author: jasonjoh
ms.topic: article
ms.technology: o365-connectors
ms.date: 08/02/2018
ms.author: jasonjoh
---

# Register your connector with the Office 365 Connectors developer dashboard

Building an Office 365 Connector for your application is easy. All you need to do is register your connector in our developer portal, add an integrated configuration experience to your application, and implement your connector. You can make it easy for your users to discover the connector by publishing to our catalog.

## Build your own Connector

### Registering your Connector

Visit the [Connector Developer Portal](https://aka.ms/connectorsdashboard) and login with your Office 365 credentials. If you do not have an Office 365 subscription you can get a one year [FREE Office 365 Subscription](https://developer.microsoft.com/office/dev-program) under the Office 365 Developer Program.

Choose **New Connector** and fill out the form. Once you choose **Save**, new options appear on the page.

- The **Sideload to Outlook** button will temporarily add your connector into the logged on user's Outlook experience. That user will be able to configure the connector either in their inbox or on any group that user is a member of.
- The **Publish to Store** button will start the publish process, including review by Microsoft for listing in the store.

### Adding an integrated configuration experience

An integrated configuration experience allows the user to configure your connector without leaving Outlook. Your application exposes the configuration as a web page that utilizes the Microsoft Teams JavaScript library to communicate configuration status back to Outlook.

For details, see [Integrating the configuration experience](https://docs.microsoft.com/microsoftteams/platform/concepts/connectors/connectors-creating#integrating-the-configuration-experience).

> [!NOTE]
> The **Integrating the configuration experience** document is Microsoft Teams-specific, but the documented methods work the same way in Outlook.

#### Outlook-specific configuration requirements

If your configuration experience requires authentication, there are additional requirements to enable the [authentication flow](https://docs.microsoft.com/microsoftteams/platform/concepts/authentication/auth-flow-tab) in Outlook on Windows.

Outlook on Windows passes an additional query parameter to your connectors authentication start page.

```http
callbackUrl=<connectors url>
```

Your app must preserve this value and pass it as an additional parameter to the `microsoftTeams.authentication.notifySuccess` or `microsoftTeams.authentication.notifyFailure` methods once authentication is complete.

```js
microsoftTeams.authentication.notifySuccess(result, callbackUrl);
```

## Publish your Connector to the Store

Once you have thoroughly tested your connector and it is ready to be listed in the Office 365 connector catalog, you can use the **Publish to Store** button to submit it for review. Once reviewed and approved your connector would be added to the connector catalog.

### Connector submission checklist

- Ensure that your connector is fully functional and thoroughly tested before submitting it to the Store.
- Test your connector cards in various clients where your users would use it: Outlook on the Web, Outlook 2016 or later, and Outlook Groups mobile apps.
- Ensure that you strictly use Markdown for text decoration and not send HTML in your connector card payload.
- Maintain a balance between adding value and generating too much noise. Ensure that the user is not bogged down with too many notifications.
- Identify the right events to send connector cards for. Ensure that the information you send to the group is valuable to the members of the group.
- When sending reports or summaries, use a digest format and allow the user to choose the time and frequency of the reports.
- When sending connector cards make the best use of Markdown to highlight important parts of the card.
- Make your connector cards actionable by providing relevant actions whenever possible.
- Actions invoked should be really low in failure rate and should have fast response by your endpoint.
- Ensure that you have provisions for the user to pause or remove the configuration.
- Have clear user-facing documentation on the capabilities your connector offers.
- When registering your connector:
  - Ensure that the name and logo of your connector does not infringe upon a trademark or copyright of any other product or service.
  - Provide a high quality logo of type jpg, jpeg, png or gif that is under 60KB in size.
  - Provide a short description of your application (e.g. 'Contoso Help Desk brings companies and customers together').
  - Provide a detailed description of your connector (e.g. 'The Contoso Help Desk connector notifies your Office 365 group about activity on your customer's tickets').
- When publishing your connector to Store:
  - Make sure to fill out step by step instructions and share test account information to let us test your connector.

## Next steps

If you have any feedback share it on <a target="_blank" href="https://officespdev.uservoice.com/forums/224641-general/category/146379-connectors">UserVoice</a>. Post your questions on Stack Overflow and tag them with [Office365Connectors](https://stackoverflow.com/questions/tagged/Office365Connectors?sort=newest).
