---
title: Payments in Outlook webhook reference | Microsoft Docs
description: Learn about the payload fields for requests sent to payment webhooks by Outlook
author: jasonjoh

ms.topic: reference
ms.technology: o365-connectors
ms.date: 05/07/2018
ms.author: jasonjoh
---
# Payments in Outlook webhook reference

## PaymentRequest webhook

The payment request webhook is called when the user starts the payment experience by choosing the "Pay" button on your message card. Additionally, if you request shipping information, the webhook may also be called multiple times during the payment experience, allowing you to update the payment information presented to the user based on selections they make.

The format of the payload sent to your webhook and the payload you must send back in the response is specified in the following section.

### Payment request webhook payload

The payload format is based on the [W3C Payment Request API specification](http://www.w3.org/TR/payment-request).

| Field | Type| Description |
|-------|-----|-------------|
| `id` | String | A developer-assigned identifier for this payment request. |
| `methodData` | Array of [PaymentMethodData](http://www.w3.org/TR/payment-request/#paymentmethoddata-dictionary) (from W3C Payment Request API spec) | Specifies data relevant to the current request. |
| `options` | [PaymentOptions](http://www.w3.org/TR/payment-request/#paymentoptions-dictionary) (from W3C Payment Request API spec) | Specifies what data your webhook needs the Payment service to provide. |
| `details` | [PaymentDetailsInit](http://www.w3.org/TR/payment-request/#paymentdetailsinit-dictionary) or [PaymentDetailsUpdate](http://www.w3.org/TR/payment-request/#paymentdetailsinit-dictionary) (from W3C Payment Request API spec) | Specifies the details of the invoice. The information in this object controls how the invoice is displayed to the user in Outlook. |
| `shippingAddress` | [AddressInit](http://www.w3.org/TR/payment-request/#addressinit-dictionary) (from W3C Payment Request API spec) | Specifies the shipping address provided by the user. |
| `shippingOption` | String | Specifies which shipping option the user selected. |

#### methodData

The `methodData` object contains information relevant to the payment request.

| Field | Type| Description |
|-------|-----|-------------|
| `supportedMethods` | String | MUST be set to `https://pay.microsoft.com/microsoftpay`. |
| `data` | [Data](#data) | Contains the data relevant to the request. |

##### Data

| Field | Type| Description |
|-------|-----|-------------|
| `supportedNetworks` | Array of String | An array indicating the payment networks you accept. Valid values are `visa`, `mastercard`, `amex`, `discover`, `diners`, and `jcb`. |
| `supportedTypes` | Array of String | An array indicating the types of payments you accept. Valid values are `credit`, `debit`, and `prepaid`. |
| `productContext` | Object | A free-form object defined by the developer. The `productContext` is used to store contextual information needed for the payment process. |
| `event` | String | Indicates the event that triggered the current POST to the webhook. Valid values are `loadentity`, `shippingaddresschange`, and `shippingoptionchange`. |
| `entity` | For invoicing - Schema.org [Invoice](http://schema.org/Invoice) | Represents the invoice. |
| `mode` | String | If present and set to `TEST`, enables test mode. In this mode, the recipient has access to pre-configured test cards in the payment experience. Payment tokens in test mode are test tokens as defined in [Stripe Test Cards and numbers](https://stripe.com/docs/testing#cards). In production, this property SHOULD be omitted. |

###### Example

```json
{
  "supportedNetworks": [
    "visa",
    "mastercard"
  ],
  "supportedTypes": [
    "credit"
  ],
  "productContext": {
    "invoiceId": "12345"
  },
  "event": "loadentity",
  "entity": {
    "@type": "Invoice",
    "@context": "http://schema.org",
    "identifier": "12345",
    "url": "https://contoso.com/invoice",
    "broker": {
      "@type": "LocalBusiness",
      "name": "Contoso"
    },
    "paymentDueDate": "2019-01-31T00:00:00",
    "paymentStatus": "PaymentDue",
    "totalPaymentDue": {
      "@type": "PriceSpecification",
      "price": 10,
      "priceCurrency": "USD"
    }
  },
  "mode": "TEST"
}
```

### Handling payment request webhook POSTs

When your webhook receives a POST, you should do the following:

1. Validate the bearer token in the `Authorization` header to ensure that the request is coming from Microsoft. See [Securing your webhooks](#securing-your-webhooks) for details.
1. Check the `event` property of the [Data](#data) object to determine what UI event triggered the POST.
    1. If `event` is set to `loadentity`, follow the steps in [Load the invoice](#load-the-invoice).
    1. If `event` is set to `shippingaddresschange`, follow the steps in [Handle shipping address change](#handle-shipping-address-change).
    1. If `event` is set to `shippingoptionchange`, follow the steps in [Handle shipping option change](#handle-shipping-option-change).
1. Return a `200 OK` response with the [Payment request webhook payload](#payment-request-webhook-payload) specified by the previous step in the body.

#### Load the invoice

When the `event` property in the incoming request is set to `loadentity` your webhook should load the invoice to provide details about the charges on the invoice to the Payments service. Your webhook will also indicate what information is required by your service regarding the payer. This event is triggered when the user clicks the **Review and Pay** button in Outlook.

The following is an example of the incoming request sent by the Payments service.

```json
{
  "methodData": [
    {
      "supportedMethods": "https://pay.microsoft.com/microsoftpay",
      "data": {
        "productContext": {
          "invoiceId": "12345"
        },
        "event": "loadentity"
      }
    }
  ]
}
```

The information in the `productContext` property corresponds to the `productContext` property set on the payment card message sent to the recipient. It should minimally contain the information required by your service to locate the corresponding invoice.

After loading the invoice, your webhook should send the following response payload:

- Include the `data` property with information about accepted payment types and networks and an Invoice entity.
- Include the `options` property to indicate what information you need about the payer and if you want the Payments service to ask for a shipping address and option.
- Include the `details` property to provide display details about the invoice.

The following is an example of a response to a `loadentity` event.

```json
{
  "methodData": [
    {
      "supportedMethods": "https://pay.microsoft.com/microsoftpay",
      "data": {
        "event": "loadentity",
        "productContext": {
          "invoiceId": "12345"
        },
        "supportedNetworks": [
          "visa",
          "mastercard"
        ],
        "supportedTypes": [
          "credit"
        ],
        "entity": {
          "@type": "Invoice",
          "@context": "http://schema.org",
          "identifier": "12345",
          "url": "https://contoso.com/invoice",
          "broker": {
            "@type": "LocalBusiness",
            "name": "Contoso"
          },
          "paymentDueDate": "2019-01-31T00:00:00",
          "paymentStatus": "PaymentDue",
          "totalPaymentDue": {
            "@type": "PriceSpecification",
            "price": 10,
            "priceCurrency": "USD"
          }
        },
        "mode": "TEST"
      }
    }
  ],
  "options": {
    "requestPayerName": true,
    "requestPayerEmail": true,
    "requestPayerPhone": true,
    "requestShipping": true
  },
  "details": {
    "id": "12345",
    "total": {
      "label": "Total Due",
      "amount": {
        "currency": "USD",
        "value": "10.00"
      },
      "pending": false
    },
    "displayItems": [
      {
        "label": "Large Widget",
        "amount": {
          "currency": "USD",
          "value": "7.00"
        },
        "pending": false
      },
      {
        "label": "Small Widget",
        "amount": {
          "currency": "USD",
          "value": "3.00"
        },
        "pending": false
      }
    ]
  }
}
```

#### Handle shipping address change

When the `event` property in the incoming request is set to `shippingaddresschange` your webhook should check the shipping address selected and return any applicable shipping options. This event is triggered when the user selects or changes a shipping address in Outlook.

> [!NOTE]
> Your webhook will only receive this event if you set the `requestShipping` property in the `PaymentOptions` object to `true` in your response to the `loadentity` event for this invoice.

The payload of the incoming request corresponds to the response sent by your webhook to the `loadentity` event, with the following key additions:

- The `event` property is set to `shippingaddresschange`.
- The `shippingAddress` property contains the address provided by the recipient.

The following is an example of the incoming request sent by the Payments service.

```json
{
  "id": "12345",
  "methodData": [
    {
      "supportedMethods": "https://pay.microsoft.com/microsoftpay",
      "data": {
        "supportedNetworks": [
          "visa",
          "mastercard"
        ],
        "supportedTypes": [
          "credit"
        ],
        "productContext": {
          "invoiceId": "12345"
        },
        "event": "shippingaddresschange",
        "entity": {
          "@type": "Invoice",
          "@context": "http://schema.org",
          "identifier": "12345",
          "url": "https://contoso.com/invoice",
          "broker": {
            "@type": "LocalBusiness",
            "name": "Contoso"
          },
          "paymentDueDate": "2019-01-31T00:00:00",
          "paymentStatus": "PaymentDue",
          "totalPaymentDue": {
            "@type": "PriceSpecification",
            "price": 10,
            "priceCurrency": "USD"
          }
        },
        "mode": "TEST"
      }
    }
  ],
  "options": {
    "requestPayerName": true,
    "requestPayerEmail": true,
    "requestPayerPhone": true,
    "requestShipping": true,
    "shippingType": "shipping"
  },
  "details": {
    "id": "12345",
    "total": {
      "label": "Total Due",
      "amount": {
        "currency": "USD",
        "value": "10.00",
        "currencySystem": null
      },
      "pending": false
    },
    "displayItems": [
      {
        "label": "Large Widget",
        "amount": {
          "currency": "USD",
          "value": "7.00",
          "currencySystem": null
        },
        "pending": false
      },
      {
        "label": "Small Widget",
        "amount": {
          "currency": "USD",
          "value": "3.00",
          "currencySystem": null
        },
        "pending": false
      }
    ]
  },
  "shippingAddress": {
    "country": "US",
    "addressLine": [
      "One Microsoft Way"
    ],
    "region": "WA",
    "city": "Redmond",
    "dependantLocality": null,
    "postalCode": "98052",
    "sortingCode": "",
    "languageCode": "",
    "organization": "Microsoft",
    "recipient": "Patti Fernandez",
    "phone": "4255551212"
  }
}
```

Your webhook should modify the incoming request payload to set the `shippingOptions` property to specify the available shipping options for the provided address.

The following is an example of a response to a `shippingaddresschange` event. In this example, the webhook is specifying two shipping options: regular shipping for free, or priority shipping for $3.00.

```json
{
  "id": "12345",
  "methodData": [
    {
      "supportedMethods": "https://pay.microsoft.com/microsoftpay",
      "data": {
        "supportedNetworks": [
          "visa",
          "mastercard"
        ],
        "supportedTypes": [
          "credit"
        ],
        "productContext": {
          "invoiceId": "12345"
        },
        "event": "shippingaddresschange",
        "entity": {
          "@type": "Invoice",
          "@context": "http://schema.org",
          "identifier": "12345",
          "url": "https://contoso.com/invoice",
          "broker": {
            "@type": "LocalBusiness",
            "name": "Contoso"
          },
          "paymentDueDate": "2019-01-31T00:00:00",
          "paymentStatus": "PaymentDue",
          "totalPaymentDue": {
            "@type": "PriceSpecification",
            "price": 10,
            "priceCurrency": "USD"
          }
        },
        "mode": "TEST"
      }
    }
  ],
  "options": {
    "requestPayerName": true,
    "requestPayerEmail": true,
    "requestPayerPhone": true,
    "requestShipping": true,
    "shippingType": "shipping"
  },
  "details": {
    "id": "12345",
    "total": {
      "label": "Total Due",
      "amount": {
        "currency": "USD",
        "value": "10.00",
        "currencySystem": null
      },
      "pending": false
    },
    "displayItems": [
      {
        "label": "Large Widget",
        "amount": {
          "currency": "USD",
          "value": "7.00",
          "currencySystem": null
        },
        "pending": false
      },
      {
        "label": "Small Widget",
        "amount": {
          "currency": "USD",
          "value": "3.00",
          "currencySystem": null
        },
        "pending": false
      }
    ],
    "shippingOptions": [
      {
        "id": "norush",
        "label": "Regular Shipping",
        "amount": {
          "currency": "USD",
          "value": "0.00"
        }
      },
      {
        "id": "priority",
        "label": "Priority Shipping",
        "amount": {
          "currency": "USD",
          "value": "3.00"
        }
      }
    ]
  },
  "shippingAddress": {
    "country": "US",
    "addressLine": [
      "One Microsoft Way"
    ],
    "region": "WA",
    "city": "Redmond",
    "dependantLocality": null,
    "postalCode": "98052",
    "sortingCode": "",
    "languageCode": "",
    "organization": "Microsoft",
    "recipient": "Patti Fernandez",
    "phone": "4255551212"
  }
}
```

#### Handle shipping option change

When the `event` property in the incoming request is set to `shippingoptionchange` your webhook should check which shipping option was selected by the user and update the invoice accordingly. This event is triggered when the user selects or changes the shipping option in Outlook.

> [!NOTE]
> Your webhook will only receive this event if you set the `requestShipping` property in the `PaymentOptions` object to `true` in your response to the `loadentity` event for this invoice.

The payload of the incoming request corresponds to the response sent by your webhook to the `loadentity` event, with the following key additions:

- The `event` property is set to `shippingoptionchange`.
- The `shippingOption` property contains the `id` of the shipping option the recipient selected.

The following is an example of the incoming request sent by the Payments service. From the previous example response, the available shipping options were regular (`norush`) and priority (`priority`). In this example, the user selected priority shipping.

```json
{
  "id": "12345",
  "methodData": [
    {
      "supportedMethods": "https://pay.microsoft.com/microsoftpay",
      "data": {
        "supportedNetworks": [
          "visa",
          "mastercard"
        ],
        "supportedTypes": [
          "credit"
        ],
        "productContext": {
          "invoiceId": "12345"
        },
        "event": "shippingaddresschange",
        "entity": {
          "@type": "Invoice",
          "@context": "http://schema.org",
          "identifier": "12345",
          "url": "https://contoso.com/invoice",
          "broker": {
            "@type": "LocalBusiness",
            "name": "Contoso"
          },
          "paymentDueDate": "2019-01-31T00:00:00",
          "paymentStatus": "PaymentDue",
          "totalPaymentDue": {
            "@type": "PriceSpecification",
            "price": 10,
            "priceCurrency": "USD"
          }
        },
        "mode": "TEST"
      }
    }
  ],
  "options": {
    "requestPayerName": true,
    "requestPayerEmail": true,
    "requestPayerPhone": true,
    "requestShipping": true,
    "shippingType": "shipping"
  },
  "details": {
    "id": "12345",
    "total": {
      "label": "Total Due",
      "amount": {
        "currency": "USD",
        "value": "10.00",
        "currencySystem": null
      },
      "pending": false
    },
    "displayItems": [
      {
        "label": "Large Widget",
        "amount": {
          "currency": "USD",
          "value": "7.00",
          "currencySystem": null
        },
        "pending": false
      },
      {
        "label": "Small Widget",
        "amount": {
          "currency": "USD",
          "value": "3.00",
          "currencySystem": null
        },
        "pending": false
      }
    ]
  },
  "shippingAddress": {
    "country": "US",
    "addressLine": [
      "One Microsoft Way"
    ],
    "region": "WA",
    "city": "Redmond",
    "dependantLocality": null,
    "postalCode": "98052",
    "sortingCode": "",
    "languageCode": "",
    "organization": "Microsoft",
    "recipient": "Patti Fernandez",
    "phone": "4255551212"
  },
  "shippingOption": "priority"
}
```

Your webhook should modify the incoming request payload to update the `details` property as appropriate. For example, if the user selects a shipping option that incurs an additional cost, your webhook might adjust the value in the `total` property to add the shipping cost and add or update a shipping line item in the `displayItems` property.

The following is an example of a response to a `shippingoptionchange` event.

```json
{
  "methodData": [
    {
      "supportedMethods": "https://pay.microsoft.com/microsoftpay",
      "data": {
        "event": "shippingoptionchange",
        "productContext": {
          "invoiceId": "12345"
        },
        "supportedNetworks": [
          "visa",
          "mastercard",
          "amex"
        ],
        "supportedTypes": [
          "credit"
        ],
        "entity": {
          "@type": "Invoice",
          "@context": "http://schema.org",
          "identifier": "12345",
          "url": "https://contoso.com/invoice",
          "broker": {
            "@type": "LocalBusiness",
            "name": "Contoso"
          },
          "paymentDueDate": "2019-01-31T00:00:00",
          "paymentStatus": "PaymentDue",
          "totalPaymentDue": {
            "@type": "PriceSpecification",
            "price": 10,
            "priceCurrency": "USD"
          }
        },
        "mode": "TEST"
      }
    }
  ],
  "options": {
    "requestPayerName": true,
    "requestPayerEmail": true,
    "requestPayerPhone": true,
    "requestShipping": true,
    "shippingType": "shipping"
  },
  "details": {
    "id": "12345",
    "total": {
      "label": "Total Due",
      "amount": {
        "currency": "USD",
        "value": "13.00"
      },
      "pending": false
    },
    "displayItems": [
      {
        "label": "Large Widget",
        "amount": {
          "currency": "USD",
          "value": "7.00"
        },
        "pending": false
      },
      {
        "label": "Small Widget",
        "amount": {
          "currency": "USD",
          "value": "3.00"
        },
        "pending": false
      },
      {
        "label": "Shipping",
        "amount": {
          "currency": "USD",
          "value": "3.00"
        },
        "pending": false
      }
    ]
  },
  "shippingAddress": {
    "country": "US",
    "addressLine": [
      "One Microsoft Way"
    ],
    "region": "WA",
    "city": "Redmond",
    "dependantLocality": null,
    "postalCode": "98052",
    "sortingCode": "",
    "languageCode": "",
    "organization": "Microsoft",
    "recipient": "Patti Fernandez",
    "phone": "4255551212"
  },
  "shippingOption": "priority"
}
```

## PaymentComplete webhook

The payment complete webhook is called when the user finalizes the payment. The Payment service processes the payment and calls your webhook with the result.

The format of the payload sent to your webhook and the payload you must send back in the response is specified in the following sections.

### Payment complete webhook incoming payload

The Payment services sends a payload to your webhook in the following format.

| Field | Type| Description |
|-------|-----|-------------|
| `requestId` | String | The request ID that corresponds to this payment. This value matches the value of `id` in the `PaymentDetailsInit` object you sent back to the `loadentity` event in your payment request webhook. |
| `methodName` | String | Will be set to `https://pay.microsoft.com/microsoftpay`. |
| `details` | [PaymentCompleteDetails](#paymentcompletedetails) | Contains original `productContext`, payment token, and information about the amount. |
| `shippingAddress` | `AddressInit` | The selected shipping address. (If shipping information was requested in `loadentity` response) |
| `shippingOption` | String | The selected shipping option. (If shipping information was requested in `loadentity` response) |
| `payerName` | String | The payer's name. (If requested) |
| `payerEmail` | String | The payer's email address. (If requested) |
| `payerPhone` | String | The payer's phone number. (If requested) |

#### PaymentCompleteDetails

Contains details about the completed payment.

| Field | Type| Description |
|-------|-----|-------------|
| `productContext` | Object | A free-form object defined by the developer. The `productContext` is used to store contextual information needed for the payment process. This value is equal to the `productContext` sent to the payment request webhook for this payment. |
| `paymentToken` | String | A JSON Web Token containing payment information. |
| `amount` | `PaymentCurrencyAmount` | The amount of the payment. |

### Payment request webhook response payload

Your webhook should return a payload in the following format.

| Field | Type| Description |
|-------|-----|-------------|
| `requestId` | String | The request ID that corresponds to this payment. This value MUST match the value of `requestId` in the incoming request. |
| `result` | String | Indicates success or failure. Valid values are `success` and `fail`. |
| `details` | String | A simple human-readable string regarding the result. Used to relay a success message or simple error message. |
| `error` | [PaymentCompleteError](#paymentcompleteerror) | A more detailed error object. Only applicable if `result` is `fail`. |
| `entity` | `Invoice` | An updated invoice that reflects the result of the payment. For example, if the payment succeeded, the total is updated to 0 and the payment status is marked complete. |

#### PaymentCompleteError

Contains more detailed error information for failed payment complete requests.

| Field | Type| Description |
|-------|-----|-------------|
| `code` | String | Required. A language-independent string that indicates the error code. |
| `message` | String | Required. A human-readable representation of the error. |
| `target` | String | Optional. The target of the error. |
| `details`| Array of Object | Optional. Contains an array of objects with `code` and `message` properties. Represents distinct related errors that occurred during the request. |
| `innererror` | `PaymentCompleteError` | Provides a more specific error. |

The following values are supported in the `code` field when using Stripe as a payment processor.

| Value | Description |
|-------|-------------|
| `invalid_number` | The card number is not a valid credit card number. |
| `invalid_expiry_month` | The card's expiration month is invalid. |
| `invalid_expiry_year` | The card's expiration year is invalid. |
| `invalid_cvc` | The card's security code is invalid. |
| `incorrect_number` | The card number is incorrect. |
| `expired_card` | The card has expired. |
| `incorrect_cvc` | The card's security code is incorrect. |
| `incorrect_zip` | The card's ZIP or postal code failed validation. |
| `card_declined` | The card was declined. |
| `processing_error` | A generic error. An error occurred while processing the card. |

### Handling payment complete webhook POSTs

When your webhook receives a POST, you should do the following:

1. Validate the bearer token in the `Authorization` header to ensure that the request is coming from Microsoft.
1. Parse the `paymentToken` value to determine:
    - The type of the token (specified in the `Format` claim in the header)
    - The token required for the specified payment processor (the payload of the `paymentToken`)
1. Submit the extracted token to the specified payment processor.
1. Return a success or error as appropriate.

#### Example incoming payload

```json
{
    "requestId": "12345",
    "methodName": "https://pay.microsoft.com/microsoftpay",
    "details": {
        "productContext": {
            "invoiceId": "12345"
        },
        "paymentToken": "eyJWZXJz...",
        "amount": {
            "currency": "USD",
            "value": "10.00",
            "currencySystem": null
        }
    },
    "shippingAddress": {
        "country": "US",
        "addressLine": [
            "1 Microsoft Way"
        ],
        "region": "WA",
        "city": "Redmond",
        "dependantLocality": null,
        "postalCode": "98052",
        "sortingCode": "",
        "languageCode": "",
        "organization": "",
        "recipient": "Patti Fernandez",
        "phone": "+14255551212"
    },
    "shippingOption": "norush",
    "payerName": "Patti Fernandez",
    "payerEmail": "patti@contoso.com",
    "payerPhone": "+14255551212"
}
```

#### Example success response

```json
{
    "requestId": "12345",
    "result": "success",
    "details": "Thank you for paying your bill!",
    "entity": {
        "@type": "Invoice",
        "@context": "http://schema.org",
        "identifier": "12345",
        "url": "https://contoso.com/invoice",
        "broker": {
            "@type": "LocalBusiness",
            "name": "Contoso"
        },
        "paymentDueDate": "2019-01-31T00:00:00",
        "paymentStatus": "PaymentComplete",
        "totalPaymentDue": {
            "@type": "PriceSpecification",
            "price": 0,
            "priceCurrency": "USD"
        },
        "confirmationNumber": "98765"
    }
}
```

#### Example failure response

```json
{
    "requestId": "12345",
    "result": "fail",
    "details": "We were unable to charge your credit card.",
    "error": {
        "code": "card_error",
        "message:": "Card cannot be processed.",
        "target": "stripeToken",
        "innererror": {
            "code": "card_declined",
            "message": "Your credit card was declined."
        }
    }
}
```

## Securing your webhooks

All action requests from Microsoft have a bearer token in the HTTP `Authorization` header. This token is a [JSON Web Token](https://jwt.io/) (JWT) token signed by Microsoft, and it includes important claims that we strongly recommend should be verified by the service handling the associated request.

| Claim name | Value |
|------------|-------|
| `aud` | The merchant ID of the target webhook. This should match the merchant ID you obtained from the [Payments in Outlook Partner Dashboard](https://outlook.office.com/connectors/opay/partnerportal/). |
| `sub` | The email address of the user of the user who took the action. |
| `sender` | The identity of the sender of the payment request message. This should match the email address you sent the payment message from. |

Typically, a service will perform the following verifications.

1. The token is signed by Microsoft and not expired.
1. The `aud` claim corresponds to your merchant ID.

With all the above verifications done, the service can trust the `sender` and `sub` claims to be the identity of the sender and the user taking the action. A service can optionally validate that the `sender` and `sub` claims match the sender and user it is expecting.