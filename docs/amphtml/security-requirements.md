---
title: Test AMP Email
description: How to test AMP email in your mailbox
author: LezaMax
ms.topic: article
ms.technology: o365-amp
ms.author: humlezg
localization_priority: Priority
---
# Security Requirements

To protect users privacy and security AMP emails are subject to additional security requirements and restrictions. Failure to comply with any of these requirements will result in a fallback behavior where the AMP for email MIME part is ignored and the HTML part is rendered instead.

## Sender Authentication
To verify the authenticity of AMP email senders, emails containing AMP need to pass the following checks:

- [Domain Keys Identified Mail (DKIM) authentication](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) 
- [Sender Policy Framework (SPF) authentication](https://en.wikipedia.org/wiki/Sender_Policy_Framework)
- [Domain-based Message Authentication
Reporting and Conformance (DMARC)](https://en.wikipedia.org/wiki/DMARC); full pass. 

## Sender Registration
Senders must [register](register-outlook.md) with Microsoft for their AMP emails to work in end user mailboxes. The from header address of any emails sent must be an exact match to the address used during registration. 