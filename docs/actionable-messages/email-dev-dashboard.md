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

<!-- cSpell:ignore jasonjoh vermaanimesh AQAB AAWG contoso -->

# Register your service with the actionable email developer dashboard

[!INCLUDE [legacy-token-deprecation](../includes/actionable-messages/legacy-token-deprecation.md)]

To test and publish actionable messages from your service, provide certain information to Microsoft to enable this functionality for emails from your service. The [developer dashboard](https://aka.ms/ActionableMessagesPortal) helps you submit and track the status of your submission through the web portal.

> [!NOTE]
> You can easily try out actionable messages via email by sending an email to yourself with the required markup without any intervention from Microsoft. This step is typically the first step you try as you dip your toes into this capability.
> Check out [these samples](send-via-email.md#sending-the-message) to send an actionable message to your mailbox, or use the [Actionable Message Designer](https://amdesigner.azurewebsites.net/) to send an actionable message to yourself.

If you're a developer working with actionable messages via email, use the portal for the following cases:

- To test actionable messages from your service to your own mailbox
- To publish actionable messages from your service so any email user within your organization using Microsoft 365 can receive these specially formatted messages. This option typically enables actionable messages from a service that is specific to your organization, like a line-of-business app.
- To publish actionable messages from your service so any email user in Microsoft 365 using your service can receive these specially formatted messages

For all these cases, submit details to Microsoft. After review and approval, Microsoft enables actionable messages for your service.

## Dashboard sections

The developer dashboard is divided into a few logical sections you need to fill out based on the scope you'd like to request Microsoft to enable actionable messages from your service.

### Details of your provider

In this section, provide key details that allow Microsoft 365 to accept emails with markup from your service, as well as URLs that the action buttons in those emails can invoke.

The key fields are:

- **Sender email address**: Enter one or more static email addresses that correspond to the service that sends emails with action markup. For example, `myservice@contoso.com`.
- **Target URLs**: Enter one or more domains that correspond to URLs that process the actions. Your target URL can correspond to the top-level domain or the subdomain of the TLD. They need to be HTTPS-enabled URLs. For example, `https://api.myservice.com`.
- **Public Key**: If you plan to send actionable messages as Signed Card, specify the public key that corresponds to the private key you use for signing the card. The format for this field is an [RSAKeyValue element](https://www.w3.org/TR/xmldsig-core/#sec-RSAKeyValue).

  ```xml
  <RSAKeyValue>
    <Modulus>xA7SEU+e0yQ...</Modulus>
    <Exponent>AQAB</Exponent>
  </RSAKeyValue>
  ```

You can generate and export an RSA key pair in the correct format by using PowerShell (7.3 or later).

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
> After your submission is approved, it might take some time for the changes to take effect. If you encounter the following error when sending signed cards, and you're sure that your payload is correct, try again after a few hours.
>
> ```text
> Adaptive card signature validation failed - Failed to validate signature
> ```

### Scope of your submission

Specify the scope where you want to enable actionable message for your service. The applicable scopes are:

- **Test Users**: This scope enables actionable emails from your service to some of the Microsoft 365 email users in your organization. Use this scope to test actionable messages integration with a few test users that you specify.
- **Organization**: This scope enables actionable message from your service to any Microsoft 365 email user within your organization. Use this scope to enable actionable messages from a service that is specific to your organization, like a line-of-business application.
- **Global**: This scope enables actionable message from your service for any email user in Microsoft 365.

Each scope is independent. You can submit only one scope per request and must obtain Microsoft approval.

> [!NOTE]
> You can easily try out actionable messages by sending an email to yourself with the required markup without any intervention from Microsoft. Use the [Actionable Message Designer](https://amdesigner.azurewebsites.net/) to send to yourself without writing any code. This step typically is the first step to try out actionable messages.

#### Self-service registration

Self-service registration is available for the following scopes.

- **Test Users**: The registration request auto-approves for your test users you specify. This registration enables actionable emails from your service sent to test users.
- **Organization**: This registration request routes to your organization's [Exchange or Global administrator](/microsoft-365/admin/add-users/about-admin-roles) for review and approval.

Once the submission is approved, whether auto-approved or by your administrator, it can take up to 24 hours for the registration to take effect.

After 24 hours, verify the registration is active by sending an actionable message from your service to your mailbox or specified test users (for **Test Users** scope), or any user in your organization (for **Organization** scope). If 24 hours have passed and the registration is still not active, contact us by using the feedback link at the top of the registration dashboard labeled **Registration not working?**.

### Test user email addresses

This section is only applicable when your scope of submission to enable actionable messages is **Test Users**.

Provide a list of Microsoft 365 email users in your organization, separated by a semicolon (`;`). This list helps you test your actionable messages integration with a few users before creating an **Organization** or **Global** scope submission.

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

Additionally, send a valid email coming from your production servers (or a server with similar DKIM/SPF/From:/Return-Path: headers) including the markup to [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com). This procedure will enable Microsoft to determine that the solution complies with all the guidelines and requirements listed in Registration Criteria.

- Make sure that the markup is correct prior to sending the email.
- Microsoft 365 removes all markup when forwarding an email. Do not forward the email but send it directly.

## Registration criteria

Keep these criteria in mind when you submit your solution for approval for **Global** scope, since it can have a broad impact on users in Microsoft 365.

### Email sender quality guidelines

- Authenticate emails through DKIM or SPF.
- The top-level domain (TLD) of the SPF check or DKIM signature must match the TLD of your `From:` email address. For example, if you use `From: myservice@contoso.com`, the DKIM or SPF must be for `contoso.com` or `-.contoso.com`.
- Send emails from a static email address, such as `myservice@contoso.com`.
- Follow the email sender guidelines.
  - For Microsoft 365, see [Microsoft 365 services for external email senders](/defender-office-365/external-senders-microsoft-365-services).
  - For Outlook.com, see [Policies, Practices, and Guidelines](https://sendersupport.olc.protection.outlook.com/pm/policies.aspx).
  - For industry guidelines, see [M3AAWG Sender Best Practices](https://www.m3aawg.org/sites/default/files/doc_files/M3AAWG_Senders_BCP_Ver3-2015-02.pdf) and [ReturnPath Sending Best Practices](https://help.returnpath.com/hc/articles/221634867-Sending-Best-Practices-PDF-).
- Maintain a consistent history of sending a high volume of mail from your domain (an order of hundred emails a day minimum to Microsoft 365) for at least a few weeks.
- Maintain a very low rate of spam complaints from users.
- Use high-fidelity, routine, and simple actions for your service. For more complex interactions, use `Action.OpenUrl` actions.
- Use actions for transactional mail where a high interaction rate is expected. Don't use them on promotional bulk mail.

### Actions guidelines

- The label of the button needs to clearly reflect the action to take.
- `Action.OpenUrl` action must deep link into the specific page associated with the entity or information presented in the actionable message.
- Maintain a low failure rate and fast response for services handling action requests.
- For additional guidelines on designing actionable messages, see [Designing Outlook actionable message cards with the Adaptive Card format](adaptive-card.md).

## Approval of your submission

You'll receive a notification at the email address you provide during your submission, so make sure you provide the right contact information.
