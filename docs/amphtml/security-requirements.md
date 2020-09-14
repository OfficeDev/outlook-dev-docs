---
title: AMP for Email security requirements
description: AMP for Email security requirements
author: LezaMax
ms.topic: article
ms.technology: o365-amp
ms.author: humlezg
localization_priority: Priority
---
# Security requirements

[!INCLUDE [end-of-support](includes/end-of-support.md)]

To protect users' experience, privacy and security for AMP emails are subject to additional requirements and restrictions. Failure to comply with any of these requirements will result in a fallback behavior where the AMP for email MIME part is ignored and the HTML part is rendered instead.

## Sender Authentication

Emails containing AMP need to adhere to this criteria:

- Emails must pass authentication via:
  - [Domain Keys Identified Mail (DKIM) authentication](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail)
  - [Sender Policy Framework (SPF) authentication](https://en.wikipedia.org/wiki/Sender_Policy_Framework)
  - [Domain-based Message Authentication Reporting and Conformance (DMARC)](https://en.wikipedia.org/wiki/DMARC) (full pass)
- The top-level domain (TLD) of the SPF check or DKIM signature must match the TLD of your `From:` email address. For example, if you use `From: myservice@contoso.com`, the DKIM or SPF must be for `contoso.com` or `-.contoso.com`.
- Emails must come from a static email address, e.g. `myservice@contoso.com`.

## Sender Registration

In addition to passing all the checks outlined in the previous section, senders of AMP email need to register the static email address their emails originate from. See [sender registration](register-outlook.md) for more details.

## Best Practices

Emails that contain AMP must observe [best practices](best-practices.md). Failure to do so may result in revocation of your sender registration.
