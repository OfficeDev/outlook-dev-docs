---
title: Get started with actionable messages via email | Microsoft Docs
description: Learn how to create an actionable message card and send it via email.
author: jasonjoh

ms.topic: get-started-article
ms.date: 04/11/2017
ms.author: jasonjoh
---

# Get started with actionable messages via email

## Create an actionable message card

Let's start by creating an actionable message card. We'll start with something simple, just a basic card with a single `OpenUri` action. We'll use the [Card Playground](https://messagecardplayground.azurewebsites.net/) to design the card.

Go to [Card Playground](https://messagecardplayground.azurewebsites.net/) and paste in the following JSON:

```json
{
  "@context": "http://schema.org/extensions",
  "@type": "MessageCard",
  "themeColor": "0072C6",
  "title": "Visit the Outlook Dev Portal",
  "text": "Click **Learn More** to learn more about Actionable Messages!",
  "potentialAction": [
    {
      "@type": "OpenUri",
      "name": "Learn More",
      "targets": [
        { "os": "default", "uri": "https://docs.microsoft.com/en-us/outlook/actionable-messages" }
      ]
    }
  ]
}
```

Feel free to experiment with this simple example in the playground. You can see the [card reference](card-reference.md) for details on the available fields. Once you have a card you're happy with, you can move on to sending it.

## Sending actionable messages via email

> [!IMPORTANT]
> If you have not yet [registered](https://aka.ms/gotactions) or are waiting for approval, you will be unable to send actionable messages via email users outside of your organization. However, you can send actionable messages to yourself or other users inside your organization using the [Office 365 SMTP server](https://support.office.com/en-us/article/POP-and-IMAP-settings-for-Outlook-Office-365-for-business-7fc677eb-2491-4cbc-8153-8e7113525f6c).

To embed an actionable message card in an email message, we need to wrap the card in a `<script>` tag. The `<script>` tag is then inserted into the `<head>` of the email's HTML body.

1. Add the `hideOriginalBody` attribute to control what happens with the body of the email. In this case we'll set the attribute to `true` so that the body will not be shown.

    ```json
    {
      "@context": "http://schema.org/extensions",
      "@type": "MessageCard",
      "hideOriginalBody": "true",
      "themeColor": "0072C6",
      "title": "Visit the Outlook Dev Portal",
      "text": "Click **Learn More** to learn more about Actionable Messages!",
      "potentialAction": [
        {
          "@type": "OpenUri",
          "name": "Learn More",
          "targets": [
            { "os": "default", "uri": "https://docs.microsoft.com/en-us/outlook/actionable-messages" }
          ]
        }
      ]
    }
    ```

1. Wrap the resulting JSON in a `<script>` tag of type `application/ld+json`.

    ```html
    <script type="application/ld+json">{
      "@context": "http://schema.org/extensions",
      "@type": "MessageCard",
      "hideOriginalBody": "true",
      "themeColor": "0072C6",
      "title": "Visit the Outlook Dev Portal",
      "text": "Click **Learn More** to learn more about Actionable Messages!",
      "potentialAction": [
        {
          "@type": "OpenUri",
          "name": "Learn More",
          "targets": [
            { "os": "default", "uri": "https://docs.microsoft.com/en-us/outlook/actionable-messages" }
          ]
        }
      ]
    }
    </script>
    ```

1. Generate an HTML document to represent the email body and include the `<script>` tag in the `<head>`.

    ```html
    <html>
    <head>
      <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
      <script type="application/ld+json">{
        "@context": "http://schema.org/extensions",
        "@type": "MessageCard",
        "hideOriginalBody": "true",
        "themeColor": "0072C6",
        "title": "Visit the Outlook Dev Portal",
        "text": "Click **Learn More** to learn more about Actionable Messages!",
        "potentialAction": [
          {
            "@type": "OpenUri",
            "name": "Learn More",
            "targets": [
              { "os": "default", "uri": "https://docs.microsoft.com/en-us/outlook/actionable-messages" }
            ]
          }
        ]
      }
      </script>
    </head>
    <body>
    Visit the <a href="https://docs.microsoft.com/en-us/outlook/actionable-messages">Outlook Dev Portal</a> to learn more about Actionable Messages.
    </body>
    </html>
    ```

1. Send a message via SMTP with the HTML as the body.

### Sending the message

For examples of sending messages, see the following.

- [Send Actionable Message via the Microsoft Graph](https://github.com/jasonjoh/send-actionable-message): A sample console app written in C# that sends an actionable message using the [Microsoft Graph](https://graph.microsoft.io/en-us/docs/api-reference/v1.0/api/message_send).
- [Send Actionable Message via SMTP](https://gist.github.com/jasonjoh/3ec367594c3fa662ee983a617bdc7deb): A sample Python script that sends an actionable message using the Office 365 SMTP server.