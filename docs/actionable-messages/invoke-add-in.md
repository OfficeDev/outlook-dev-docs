---
title: Invoke an Outlook add-in from an actionable message
description: Learn how to invoke an Outlook add-in from an actionable message and pass initialization data to the add-in.
author: jasonjoh
ms.topic: article
ms.technology: o365-connectors
ms.date: 05/07/2018
ms.author: jasonjoh
---

# Invoke an Outlook add-in from an actionable message

Actionable messages allow the user to take quick actions on an email message or connector card, and Outlook add-ins allow you to extend Outlook to add new features and interactions. Now, with the `Action.InvokeAddInCommand` action type, you can combine these two types of integrations to create more powerful and compelling experiences. For example, you could:

- Send a welcome message as an actionable email message to new users after they sign up for your service, with an action that allows them to quickly install and start using your add-in.
- Use an add-in for more complex actions (i.e. to present a form to the user), for scenarios where a simple action input would not suffice.
- Pre-populate UI elements in your add-in before presenting UI to the user.

`Action.InvokeAddInCommand` actions can work with add-ins that are already installed by the user, or they can work with add-ins that are not installed. If the required add-in is not installed, the user is prompted to install the add-in with a single click.

> [!NOTE]
> Single-click installation of the required add-in is only supported if the add-in is published in the [Office Store](https://docs.microsoft.com/office/dev/store/submit-to-the-office-store).

The following example shows the prompt users see if the add-in is not installed.

![A screenshot of the prompt to install an add-in when invoked from an actionable message](images/invoke-mha-add-in.png)

## Invoking the add-in

Actionable messages invoke add-ins by specifying an [Action.InvokeAddInCommand action](adaptive-card.md#actioninvokeaddincommand) in the message. This action specifies the add-in to invoke, along with the identifier of the add-in button that opens the appropriate task pane.

The required information is found in the [add-in's manifest](../add-ins/manifests.md). First, you'll need the add-in's identifier, which is specified in the [Id element](https://docs.microsoft.com/office/dev/add-ins/reference/manifest/id).

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<OfficeApp
  xmlns="http://schemas.microsoft.com/office/appforoffice/1.1"
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:bt="http://schemas.microsoft.com/office/officeappbasictypes/1.0"
  xsi:type="MailApp">
  <Id>527104a1-f1a5-475a-9199-7a968161c870</Id>
  <Version>1.0.0.0</Version>
  ...
</OfficeApp>
```

For this add-in, the add-in identifier is `527104a1-f1a5-475a-9199-7a968161c870`.

Next, you'll need the `id` attribute of the [Control element](https://docs.microsoft.com/office/dev/add-ins/reference/manifest/control) that defines the add-in button that opens the appropriate task pane. Keep in mind that the `Control` element MUST:

- Be defined inside a [MessageReadCommandSurface extension point](https://docs.microsoft.com/office/dev/add-ins/reference/manifest/extensionpoint#messagereadcommandsurface)
- Have its `xsi:type` attribute set to `Button`
- Contain an [Action element](https://docs.microsoft.com/office/dev/add-ins/reference/manifest/action) of type `ShowTaskpane`

```xml
<ExtensionPoint xsi:type="MessageReadCommandSurface">
  <OfficeTab id="TabDefault">
    <Group id="msgReadCmdGroup">
      <Label resid="groupLabel"/>
      <Control xsi:type="Button" id="showInitContext">
        <Label resid="readButtonLabel"/>
        <Supertip>
          <Title resid="readButtonTitle"/>
          <Description resid="readButtonDesc"/>
        </Supertip>
        <Icon>
          <bt:Image size="16" resid="icon-16"/>
          <bt:Image size="32" resid="icon-32"/>
          <bt:Image size="80" resid="icon-80"/>
        </Icon>
        <Action xsi:type="ShowTaskpane">
          <SourceLocation resid="readPaneUrl"/>
          <SupportsPinning>true</SupportsPinning>
        </Action>
      </Control>
    </Group>
  </OfficeTab>
</ExtensionPoint>
```

For this add-in button, the ID is `showInitContext`.

With these two pieces of information, we can create a basic `Action.InvokeAddInCommand` action as follows:

```json
{
  "type": "AdaptiveCard",
  "version": "1.0",
  "body": [
    {
      "type": "TextBlock",
      "text": "Invoking an add-in command from an Actionable Message card",
      "size": "large"
    }
  ],
  "actions": [
    {
      "type": "Action.InvokeAddInCommand",
      "title": "Invoke \"View Initialization Context\"",
      "addInId": "527104a1-f1a5-475a-9199-7a968161c870",
      "desktopCommandId": "showInitContext"
    }
  ]
}
```

## Passing initialization data to the add-in

The `Action.InvokeAddInCommand` action can also provide additional context to the add-in, allowing the add-in to do more than just simply activate. For example, the action could provide initial values for a form, or provide information that allows the add-in to "deep link" to a specific item in your back-end service.

In order to pass initialization data, include an `initializationContext` property in the `Action.InvokeAddInCommand` action. There is no set schema for the `initializationContext` property, you can include any valid JSON object.

For example, to extend the sample action from above, we could modify the action as follows:

```json
{
  "type": "AdaptiveCard",
  "version": "1.0",
  "body": [
    {
      "type": "TextBlock",
      "text": "Invoking an add-in command from an Actionable Message card",
      "size": "large"
    }
  ],
  "actions": [
    {
      "type": "Action.InvokeAddInCommand",
      "title": "Invoke \"View Initialization Context\"",
      "addInId": "527104a1-f1a5-475a-9199-7a968161c870",
      "desktopCommandId": "showInitContext",
      "initializationContext": {
        "property1": "Hello world",
        "property2": 5,
        "property3": true
      }
    }
  ]
}
```

## Receiving initialization data in the add-in

If your action passes initialization data, the add-in must be prepared to receive it. Add-ins can retrieve initialization data by calling the [Office.context.mailbox.item.getInitializationContextAsync](https://docs.microsoft.com/office/dev/add-ins/reference/objectmodel/preview-requirement-set/Office.context.mailbox.item) method. This should be done whenever the task pane opens or loads a new message.

```js
// Get the initialization context (if present)
Office.context.mailbox.item.getInitializationContextAsync(
  function(asyncResult) {
    if (asyncResult.status == Office.AsyncResultStatus.Succeeded) {
      if (asyncResult.value != null && asyncResult.value.length > 0) {
        // The value is a string, parse to an object
        var context = JSON.parse(asyncResult.value);
        // Do something with context
      } else {
        // Empty context, treat as no context
      }
    } else {
      if (asyncResult.error.code == 9020) {
        // GenericResponseError returned when there is
        // no context
        // Treat as no context
      } else {
        // Handle the error
      }
    }
  }
);
```

## Resources

- [Outlook-Add-In-Actionable-Message sample add-in](https://github.com/OfficeDev/Outlook-Add-In-Actionable-Message)