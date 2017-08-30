---
title: Actionable messages via email developer dashboard guide | Microsoft Docs
description: Learn about the actionable message developer dashboard.
author: jasonjoh

ms.topic: article
ms.technology: office-365-connectors
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Register your service with the actionable email developer dashboard

To test and publish actionable messages from your service, you need to provide certain information to Microsoft to enable this functionality for emails from your service. The [developer dashboard](https://aka.ms/publishoam) helps you submit and track status of your submission via the web portal.

If you are a developer working with actionable messages via email, you will use the portal for the following cases:

- To test actionable messages from your service to your own mail box
- To publish actionable messages from your service so any email user in Office 365 using your service can receive these specially formatted messages
- To publish actionable message from your service so any email user within your organization using Office 365 can receive these specially formatted message (this is typically used for enabling actionable messages from a service that is specific to your organization, like a line-of-business app)

For all the above cases, you will be submitting certain details to Microsoft, which after being reviewed and approved, will enable actionable messages for your service.

> [!NOTE]
> You can easily try out actionable messages via email by sending email to self with the required markup without any intervention from Microsoft. This would typically be the first step you try out as you dip your toes into this capability.
> Check out [these samples](actionable-messages-via-email.md#sending-the-message) to send an actionable message to your mailbox.

## Dashboard sections

The developer dashboard is divided into a few logical sections you need to fill out based on the scope you’d like to request Microsoft to enable actionable message from your service.

### Details of your provider

In this section, you need to supply key details that will allow Office 365 to accept emails with markup from your service as well as URLs that can be invoked via the action buttons from those emails.

The key fields are:

- **Sender email address**: This is one or more static email addresses corresponding to the service that will send out emails with action markup. Example: `myservice@contoso.com`.
- **Target URLs**: This is one or more domains corresponding to URLs that will process the actions. Your target URL can correspond to the top level domain or the subdomain of the TLD. They need to be https enabled URLs. Example. `https://api.myservice.com`.

### Scope of your submission

In this section, you need to specify at what scope you want to enable actionable message for your service. The applicable scopes are:

- **My mailbox**: This will enable actionable emails from your service sent to your own mailbox.
- **Global**: This will enable actionable message from your service for any email user in Office 365.
- **My organization**: This will enable actionable message from your service to any Office 365 email user within your organization. This scope is typically used for enabling actionable messages from a service that is specific to your organization, like a line- of-business application internal to your organization.

Each of the above are independent steps. i.e. you can pick only one scope for each submission and will be subject to the approval process by Microsoft.

> [!NOTE]
> Remember, you can easily try out actionable messages by sending an email to self with the required markup without any intervention from Microsoft. This would typically be the first step to try out actionable messages.

### Contact info

In this section, you need to provide contact details, so we can reach out to you if we have further questions regarding your submission. All information provided must be valid and accurate.

### Publisher information

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide details about your service that will be sending Actionable Messages and related support information, so we can reach out to you or direct customers to your support site. This information will also be used as part of the approval process by Microsoft for your provider.

### Scenario details

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide details on the scenario for which users will consume actionable messages from your service and other relevant details. This is to help Microsoft determine that validity and usefulness of the solution provided by your service.

### Verification details

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide details for Microsoft to verify the actionable message and the corresponding actions that are invoked from the email sent by your provider/service.

Additionally, send a valid email coming from your production servers (or a server with similar DKIM/SPF/From:/Return-Path: headers) including the markup to amverify@microsoft.com. This procedure will enable Microsoft to determine that the solution complies with all the guidelines and requirements listed in Registration Criteria.

- Make sure that the markup is correct prior to sending the email.
- Office 365 removes all markup when forwarding an email. Do not forward the email but send it directly.

## Registration criteria

There are some things you need to keep in mind when you submit your solution for approval for **Global** scope since it can have broad impact to users in Office 365.

### Email sender quality guidelines

- Emails must be authenticated via DKIM or SPF.
- The top-level domain (TLD) of the SPF check or DKIM signature must match the TLD of your `From:` email address. For example, if you use `From: myservice@contoso.com` the DKIM or SPF must be for `contoso.com` or `subdomain.contoso.com`. 
- Emails must come from a static email address, e.g. `myservice@contoso.com`.
- Emails must follow the email sender guidelines.
  - See [Sending mail to Office 365](https://technet.microsoft.com/en-us/library/mt706217(v=exchg.150).aspx) for Office 365.
  - See [Policies, Practices, and Guidelines](https://mail.live.com/mail/policies.aspx) for Outlook.com.
  - See [M3AAWG Sender Best Practices](https://www.m3aawg.org/sites/default/files/document/M3AAWG_Senders_BCP_Ver3-2015-02.pdf) and [ReturnPath Sending Best Practices](https://help.returnpath.com/hc/en-us/articles/221634867-Sending-Best-Practices-PDF-) for industry guidelines.
- Consistent history of sending a high volume of mail from your domain (order of hundred emails a day minimum to Office 365) for a few weeks at least.
- A very low rate of spam complaints from users.
- High-fidelity, routine and simple actions available for your service should be used. For more complex interactions, `OpenURI` actions can be used.
- Actions should be used for transactional mail where a high interaction rate is expected. They should not be used on promotional bulk mail.

### Actions guidelines

- Label of the button needs to reflect clear action to be taken.
- `OpenURI` action must deep link into the specific page associated with the entity/information presented in the actionable message.
- Low failure rate and fast response for services handling action requests.
- Please see the [Card reference](card-reference.md) for additional guidelines on designing actionable messages.

## Approval of your submission

We will notify you on the email address you provided during your submission, so please ensure you provide the right contact information.
