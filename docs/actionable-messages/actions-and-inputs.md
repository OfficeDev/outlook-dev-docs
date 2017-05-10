---
title: Learn about the available action and input types in Outlook Actionable Messages | Microsoft Docs
description: Outlook Actionable Messages support calling an external service with optional user input and opening a URI. Learn more about the available types.
author: jasonjoh

ms.topic: article
ms.technology: office-365-connectors
ms.date: 05/09/2017
ms.author: jasonjoh
---

# Avaialable action and input types for Outlook Actionable Messages

Outlook Actionable Messages support two types of actions: calling an external service via HTTP POST, and opening a URI. When calling an external service, an actionable message can also contain a number of different input types to get information from the user.

### Calling an external service

The [`HttpPOST`](card-reference.md#httppost-action) action type calls an external service. It supports collecting input from the user and including that in the call.

#### Actions without input

There are many scenarios where the user is expected to confirm a pre-defined request. For example, marking a task as complete or approving access requests to documents. In these scenarios a simple click of a button can invoke an action.

#### Actions with input

Actionable messages can be used to support several scenarios with input. Some scenarios require specialized controls to support them. Here's some example of scenarios corresponding to the controls that are available with actionable messages.

- Date input: The date control enables many scenarios where a date input is desired. For example, it can be used for setting due dates for tasks/projects.
- Choosing from a list: The list box control enables scenarios that require users to pick a value from a pre-defined list. For example, setting a stage for a lead in a CRM system, setting the status of an issue in an incident tracking system.
- Text input: The text box control enables scenarios that require the user to enter text, such as commenting on an assigned task.

### Opening a URI

The [`OpenUri`](card-reference.md#openuri-action) action type opens a URI, either in a browser or as a deep link into an app. This is typically used to view the content that was received in the actionable message directly in the service.

### Next steps

To learn more about designing cards and actions, see the [Card reference](card-reference.md).