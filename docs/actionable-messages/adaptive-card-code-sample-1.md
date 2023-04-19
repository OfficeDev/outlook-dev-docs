---
title: Universal Actions Model code sample - Expense Approval
description: Demonstrating how to implement Universal Actions in Outlook with Expense Approval scenario
author: avyad
ms.topic: sample
ms.technology: o365-connectors
ms.date: 04/08/2023
ms.author: avijityadav
ms.localizationpriority: high
---

# Universal Actions Model code sample - Expense Approval

This sample illustrates the Universal Action Model implementation available for adaptive cards version 1.4 or higher.

## Prequistise
* Outlook/OWA client is available and you have an account
* [.NET Core SDK](https://dotnet.microsoft.com/download) version 6.0

## Setup for bot
* Register a new application in the [Azure Active Directory – App Registrations](https://go.microsoft.com/fwlink/?linkid=2083908) portal.
* Register a bot with Azure Bot Service, following the instructions [here](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-3.0).
* Ensure that you've [enabled the Outlook Channel](https://learn.microsoft.com/azure/bot-service/bot-service-channel-connect-actionable-email?view=azure-bot-service-4.0)
* Request for access to send Actionable Messages.
    - Open your bot resource in the [Azure portal](https://ms.portal.azure.com/).
    - Open the **Channels** pane.
    - Select the **Outlook** channel.
    - On the **Configure Outlook** page, select **please register here**.
    - Fill out the registration form to request access. See [Register your service with the actionable email developer dashboard](./email-dev-dashboard.md) for more information.

## Step 1: Ensure your adaptive card payloads are ready

For the Approvals scenario, you can find the [JSON Payload here](./Approval.json). Below, You can see the payload rendering in mobile and desktop screens. 

# [Mobile](#tab/mobile)

:::image type="content" source="./images/approvals-mobile.png" alt-text="Mobile card for approvals":::

# [Desktop](#tab/desktop)

:::image type="content" source="./images/approvals-desktop.png" alt-text="Desktop card for approvals":::

* * *

For Universal Actions, you need to use `Action.Execute` which gathers input fields and send an event Invoke activity of type adaptiveCard/action to the target Bot. Target bot can identify the the Action done using the `verb` field. Any additional input can be sent using the `data` field.

Here is a snippet of Actions for Approval scenario.

```JSON
{
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

For more information, see [Action.Execute schema and properties](https://learn.microsoft.com/adaptive-cards/authoring-cards/universal-action-model#actionexecute)

## Step 2: Write custom logic in the bot for approval

In the Azure bot, you can write logic to capture the action using the `verb` field, add you business logic and send the refresh card back to Outlook.

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


