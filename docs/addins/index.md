---
title: Outlook add-ins overview | Microsoft Docs
description: Learn about Outlook add-ins
author: jasonjoh

ms.topic: article
ms.date: 04/11/2017
ms.author: jasonjoh
---

# Outlook add-ins overview

Outlook add-ins are integrations built by third parties into Outlook using the new web technologies based platform. Outlook add-ins have three key aspects:

- The same add-in and business logic works across desktop Outlook for Windows and Mac, web (Office 365 and Outlook.com), and mobile.
-  Outlook add-ins consist of a manifest, which describes how the add-in integrates into Outlook (for example, a button or a task pane), and JavaScript/HTML code, which makes up the UI and business logic of the add-in.
- Outlook add-ins can be acquired from the Office store or side-loaded by end-users or administrators.

Outlook add-ins are different from COM or VSTO add-ins, which are older integrations specific to Outlook running on Windows. Unlike COM add-ins, Outlook add-ins don't have any code physically installed to the user's device or Outlook client. For an Outlook add-in, Outlook reads the manifest and hooks up the specified controls in the UI, then loads the JavaScript and HTML. This all executes in the context of a browser in a sandbox.

The Outlook items that support mail add-ins include email messages, meeting requests, responses and cancellations, and appointments. Each mail add-in defines the context in which it is available, including the types of items and if the user is reading or composing an item.

To learn more about Outlook add-ins, see <a href="https://dev.office.com/docs/add-ins/outlook/outlook-add-ins?product=outlook" target="_blank">Outlook add-ins</a>.