---
title: Sending actionable email from dynamic addresses | Microsoft Docs
description: Learn how to enable sending actionable email messages from dynamic email addresses in your organization.
author: jasonjoh

ms.topic: article
ms.technology: o365-connectors
ms.date: 05/07/2018
ms.author: jasonjoh
---
# Sending actionable email from dynamic addresses

Actionable emails originate from a service. There are two major scenarios for actionable emails:

- Static senders: The sender is one of a predefined list of static service email addresses, such as `notification@contoso.com`.
- Dynamic senders: The sender is anyone in the service's organization.

All actionable emails are checked against a list of approved senders before being delivered to the intended recipient. If your service uses static addresses to send actionable email, you can register those addresses at the [actionable email developer dashboard](actionable-email-dev-dashboard.md). However, if your service needs to use dynamic addresses, there are additional requirements.

- You must [get approved](#get-approved-to-use-dynamic-senders) to use dynamic senders.
- You must [encode and sign your message card payloads](#encoding-and-signing-message-card-payloads) with a certificate, and you must share the public key for that certificate with Microsoft.

## Get approved to use dynamic senders

In order to get approved, contact us at [onboardoam@microsoft.com](mailto:onboardoam@microsoft.com). Once you are approved, you will be provided with an originator ID that you must include in your message card payloads.

## Encoding and signing message card payloads

When using dynamic senders, you must encode your message card payloads using the [JSON Web Token (JWT) format](https://tools.ietf.org/html/rfc7519) and sign the payload using the [JSON Web Signature (JWS) format](https://tools.ietf.org/html/rfc7515).

### Payload format

The payload of the encode card uses the following format.

| Field | Type | Description |
|-------|------|-------------|
| `sender` | String | The email address of the dynamic sender sending this particular card. |
| `originator` | UUID | The originator ID provided by Microsoft. |
| `messageCardSerialized` | String | The message card serialized into a string. |

#### Example payload

```json
{
  "sender": "john@contoso.com",
  "originator": "65c680ef-36a6-4a1b-b84c-a7b5c6198792",
  "messageCardSerialized": "{ \"@type\": \"MessageCard\", \"title\": \"Hello World\"  }"
}
```

### Signing the payload

When generating the JWT for the signed message card, you can use one of the following types of signing keys:

- RSA keys
- Self-signed X509 certificate
- X509 certificate issued by a certificate authority

You must provide the public key for whichever key type you use to Microsoft so that the signature can be validated.

### Inserting the signed card into email

The signed card is inserted into the HTML body of an email inside a `<script type="ld+json">` tag. The text inside the `script` tag uses the following format.

| Field | Type | Description |
|-------|------|-------------|
| `@context` | String | Must  be set to `http://schema.org/extensions`. |
| `@type` | String | Must be set to `SignedMessageCard`. |
| `signedMessageCard` | String | The generated JWT that contains the message card payload. |

#### Example script tag

```html
<script type="application/ld+json">
{
  "@context": "http://schema.org/extensions",
  "@type": "SignedMessageCard",
  "signedMessageCard": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzZW5kZXIiOiJqb2huQGNvbnRvc28uY29tIiwib3JpZ2luYXRvciI6IjY1YzY4MGVmL TM2YTYtNGExYi1iODRjLWE3YjVjNjE5ODc5MiIsIm1lc3NhZ2VDYXJkU2VyaWFsaXplZCI6InsgXCJAdHlwZVwiOiBcIk1lc3NhZ2VDYXJ kXCIsIFwidGl0bGVcIjogXCJIZWxsbyBXb3JsZFwifSJ9.3PQC2dfERAPTm_vT_eR_Du0_A4ntCLxZoOyKOz53gnkQhZsKSYdMuNBbl1PR 5B22GPZ33jJaEz1T5Vg2q6bLjGNUdFg0EY5vaZRxNuxT7HXvBwJqMjMD86vMHJgJf5hOYLPVoWeejC67ofIz5hIpfye3MV7oQGtdVMd46WgUHBMMTQP-ieVzQ_4ruXID87BepFgWCDa5q2htG_OfaP6kbZQhLpodfAiQU9QE6D0E9fxd2tgif5j4T1R8jygvs_wVHCCZNLYsluh3EaIA9IpMggnMtPAGTFdCVetWwjaeCQSNNVXzgl0qHs1QVobd09 TIboxUqN-KmTTf6iDtn-3w"
}
</script>
```

## Signature validation

During the delivery of the message, Microsoft validates the signature in the signed message card using the public key you provided. Microsoft also makes sure that the value of the `sender` field in the [payload](#payload-format) matches the sender of the email.