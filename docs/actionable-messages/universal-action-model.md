---
title: Overview of Universal Action Model
description: Learn more about Universal Actions for Adaptive Cards in Outlook
author: avijityadav
ms.topic: conceptual
ms.service: outlook
ms.date: 05/18/2023
ms.author: avyad
ms.localizationpriority: high
ms.subservice: o365-connectors
---

# Overview of Universal Action Model

Adaptive Cards are platform-agnostic snippets of UI, authored using a lightweight JSON format, that apps and services can share. Adaptive Cards not only adapt to the look-and-feel of the host, but also provide rich interaction capabilities.

As Adaptive Cards grew in popularity, different hosts started supporting different action models and this led to fragmentation. To solve this problem, the Teams, Outlook and Adaptive Cards teams worked on creating a new universal Bot action model compatible across hosts. This effort led to the following:

- The generalization of Bots and the Bot Framework as the way to implement Adaptive Card-based scenarios for both Outlook (Actionable Messages) and Teams (Bots)
- `Action.Execute` as a replacement for both `Action.Http` (currently used by Actionable Messages) and `Action.Submit` (currently used by Bots).

## Schema

> [!IMPORTANT]
> Universal Action Model is introduced in Adaptive card schema version 1.4 and above. In order to use these new capabilities, the `version` property of your Adaptive Card must be set to 1.4 or greater, as shown in below examples.

### Action.Execute

When authoring Adaptive Cards for Outlook, use `Action.Execute` in place of `Action.Http`. Here is an example of adaptive card using `Action.Execute` in the JSON payload.

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "originator":"c9b4352b-a76b-43b9-88ff-80edddaa243b",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "Present a form and submit it back to the originator"
    },
    {
      "type": "Input.Text",
      "id": "firstName",
      "placeholder": "What is your first name?"
    },
    {
      "type": "Input.Text",
      "id": "lastName",
      "placeholder": "What is your last name?"
    },
    {
      "type": "ActionSet",
      "actions": [
        {
          "type": "Action.Execute",
          "title": "Submit",
          "verb": "personalDetailsFormSubmit"
        }
      ]
    }
  ]
}
```

#### Properties

| Property     | Type                        | Required | Description |
|--------------|-----------------------------|----------|-------------|
| **type**     | `string`                    | Yes      | Must be `Action.Execute`. |
| **verb**     | `string`                    | No       | A convenience string that can be used by developer to identify the action |
| **data**     | `string`, `object`          | No       | Initial data that input fields will be combined with. These are essentially hidden properties. |
| **title**    | `string`                    | No       | Label for button or link that represents this action. |
| **iconUrl**  | `uri`                       | No       | Optional icon to be shown on the action in conjunction with the title. Supports data URI in Adaptive Cards version 1.2+ |
| **style**    | `ActionStyle`               | No       | Controls the style of an Action, which influences how the action is displayed, spoken, etc. |
| **fallback** | `<action object>`, `"drop"` | No       | Describes what to do when Action.Execute is unsupported by the client displaying the card. |
| **requires** | `Dictionary<string>`        | No       | A series of key/value pairs indicating features that the item requires with corresponding minimum version. When a feature is missing or of insufficient version, fallback is triggered. |

### Refresh mechanism

Alongside `Action.Execute`, a new refresh mechanism is now supported, making it possible to create Adaptive Cards that automatically update at the time they are displayed. This ensures that users always see up-to-date data. To allow an Adaptive Card to automatically refresh in Outlook, define its `refresh` property, which embeds an `action` of type `Action.Execute`.

| Property | Type | Required | Description
| -------- | ---- | -------- | -----------
| **action** | `"Action.Execute"` | Yes | Must be an action instance of type `"Action.Execute"`. |
| **userIds** | `Array<string>` | Yes | An array of Message Resource Identifier (`MRI`) of users for whom Auto Refresh must be enabled in Teams. For Teams, refer to [User IDs in refresh](/microsoftteams/platform/task-modules-and-cards/cards/universal-actions-for-adaptive-cards/work-with-universal-actions-for-adaptive-cards#user-ids-in-refresh) to learn more. Note that the `userIds` property is ignored in Outlook, and the `refresh` property is always automatically honored.|

#### Sample JSON

```JSON
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "originator":"c9b4352b-a76b-43b9-88ff-80edddaa243b",
  "version": "1.4",
  "refresh": {
    "action": {
      "type": "Action.Execute",
      "title": "Submit",
      "verb": "personalDetailsCardRefresh"
    },
    "userIds": []
  },
  "body": [
    {
      "type": "TextBlock",
      "text": "Present a form and submit it back to the originator"
    },
    {
      "type": "Input.Text",
      "id": "firstName",
      "placeholder": "What is your first name?"
    },
    {
      "type": "Input.Text",
      "id": "lastName",
      "placeholder": "What is your last name?"
    },
    {
      "type": "ActionSet",
      "actions": [
        {
          "type": "Action.Execute",
          "title": "Submit",
          "verb": "personalDetailsFormSubmit",
          "fallback": "Action.Submit"
        }
      ]
    }
  ]
}
```

## Backward compatibility

The new `Action.Execute` universal action model is a departure from the `Action.Http` action model used in adaptive card version below 1.4, where actions are encoded in the Adaptive Card as explicit HTTP calls. The Action.Execute model makes it possible for developers to implement scenarios that work in both Outlook and Teams. Actionable Message scenarios can either use the `Action.Http` model or the new `Action.Execute` model, but not both. Scenarios that use the universal `Action.Execute` model must be implemented as Bots and subscribe to the **Outlook Actionable Messages** channel.

## Code samples

Once you have gone through the documentation, you are ready to start trying with some examples.

- [Expense Approvals](./adaptive-card-expense-approval-sample.md)
- [Project management](./adaptive-card-project-management-sample.md)
