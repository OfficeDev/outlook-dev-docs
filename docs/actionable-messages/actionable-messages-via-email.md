---
title: Get started with actionable messages via email | Microsoft Docs
description: Learn how to create an actionable message card and send it via email.
author: jasonjoh

ms.topic: get-started-article
ms.technology: o365-connectors
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Send an actionable message via email in Office 365

## Create an actionable message card

Let's start by creating an actionable message card. We'll start with something simple, just a basic card with an `HttpPOST` action and an `OpenUri` action. We'll use the [Card Playground](https://messagecardplayground.azurewebsites.net/) to design the card.

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
      "@type": "ActionCard",
      "name": "Send Feedback",
      "inputs": [
        {
          "@type": "TextInput",
          "id": "feedback",
          "isMultiline": true,
          "title": "Let us know what you think about Actionable Messages"
        }
      ],
      "actions": [
        {
          "@type": "HttpPOST",
          "name": "Send Feedback",
          "isPrimary": true,
          "target": "http://..."
        }
      ]
    },
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
> You can design and test actionable messages by using the [Card Playground](https://messagecardplayground.azurewebsites.net/), which allows you to send actionable messages to yourself. You can also send actionable messages to yourself using the [Office 365 SMTP server](https://support.office.com/en-us/article/POP-and-IMAP-settings-for-Outlook-Office-365-for-business-7fc677eb-2491-4cbc-8153-8e7113525f6c). You will be unable to send actionable messages to any other user until you have registered using the [actionable messages developer dashboard](https://aka.ms/publishoam).

To embed an actionable message card in an email message, we need to wrap the card in a `<script>` tag. The `<script>` tag is then inserted into the `<head>` of the email's HTML body.

> [!NOTE]
> Because the card JSON must be wrapped in a `<script>` tag, the body of the actionable message email MUST be HTML. Plain-text messages are not supported.

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
          "@type": "ActionCard",
          "name": "Send Feedback",
          "inputs": [
            {
              "@type": "TextInput",
              "id": "feedback",
              "isMultiline": true,
              "title": "Let us know what you think about Actionable Messages"
            }
          ],
          "actions": [
            {
              "@type": "HttpPOST",
              "name": "Send Feedback",
              "isPrimary": true,
              "target": "http://..."
            }
          ]
        },
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
          "@type": "ActionCard",
          "name": "Send Feedback",
          "inputs": [
            {
              "@type": "TextInput",
              "id": "feedback",
              "isMultiline": true,
              "title": "Let us know what you think about Actionable Messages"
            }
          ],
          "actions": [
            {
              "@type": "HttpPOST",
              "name": "Send Feedback",
              "isPrimary": true,
              "target": "http://..."
            }
          ]
        },
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
            "@type": "ActionCard",
            "name": "Send Feedback",
            "inputs": [
              {
                "@type": "TextInput",
                "id": "feedback",
                "isMultiline": true,
                "title": "Let us know what you think about Actionable Messages"
              }
            ],
            "actions": [
              {
                "@type": "HttpPOST",
                "name": "Send Feedback",
                "isPrimary": true,
                "target": "http://..."
              }
            ]
          },
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