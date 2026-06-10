---
title: Actionable messages via email developer dashboard guide
description: The developer dashboard helps you submit and track status of your submission via the web portal.
author: jasonjoh
ms.topic: article
ms.service: outlook
ms.date: 06/10/2026
ms.author: vermaanimesh
ms.localizationpriority: high
ms.subservice: o365-connectors
---

# Register your service with the actionable email developer dashboard

[!INCLUDE [legacy-token-deprecation](../includes/actionable-messages/legacy-token-deprecation.md)]

To test and publish actionable messages from your service, you need to provide certain information to Microsoft to enable this functionality for emails from your service. The [developer dashboard](https://aka.ms/ActionableMessagesPortal) helps you submit and track status of your submission via the web portal.

> [!NOTE]
> You can easily try out actionable messages via email by sending email to yourself with the required markup without any intervention from Microsoft. This would typically be the first step you try out as you dip your toes into this capability.
> Check out [these samples](send-via-email.md#sending-the-message) to send an actionable message to your mailbox, or use the [Actionable Message Designer](https://amdesigner.azurewebsites.net/) to send an actionable message to yourself.

If you are a developer working with actionable messages via email, you will use the portal for the following cases:

- To test actionable messages from your service to your own mail box
- To publish actionable messages from your service so any email user within your organization using Office 365 can receive these specially formatted messages (this is typically used for enabling actionable messages from a service that is specific to your organization, like a line-of-business app)
- To publish actionable messages from your service so any email user in Office 365 using your service can receive these specially formatted messages

For all these cases, you will submit details to Microsoft. After review and approval, actionable messages will be enabled for your service.

## Dashboard sections

The developer dashboard is divided into a few logical sections you need to fill out based on the scope you'd like to request Microsoft to enable actionable messages from your service.

### Details of your provider

In this section, you need to supply key details that will allow Office 365 to accept emails with markup from your service as well as URLs that can be invoked via the action buttons from those emails.

The key fields are:

- **Sender email address**: This is one or more static email addresses corresponding to the service that will send out emails with action markup. Example: `myservice@contoso.com`.
- **Target URLs**: This is one or more domains corresponding to URLs that will process the actions. Your target URL can correspond to the top level domain or the sub-domain of the TLD. They need to be https enabled URLs. Example: `https://api.myservice.com`.
- **Public Key**: If you plan to send actionable messages as Signed Card, then you need to specify the public key corresponding to the private key you will use for signing the card. The format for this field is an [RSAKeyValue element](https://www.w3.org/TR/xmldsig-core/#sec-RSAKeyValue).

  ```xml
  <RSAKeyValue>
    <Modulus>xA7SEU+e0yQ...</Modulus>
    <Exponent>AQAB</Exponent>
  </RSAKeyValue>
  ```

A RSA key pair can be generated and exported in the correct format using PowerShell (7.3 or later):

``` powershell
# Generate a key pair:
$rsa = [System.Security.Cryptography.RSA]::Create();

# Private Key, ensure this is saved securely and kept secret:
$rsa.ToXmlString($true)

## Public Key, copy output to dashboard:
$rsa.ToXmlString($false)
```

For an example of how to get public key XML from a .cert file, see [PublicKey class](/dotnet/api/system.security.cryptography.x509certificates.publickey#examples).

> [!NOTE]
> Once your submission is approved, it may take some time to take effect. If you encounter the error below when sending signed cards, and you're sure that your payload is correct, please try again after a few hours.
>
> ```text
> Adaptive card signature validation failed - Failed to validate signature
> ```

### Scope of your submission

In this section, you need to specify at what scope you want to enable actionable message for your service. The applicable scopes are:

- **Test Users**: This enables actionable emails from your service to some of the O365 email users in your organization. This scope is generally used for testing actionable messages integration with few test users that you have specified.
- **Organization**: This enables actionable message from your service to any Microsoft 365 email user within your organization. This scope is typically used for enabling actionable messages from a service that is specific to your organization, like a line-of-business application.
- **Global**: This enables actionable message from your service for any email user in Office 365.

Each scope is independent — you can submit only one scope per request and must obtain Microsoft approval.

> [!NOTE]
> Remember, you can easily try out actionable messages by sending an email to yourself with the required markup without any intervention from Microsoft. You can use the [Actionable Message Designer](https://amdesigner.azurewebsites.net/) to send to yourself without writing any code. This would typically be the first step to try out actionable messages.

#### Self-service registration

Self-service registration is available for the following scopes.

- **Test Users**: The registration request is auto-approved for your test users you specify. This will enable actionable emails from your service sent to test users.
- **Organization**: This registration request is routed to your organization's [Exchange or Global administrator](https://learn.microsoft.com/en-us/microsoft-365/admin/add-users/about-admin-roles?view=o365-worldwide) for review and approval.

Once the submission is approved, whether auto-approved or by your administrator, it will take up to 24 hours for the registration to take effect.

After 24 hours, verify the registration is active by sending an actionable message from your service to your mailbox or specified test users (for **Test Users** scope), or any user in your organization (for **Organization** scope). If 24 hours have passed and the registration is still not active, contact us using the feedback link at the top of the registration dashboard labeled **Registration not working?**.

### Test user email addresses

This section is only applicable when your scope of submission to enable actionable messages is **Test Users**.

In this section, provide a list of Microsoft 365 email users in your organization, separated by a semicolon (`;`). This will help you test your actionable messages integration with a few users before creating an **Organization** or **Global** scope submission.


### Contact info

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide contact details, so we can reach out to you if we have further questions regarding your submission. All information provided must be valid and accurate.

### Publisher information

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide details about your service that will be sending Actionable Messages and related support information, so we can reach out to you or direct customers to your support site. This information will also be used as part of the approval process by Microsoft for your provider.

### Scenario details

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide details about the scenario in which users will use actionable messages from your service. This helps Microsoft determine the validity and usefulness of your solution.

### Verification details

This section is only applicable when your scope of submission to enable actionable messages is **Global**.

In this section, you need to provide details for Microsoft to verify the actionable message and the corresponding actions that are invoked from the email sent by your provider/service.

Additionally, send a valid email coming from your production servers (or a server with similar DKIM/SPF/From:/Return-Path: headers) including the markup to onboardoam@microsoft.com. This procedure will enable Microsoft to determine that the solution complies with all the guidelines and requirements listed in Registration Criteria.

- Make sure that the markup is correct prior to sending the email.
- Office 365 removes all markup when forwarding an email. Do not forward the email but send it directly.

## Registration criteria

There are some things you need to keep in mind when you submit your solution for approval for **Global** scope since it can have broad impact to users in Office 365.

### Email sender quality guidelines

- Emails must be authenticated via DKIM or SPF.
- The top-level domain (TLD) of the SPF check or DKIM signature must match the TLD of your `From:` email address. For example, if you use `From: myservice@contoso.com` the DKIM or SPF must be for `contoso.com` or `-.contoso.com`.
- Emails must come from a static email address, e.g. `myservice@contoso.com`.
- Emails must follow the email sender guidelines.
  - See [Sending mail to Office 365](/office365/SecurityCompliance/sending-mail-to-office-365) for Office 365.
  - See [Policies, Practices, and Guidelines](https://sendersupport.olc.protection.outlook.com/pm/policies.aspx) for Outlook.com.
  - See [M3AAWG Sender Best Practices](https://www.m3aawg.org/sites/default/files/doc_files/M3AAWG_Senders_BCP_Ver3-2015-02.pdf) and [ReturnPath Sending Best Practices](https://help.returnpath.com/hc/articles/221634867-Sending-Best-Practices-PDF-) for industry guidelines.
- Consistent history of sending a high volume of mail from your domain (order of hundred emails a day minimum to Office 365) for a few weeks at least.
- A very low rate of spam complaints from users.
- Use high-fidelity, routine, and simple actions for your service. For more complex interactions, use `Action.OpenUrl` actions.
- Use actions for transactional mail where a high interaction rate is expected. Do not use them on promotional bulk mail.

### Actions guidelines

- Label of the button needs to reflect clear action to be taken.
- `Action.OpenUrl` action must deep link into the specific page associated with the entity/information presented in the actionable message.
- Low failure rate and fast response for services handling action requests.
- Please see [Designing Outlook actionable message cards with the Adaptive Card format](adaptive-card.md) for additional guidelines on designing actionable messages.

## Approval of your submission

We will notify you on the email address you provided during your submission, so please ensure you provide the right contact information.
