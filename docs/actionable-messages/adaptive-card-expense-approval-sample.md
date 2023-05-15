---
title: Universal Actions Model code sample - Expense Approval
description: Demonstrating how to implement Universal Actions in Outlook with Expense Approval scenario
author: avijityadav
ms.topic: sample
ms.service: outlook
ms.subservice: o365-connectors
ms.date: 04/08/2023
ms.author: avyad
ms.localizationpriority: high
---

# Universal Actions Model code sample - Expense Approval

This sample illustrates the Universal Action Model implementation available for adaptive cards version 1.4 or higher.

## Prerequisites

- Outlook/OWA client is available and you have an account
- [.NET Core SDK](https://dotnet.microsoft.com/download) version 6.0

## Setup for bot

- Register a new application in the [Azure Active Directory – App Registrations](https://go.microsoft.com/fwlink/?linkid=2083908) portal.
- Register a bot with Azure Bot Service, following the instructions [here](/azure/bot-service/bot-service-quickstart-registration).
- Ensure that you've [enabled the Outlook Channel](/azure/bot-service/bot-service-channel-connect-actionable-email)
- Request for access to send Actionable Messages.
  - Open your bot resource in the [Azure portal](https://ms.portal.azure.com/).
  - Open the **Channels** pane.
  - Select the **Outlook** channel.
  - On the **Configure Outlook** page, select **please register here**.
  - Fill out the registration form to request access. See [Register your service with the actionable email developer dashboard](./email-dev-dashboard.md) for more information.

## Step 1: Ensure your adaptive card payloads are ready

For the approvals scenario, you can find the [JSON payload here](https://github.com/OfficeDev/outlook-dev-docs/blob/main/files/actionable-messages/samples/Approval.json). Below, you can see the payload rendering in mobile and desktop screens.

<!-- markdownlint-disable MD051 -->
### [Mobile](#tab/mobile)

:::image type="content" source="./images/approvals-mobile.png" alt-text="Mobile card for approvals":::

### [Desktop](#tab/desktop)

:::image type="content" source="./images/approvals-desktop.png" alt-text="Desktop card for approvals":::

---
<!-- markdownlint-enable MD051 -->

For Universal Actions, you need to use `Action.Execute` which gathers input fields and send an `Invoke` activity of type `adaptiveCard/action` to the target bot. Target bot can identify the the Action done using the `verb` field. Any additional input can be sent using the `data` field.

### JSON payload

Here is a snippet of Actions for Approval scenario.

:::code language="json" source="samples/Approval.json" range="652-697":::

For more information, see [Action.Execute schema and properties](/adaptive-cards/authoring-cards/universal-action-model#actionexecute)

## Step 2: Write custom logic in the bot for approval

In the Azure bot, you can write logic to capture the action using the `verb` field, add you business logic and send the refresh card back to Outlook.

```csharp
protected override async Task<AdaptiveCardInvokeResponse> OnAdaptiveCardInvokeAsync(
    ITurnContext<IInvokeActivity> turnContext,
    AdaptiveCardInvokeValue invokeValue,
    CancellationToken cancellationToken)
{
    try
    {
        // Approval Scenario
        if (invokeValue.Action.Verb == "approvalAccept")
        {
            // This function can contain your business logic
            // to capture the approval and show the refresh card
            return await ProcessApprovalAccepted();
        }
        else if (invokeValue.Action.Verb == "approvalReject")
        {
            // This function can contain you business logic
            // to capture the rejection and show the refresh card
            return await ProcessApprovalRejected();
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

## Step 3: Sending the Actionable Message

You can send the Actionable Message backed by Universal Actions similar to any other Actionable message. For more information, please see [Send an actionable message via email](./send-via-email.md).
