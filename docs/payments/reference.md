---
title: Payments in Outlook webhook reference | Microsoft Docs
description: Learn about the payload fields for requests sent to payment webhooks by Outlook
author: jasonjoh

ms.topic: reference
ms.technology: o365-connectors 
ms.date: 03/20/2018
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
| `productContext` | Object | A free-form object defined by the developer. The `productContext` can be used to store any contextual information needed for the payment process. |
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

1. Validate the bearer token in the `Authorization` header to ensure that the request is coming from Microsoft.
1. Check the `event` property of the [Data](#data) object to determine what UI event triggered the POST.
    1. If `event` is set to `loadentity`, follow the steps in [Load the invoice](#load-the-invoice).
    1. If `event` is set to `shippingaddresschange`, follow the steps in [Handle shipping address change]().
    1. If `event` is set to `shippingoptionchange`, follow the steps in [Handle shipping option change]().
1. Return a `200 OK` response with the [Payment request webhook payload](#payment-request-webhook-payload) specified by the previous step in the body.

#### Load the invoice

When the `event` property in the incoming request is set to `loadentity` your webhook should load the invoice to provide details about the charges on the invoice to the Payments service. Your webhook will also indicate what information is required by your service regarding the payer.

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

When the `event` property in the incoming request is set to `shippingaddresschange` your webhook should check the shipping address selected and return any applicable shipping options.

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

When the `event` property in the incoming request is set to `shippingoptionchange` your webhook should check which shipping option was selected by the user and update the invoice accordingly.

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