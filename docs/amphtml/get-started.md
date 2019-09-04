---
title: Get Started
description: How to create and test AMP emails in Outlook.com
author: LezaMax
ms.topic: article
ms.technology: o365-amp
ms.author: humlezg
localization_priority: Priority
---
# Get Started with AMP for Email

## Author AMP for Email code

> [!NOTE]
> If you already created an AMP email that works in another email provider you can skip to the [next section](#test-amp-emails-in-outlookcom). Your email should work "as is".

1. Read the [AMP for Email specification](https://amp.dev/documentation/guides-and-tutorials/learn/amp-email-format) to understand the fundamentals.
1. If this is your first time creating an AMP email we recommend you to start with this [tutorial](https://amp.dev/documentation/guides-and-tutorials/start/create_email/?format=email).
1. Develop your code using [AMP Components](https://amp.dev/documentation/components/?format=email) and test using the [AMP Playground](https://playground.amp.dev/?runtime=amp4email).

## Test AMP emails in Outlook.com

When developing AMP emails, you can use your own Outlook.com account to test them.

1. Register the sender address with Outlook. During development you can self-register. See [Sender registration](register-outlook.md) for details.
1. Send AMP email to your Outlook.com mailbox via your own SMTP server or an ESP that supports the format such as [Mailgun](https://www.mailgun.com/blog/mailgun-supports-amp-email), [AWeber](https://help.aweber.com/hc/en-us/articles/360025741194-Getting-Started-with-AMP-for-Email-in-AWeber), [SparkPost](https://www.sparkpost.com/docs/user-guide/amp-for-email/) and others. See [AMP Tools](https://amp.dev/documentation/tools/) for a full list.

## Send emails to Outlook.com users

Once you are ready to send emails to Outlook.com end users (your actual customers) you must [register to become an authorized sender](register-outlook.md).
