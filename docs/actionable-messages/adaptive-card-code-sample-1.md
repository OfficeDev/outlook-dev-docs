---
title: Actionable message using adaptive card version 1.4+
description: Demonstrating how to implement Universal Action Model in Outlook
author: avijityadav
ms.topic: sample
ms.technology: o365-connectors
ms.date: 04/08/2023
ms.author: avijityadav
ms.localizationpriority: high
---

# Universal Actions model code sample - Expense Approval

This sample illustrates the Universal Action Model implementation available for adaprive cards version 1.4 or higher.

## Prequistise
* Outlook/OWA client is available and you have an account
* [.NET Core SDK](https://dotnet.microsoft.com/en-us/download) version 6.0

## Setup for bot
* Register a new application in the [Azure Active Directory – App Registrations](https://go.microsoft.com/fwlink/?linkid=2083908) portal.
* Register a bot with Azure Bot Service, following the instructions [here](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-3.0).
* Ensure that you've [enabled the Outlook Channel](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-actionable-email?view=azure-bot-service-4.0)
* Request for access to send Actionable Messages.
    - Open your bot resource in the [Azure portal](https://ms.portal.azure.com/).
    - Open the **Channels** pane.
    - Select the **Outlook** channel.
    - On the **Configure Outlook** page, select **please register here**.
    - Fill out the registration form to request access. See [Register your service with the actionable email developer dashboard](./email-dev-dashboard.md) for more information.

## Step 1: Ensure your adaptive card payloads are ready

For the Approvals scenario, json payload is added here.

# [Mobile](#tab/mobile)

:::image type="content" source="~/images/approvals-mobile.png" alt-text="Mobile card for approvals":::

# [Desktop](#tab/desktop)

:::image type="content" source="~/images/approvals-desktop.png" alt-text="desktop card for approvals":::

* * *

Here is snippet for json around how you can include Univsal Actions in Microsoft.

```JSON
{
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.Execute",
              "title": "Accept",
              "verb": "approvalAccept",
              "data": {},
              "isPrimary": true,
              "style": "positive"
            },
            {
              "type": "Action.ShowCard",
              "id": "e1487cbc-66b0-037e-cdc4-045fb7d8d0b8",
              "title": "Reject",
              "card": {
                "type": "AdaptiveCard",
                "body": [
                  {
                    "type": "Input.Text",
                    "id": "Comment",
                    "placeholder": "Add a comment",
                    "isMultiline": true
                  },
                  {
                    "type": "ActionSet",
                    "actions": [
                      {
                        "type": "Action.Execute",
                        "title": "Submit",
                        "verb": "approvalReject",
                        "data": {
                          "comment": "${Comment.value}"
                        }
                      }
                    ]
                  }
                ],
                "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                "fallbackText": "Unable to render the card",
                "padding": "None"
              }
            }
          ],
          "spacing": "None"
        }
```

## Step 2: Write custom logic in the bot for approval




```C#
       protected override async Task<AdaptiveCardInvokeResponse> OnAdaptiveCardInvokeAsync(ITurnContext<IInvokeActivity> turnContext, AdaptiveCardInvokeValue invokeValue, CancellationToken cancellationToken)
        {
            try
            {
                // Approval Scenario
                if (invokeValue.Action.Verb == "approvalAccept")
                {
                    return await ProcessApprovalAccepted(); // This function can contain your business logic to capture the approval and show the refresh card
                }
                else if (invokeValue.Action.Verb == "approvalReject")
                {
                    return await ProcessApprovalRejected(); // This function can contain you business logic to capture the rejection and show the refresh card
                }
                else
                {
                    throw new InvokeResponseException(HttpStatusCode.NotImplemented);
                }
            }
            catch (AdaptiveCardActionException e)
            {
                throw new InvokeResponseException(HttpStatusCode.NotImplemented, e.Response);
            }
        }
```

## Sending the Actionable Message

You can send the Actionable Message backed by Universal Actions similar to any other Actionable message. For more information, please see [Send an actionable message via email](./send-via-email.md).


