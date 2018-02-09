---
title: Get started with Outlook Pay in .NET | Microsoft Docs
description: Follow this step-by-step walkthrough to implement an example Outlook Pay solution in .NET
author: jasonjoh

ms.topic: get-started-article
ms.technology: o365-connectors
ms.date: 02/08/2018
ms.author: jasonjoh
---

# Get started with Outlook Pay in an ASP.NET MVC app

The purpose of this guide is to walk through the process of implementing a simple Outlook Pay solution in an ASP.NET MVC C# app. The source code in this [repository](#) is what you should end up with if you follow the steps outlined here.

This tutorial will use the [Microsoft Authentication Library (MSAL)](https://www.nuget.org/packages/Microsoft.Identity.Client) to make OAuth2 calls and the [Microsoft Graph Client Library](https://www.nuget.org/packages/Microsoft.Graph/) to call the Mail API.

## Prerequisites

- Visual Studio 2017. If you don’t already have Visual Studio 2017 installed, you can download and use the free [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/). Make sure that you enable **Azure development** during the Visual Studio setup.
- An active Azure account. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin. We'll be using an Azure web app to host the solution.