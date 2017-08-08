---
title: Outlook add-in requirements | Microsoft Docs
description: Learn about the client and server requirements to use Outlook add-ins.
author: jasonjoh

ms.topic: article
ms.technology: office-add-ins
ms.date: 08/08/2017
ms.author: jasonjoh
---

# Outlook add-in requirements

In order for Outlook add-ins to load and function properly, there are a number of requirements for both the servers and the clients.

## Client requirements

1. The client must be one of the supported hosts for Outlook add-ins. The following clients support add-ins:

    - Outlook 2013 and 2016 for Windows
    - Outlook 2016 for Mac
    - Outlook for iOS
    - Outlook on the web for Exchange 2016 and Office 365
    - Outlook Web Access for Exchange 2013
    - Outlook.com
1. The client must be connected to an Exchange server or Office 365 using a direct connection. When configuring the client, the user must choose an **Exchange**, **Office 365**, or **Outlook.com** account type. If the client is configured to connect with POP3 or IMAP, add-ins will not load.

## Mail server requirements

If the user is connected to Office 365 or Outlook.com, mail server requirements are all taken care of already. However, for users connected to on-premises installations of Exchange Server, the following requirements apply.

1. The server must be Exchange 2013 or later.
1. Exchange Web Services (EWS) must be enabled and must be exposed to the internet. Many add-ins require EWS to function properly.
1. The server must have a valid authentication certificate in order for the server to issue valid identity tokens. New installations of Exchange server include a default authentication certificate. For more information, see [Digital certificates and encryption in Exchange 2016](https://technet.microsoft.com/en-us/library/dd351044(v=exchg.160).aspx) and [Set-AuthConfig](https://technet.microsoft.com/en-us/library/jj215766(v=exchg.160).aspx).
1. In order to access add-ins from the [Office Store](https://store.office.com), the client access servers must be able to communicate with https://store.office.com. 

## Add-in server requirements

Add-in files (HTML, JavaScript, etc.) can be hosted on any web server platform desired. The only requirement is that the server must be configured to use HTTPS, and the SSL certificate must be trusted by the client.