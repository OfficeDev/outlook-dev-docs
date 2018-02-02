# Partner Onboarding

## Introduction

There are three components to creating a payment experience in Outlook:

1. **Actionable Message** -- Mark-up added to your email body that displays a card as the part of the native UI of Outlook. [Overview Here](https://docs.microsoft.com/en-us/outlook/actionable-messages/)

2. **Payment Request Webhook** -- The API endpoint that Outlook calls after the user clicks 'Pay' in the Actionable Message. It is expected to returns a payload to create a checkout experience in Outlook

3. **Payment Complete Webhook** -- The API endpoint that is called when a user completes the payment process. It provisions the paid item and processes the payment instrument and amount

## Actionable Message Markup

Below is a sample Actionable message with a sample Merchant ID. The Merchant ID is associated with your organization and payment token format.

```
{
    "@type" : "MessageCard",
    "@context" : "http://schema.org/extensions",
    "themeColor" : "00188F",
    "originator": "test",
    "summary" : "Mock Merchant Test Card",
    "sections" : [{
        "title" : "**You have an invoice from Furniture Direct LLC.**",
        "activityTitle" : "Invoice summary:",
        "facts": [
            {
                "name": "Invoice Number",
                "value": "103032"
            },
            {
                "name": "Due Date",
                "value": "01/31/2019"
            },
            {
                "name": "Balance Due",
                "value": "$10.00"
            }
        ],
        "potentialAction" : [{       
            "@type" : "Transaction",
            "name" : "Pay",
            "isPrimaryAction" : true,
            "merchantId": "SAMPLE_ID",
            "displayId": "SAMPLE_ID",
            "productContext": {"invoiceId": "103032", "createdDate" :"12/8/2017", "dueDate":"01/31/2019"},
            "environment": "ppe"
        }]
    }]
}
```

This additional mark-up is inserted in the body of the invoice email wrapped in `<script type="application/ld+json">` tag. Certain fields should be dynamically generated. In the `potentialAction` key:

|Key |Value |
|---|---|
|`productContext`| This is an object you can embed into the email to be replayed. 
## Webhooks

There are two webhooks that Outlook uses to call into the partner to drive this experience:

* **PaymentRequest** This endpoint has two purposes. 
  1. It returns a definition of the properties needed to complete a transaction. Outlook will render the UX to collect the necessary information from the user.  
  
  2. It also returns the most updated invoice entity to be shown to the user as payment context.

* **PaymentComplete** This endpoint also has two purposes:

    1. It will charge the payment instrument provided by Outlook for the amount provided in the payload
    2. It will set the invoice state to paid after successfully charging the payment instrument

### Payment Request Webhook

the partner will need to notify Outlook the URLs of the webhooks the partner will be providing for Outlook to call.

To render the Payment experience and provide context to the user for the most up-to-date invoice details,Outlook will post to the partner's registered endpoint with the following request:

```
{ 
  "methodData": [{ 
      "supportedMethods": "https://pay.microsoft.com/microsoftpay", 
      "data": { 
      "event": "loadentity", 
      "productContext": { 
        "test_key": "test_value"     
      }
    }
  }] 
} 
```

The main property to note is the `productContext`
|Key |Value |
|---|---|
|`productContext`| This property passes through the email embedded context to the payment request webhook |

The expected response by the partner is the full payment request object.

Below is an example:

```
{ 
  "id": "test_invoice", 
  "methodData": [{ 
    "supportedMethods": "https://pay.microsoft.com/microsoftpay", 
    "data": { 
      "supportedNetworks": ['visa', 'mastercard', 'amex'], 
      "supportedTypes": ['credit'], 
      "market": "US", 
      "productContext": { "test_key": "test_value" }, 
      "event": "loadlandingpage" 
      "entity": { 
        "@context": "http://schema.org", 
        "@type": "Invoice", 
        "identifier": "test_invoice_id", 
        "url": "https://url-to-invoice-details.com", 
        "broker": { 
          "@type": "LocalBusiness", 
          "name": "Contoso Widget Store" 
        }, 
        "paymentDueDate": "2017-12-15", 
        "totalPaymentDue": { 
          "@type": "PriceSpecification", 
          "price": 10.00, 
          "priceCurrency": "USD" 
        }, 
        "paymentStatus": "PaymentDue", 
      } 
    } 
  }], 
  "options": { 
    "requestPayerName": true, 
    "requestPayerEmail": true, 
    "requestPayerPhone": true, 
    "requestShipping": false, 
  }, 
  "details": { 
    "id": "test_invoice", 
    "total": { 
      "label": "Total Due", 
      "amount": { 
        "currency": "USD", 
        "value": "12.00" 
      } 
    }, 
    "displayItems": [{ 
      "label": "Invoice #1000", 
      "amount": { 
        "currency": "USD", 
        "value": "10.00" 
      } 
    }] 
  } 
}
``` 
This object defines the attributes to be collected in the payment workflow to be replayed to the partner's PaymentComplete webhook. 
The important keys are defined below:

|Key |Value |
|---|---|
|`entity`| This property requires a schema.org invoice entity as defined in the same `entity` payload |
|`options`| The additional information to needed to complete the transaction |
|`total`| Amount to be collected to be displayed in payment UX |
|`displayItems`| Array of line items for the breakdown of the total amount |

### Payment Complete Webhook

Upon the user clicking "Pay", Outlook will submit the payment payload to the partner's PaymentComplete webhook. 

## Webhook Format

An example of the request:
```
{ 
  "requestId":" "test_invoice", 
  "methodName": "https://pay.microsoft.com/microsoftpay", 
  "details": { 
    "productContext": { test_key: "test_value" }, 
    "paymentToken": <stripe token>, 
    "amount": { 
      "currency": "USD", 
      "value": "18.00" 
    } 
  }, 
  "shippingAddress": { 
    "country": "US", 
    "addressLine": [ 
      "250 Bellevue Way NE", 
      "Apt 555" 
    ], 
    "languageCode": "en-us", 
    "region": "WA", 
    "city": "Bellevue", 
    "postalCode": "98004", 
    "organization": "Microsoft", 
    "recipient": "Peter Chen", 
    "phone": "123-555-1234 
  }, 
  "payerName": "Peter Chen", 
  "payerEmail": "test@test.com", 
  "payerPhone": "123-555-1234" 
} 
```

The payload contains the collected information from the payment workflow including the Stripe token that can be charged. Note that the product context is still provided so the webhook may remain stateless. 

The expected response:
```
{ 
  "requestId": "test_invoice", 
  "result": "success" (or "fail" if failure), 
  "details": "more details",   //this field will surface in the actionable message 
  "entity": { 
    "@context": "http://schema.org", 
    "@type": "Invoice", 
    "identifier": "test_invoice_id", 
    "url": "https://url-to-invoice-details.com", 
    "broker": { 
      "@type": "LocalBusiness", 
      "name": "Contoso Widget Store" 
    }, 
    "paymentDueDate": "2017-12-15", 
    "totalPaymentDue": { 
      "@type": "PriceSpecification", 
      "price": 0.00, 
      "priceCurrency": "USD" 
    }, 
    "confirmationNumber": "test_confirmation_number", 
    "paymentStatus": "http://schema.org/PaymentComplete" 
  } 
}       
```

## Payment Instrument

In the webhook contract highlighted above, Outlook will provide a `paymentToken` under the details field. The paymentToken is a tokenized payment instrument that may come in two formats:

|Type |Description |
|---|---|
|Stripe Token| Microsoft with authenticate with you via Stripe OAuth to create Stripe tokens on behalf of you or your connect merchants. This format works for two types of Stripe accounts. For Stripe Connect platforms,  |
|MSPay Token | Microsoft will encrypt a payment instrument (PAN) using RSA encryption. Onboarding will be via key ceremony where a public-private key pair is generated |










