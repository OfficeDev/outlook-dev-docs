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

# Universal Actions Model code sample - Project Management

This sample illustrates the Universal Action Model implementation available for adaptive cards version 1.4 or higher.

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

For the Project management scenario, you can find the [JSON Payload here](./ProjectManagement.json). Below, You can see the payload rendering in mobile and desktop screens. 

# [Mobile](#tab/mobile)

:::image type="content" source="./images/projectmanagement-mobile.png" alt-text="Mobile card for project management":::

# [Desktop](#tab/desktop)

:::image type="content" source="./images/projectmanagement-desktop.png" alt-text="Desktop card for project management":::

* * *

For Universal Actions, you need to use `Action.Execute` which gathers input fields and send an event Invoke activity of type adaptiveCard/action to the target Bot. Target bot can identify the the Action done using the `verb` field. Any additional input can be sent using the `data` field.

Here is a snippet of Actions for Project management scenario.

```JSON
{
         {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.Execute",
              "title": "Mark complete",
              "verb": "markComplete",
              "data": {},
              "isPrimary": true,
              "style": "positive"
            },
            {
              "type": "Action.ShowCard",
              "title": "Add a comment",
              "card": {
                "type": "AdaptiveCard",
                "body": [
                  {
                    "type": "Input.Text",
                    "id": "AddComment",
                    "placeholder": "Add a comment",
                    "isMultiline": true
                  },
                  {
                    "type": "ActionSet",
                    "actions": [
                      {
                        "type": "Action.Execute",
                        "title": "Submit",
                        "verb": "projectSubmitComment",
                        "data": "{Reason: {{AddComment.value}}}"
                      }
                    ]
                  }
                ],
                "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
                "padding": "None"
              }
            }
          ],
          "spacing": "None"
        }
```

For more information, see [Action.Execute schema and properties](https://learn.microsoft.com/en-us/adaptive-cards/authoring-cards/universal-action-model#actionexecute)

## Step 2: Write custom logic in the bot for Project management

In the Azure bot, you can write logic to capture the action using the `verb` field, add you business logic and send the refresh card back to Outlook.

```C#
        protected override async Task<AdaptiveCardInvokeResponse> OnAdaptiveCardInvokeAsync(ITurnContext<IInvokeActivity> turnContext, AdaptiveCardInvokeValue invokeValue, CancellationToken cancellationToken)
        {
            try
            {
                if (invokeValue.Action.Verb == "markComplete")
                {
                    return await ProcessProjectMarkComplete();
                }
                else if (invokeValue.Action.Verb == "projectSubmitComment")
                {
                    return await ProcessProjectSubmitComment();
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


