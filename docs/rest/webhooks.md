---
title: Webhooks for Mail, Calendar and Contacts REST notifications | Microsoft Docs
description: Learn how to use webhooks to receive notifications of changes to mail, calendar, and contact data.
author: jasonjoh

ms.topic: conceptual
ms.technology: graph-webhooks
ms.date: 04/26/2017
ms.author: jasonjoh
---

# Webhooks for Mail, Calendar and Contacts REST notifications

We are excited about the release of the [Outlook Notifications REST API](https://msdn.microsoft.com/office/office365/APi/notify-rest-operations) for mail, calendar, and contacts in Office 365 and Outlook.com. The Outlook Notifications REST API uses a webhook mechanism to deliver notifications to clients. A client, in this context, is actually a third-party web service that configures its own notification URL to which Office 365 pushes these notifications.  Mail and calendaring apps typically use notifications to update their local cache and corresponding client views upon changes. Additionally, CRM apps usually use notifications to update their systems with changes to relevant conversations.
 
Using the Outlook Notifications REST API, an app can now subscribe to notifications of a specific resource (such as messages in the Inbox) to get informed about changes to the resource. Once the Outlook notifications service accepts the subscription request, it pushes notifications through webhooks to the notification URL specified in the subscription. The app then takes action according to its business logic (e.g. fetches new messages, update unread count, remove deleted messages from the client view, etc.).
 
Apps need to renew their subscriptions before they expire. They can also unsubscribe at any time to stop getting notifications. 
 
Let's take a look at the subscription process.

## Creating a subscription

Creating a subscription is the first step needed to start receiving notifications for a specific entity collection. The subscription process is as follows:

1. Client sends a subscription (POST) request for a specific entity collection. 
1. Outlook notifications service verifies the request, validates the notification URL and sends response to the client.
  1. Success: creates subscription with unique ID and sends corresponding response.
  1. Failure: sends error response with error code and details.
1. Client must store the subscription ID in order to correlate notification with corresponding subscription. 
 
### Characteristics of subscriptions

- Subscriptions can be performed for entity collections (such as messages, events, and contacts).
  - Specific folder including search folders:
    - `https://outlook.office.com/api/v2.0/me/mailfolders('inbox')/messages`
  - Top level collection:
    - `https://outlook.office.com/api/v2.0/me/events`
    - `https://outlook.office.com/api/v2.0/me/messages`
- A subscription persists throughout its lifetime. This is an advantage over our older SOAP API Exchange Web Services (EWS) where subscriptions can be lost because of problems on the server side such as application pool crashes or server failover.
- Creating a subscription requires read scope to the resource; e.g. `mail.read` is required to get notifications on Inbox messages.
- Subscriptions work on regular folders such as Inbox as well as search folders.
- Subscriptions expire. The current maximum expiration time is 3 days from time of creation. Apps need to renew their subscriptions before expiration time otherwise they will need to re-subscribe.  
 
### Notification URL validation 
The Outlook notifications service validates the notification URL in a subscription request before creating a new subscription which occurs as follows: 

1. Outlook notifications service sends a POST to the notification URL:

        POST https://{notificationUrl}?validationtoken={TokenDefinedByService}
        ClientState: {Data sent in ClientState value in subscription request (if any)}
     
1. Webhooks service must provide a `200` response with the `validationtoken` value in its body as type `plain/text` within 5 seconds. The validation token is a random string that should be discarded by the webhook after providing it in the response.

### Sample create subscription request

    POST https://outlook.office.com/api/v2.0/me/subscriptions  HTTP/1.1
    Content-Type: application/json
    {
      "@odata.type": "#Microsoft.OutlookServices.PushSubscription",
      "Resource": "https://outlook.office.com/api/v2.0/me/mailfolders('inbox')/messages",
      "NotificationURL": "https://mywebhook.azurewebsites.net/api/send/myNotifyClient",  
      "ChangeType": "Created, Updated, Deleted",
      "SubscriptionExpirationDateTime": "2015-11-20T23:05:01.9420124Z"
      "ClientState": "MySecretClientStateString"
    }

The `@odata.type`, `Resource`, `NotificationURL` and `ChangeType` properties are required while the rest are optional. Refer to https://msdn.microsoft.com/office/office365/APi/notify-rest-operations for property definitions and values.
 
### Sample create subscription response

Subscription response is a replay of the request with the following additional properties and values:

- `Id` - A unique identifier per subscription. Client needs to store this identifier to match it with notifications.
- `SubscriptionExpirationDateTime` The actual expiration time in case it was not specified in the request, or the time specified in the request was greater than three days.
- `ChangeType` adds two notification types: `Acknowledgment` and `Missed`. They will be explained in the notification section. 

For example, the sample request above generates a response like the following:

    {
      "@odata.context": "https://outlook.office.com/api/v2.0/$metadata#Me/Subscriptions/$entity",
      "@odata.id": "https://outlook.office.com/api/v2.0/Users('user@contoso.com')/subscriptions('NDc0MEE4...QQ==')",
      "@odata.type": "#Microsoft.OutlookServices.PushSubscription",
      "ChangeType": "Created, Updated, Deleted, Acknowledgment, Missed",
      "ClientState": "MySecretClientStateString",
      "Id": "NDc0MEE4...QQ==",
      "NotificationURL": "https://mywebhook.azurewebsites.net/api/send/myNotifyClient",
      "Resource": "https://outlook.office.com/api/v2.0/me/folders('inbox')/messages",
      "SubscriptionExpirationDateTime": "2015-11-20T23:05:01.9420124Z"
    }
 
## Renewing a subscription

The client can renew a subscription to the specified expiration date (up to three days from the time of request). The `SubscriptionExpirationDateTime` property is optional. If no value is specified, it renews to 3 days from the time of the request.

### Sample renew subscription request
 
    PATCH https://outlook.office.com/api/v2.0/me/subscriptions('NDc0MEE4...QQ==') 
    
    {
      "@odata.type": "#Microsoft.OutlookServices.PushSubscription",
      "SubscriptionExpirationDateTime": "2015-11-22T23:16:31.3604491Z"
    }
 
 
### Sample renew subscription response

THe renew response is a replay of the create subscription response with the new expiration date-time.

    {
      "@odata.context": "https://outlook.office.com/api/v2.0/$metadata#Me/Subscriptions/$entity",
      "@odata.id": "https://outlook.office.com/api/v2.0/Users('user@contoso.com')/subscriptions('NDc0MEE4...QQ==')",
    "@odata.type": "#Microsoft.OutlookServices.PushSubscription",
      "ChangeType": "Created, Updated, Deleted, Acknowledgment, Missed",
      "ClientState": "MySecretClientStateString",
      "Id": "NDc0MEE4...QQ==",
      "Resource": "https://outlook.office.com/api/v2.0/me/folders('inbox')/messages",
      "SubscriptionExpirationDateTime": "2015-11-22T23:16:31.3604491Z"
    }
 
 
## Unsubscribe

The client unsubscribes from a subscription by deleting the subscription using its ID.

    DELETE https://outlook.office.com/api/v2.0/me/subscriptions('NDc0MEE4...QQ==')
 
A successful response is indicated by an HTTP `204 No Content` response code. 
 
## Notifications

Once a subscription is established, the client will start receiving notifications for changes related to the corresponding resource via a webhook to its notification URL, according to the client's specified change type (such as `Changed`). 
 
You have probably noticed that the subscription response has two additional change types - `Missed` and `Acknowledgment`. They are special types of notifications that are used as follows:

- The notifications services send a `Missed` notification if the service is unable to send out the requested notifications. Although it is good for clients to handle this notification, its discussion will be part of advanced topics in a later post.
- `Acknowledgement` notification has been deprecated because it was replaced by the notification URL validation process. We are in the process of removing it from the subscription response.
 
### Structure

Each notification has the following structure, regardless of its type:

- `ClientState` header: ONLY if the client specified the `ClientState` property in the subscription request.
- `SubscriptionId`: The ID for the subscription to which this notification belongs.
- `SubscriptionExpirationDateTime`: The expiration time for the subscription.
- `ChangeType`: The event type that caused the notification (e.g. `Created` on mail receive, or `Update` on marking a message read, etc.). 
- `SequenceNumber`: To help identify if a notification was missed by the client. Handling sequence number on the client will be discussed as part of advanced topics in a later post.
 
Additionally, a notification related to a resource change (such as receiving, reading or deleting a message) has an additional `ResourceData` property that contains the ID of the item that was changed. A client can use this ID to handle this item according to its business logic (e.g. fetch this item, sync its folder, etc.).
 
### Sample notifications

#### Receive email notification 

When the user receives a new message in the Inbox, the notifications service sends a notification with `ChangeType` set to `Created`.

    {
      "value": [
        {
          "@odata.type": "#Microsoft.OutlookServices.Notification",
          "Id": null,
          "SubscriptionId": "NDc0MEE4...QQ==",
          "SubscriptionExpirationDateTime": "2015-11-20T23:16:31.3604491Z",
          "SequenceNumber": 8,
          "ChangeType": "Created",
          "Resource": "https://outlook.office365.com/api/v2.0/Users('user@contoso.com')/Messages('AAMkAGY1...AAA=')",
          "ResourceData": {
            "@odata.type": "#Microsoft.OutlookServices.Message",
            "@odata.id": "https://outlook.office365.com/api/v2.0/Users('user@contoso.com')/Messages('AAMkAGY1...AAA=')",
            "@odata.etag": "W/\"CQAAABYAAABU4oDY3YDWSanCtdlMJCDOAADe14EA\"",
            "Id": "AAMkAGY1...AAA="
          }
        }
      ]
    }
 
#### Read email notification

When the user marks a message read, the notifications services sends a notification with the `ChangeType` set to `Updated`.

    {
      "value": [
        {
          "@odata.type": "#Microsoft.OutlookServices.Notification",
          "Id": null,
          "SubscriptionId": "NDc0MEE4...QQ==",
          "SubscriptionExpirationDateTime": "2015-11-20T23:16:31.3604491Z",
          "SequenceNumber": 2,
          "ChangeType": "Updated",
          "Resource": "https://outlook.office365.com/api/v2.0/Users('user@contoso.com')/Messages('AAMkAGY1...AAA=')",
          "ResourceData": {
            "@odata.type": "#Microsoft.OutlookServices.Message",
            "@odata.id": "https://outlook.office365.com/api/v2.0/Users('user@contoso.com')/Messages('AAMkAGY1...AAA=')",
            "@odata.etag": "W/\"CQAAABYAAABU4oDY3YDWSanCtdlMJCDOAADe14EA\"",
            "Id": "AAMkAGY1...AAA="
        }
      ]
    }
 
#### Missed notification

    {
      "value": [
        {
          "@odata.type": "#Microsoft.OutlookServices.Notification",
          "Id": null,
          "SubscriptionId": "NDc0MEE4...QQ==",
          " SubscriptionExpirationDateTime": "2015-11-20T23:16:31.3604491Z ",
          "SequenceNumber": 1,
          "ChangeType": "Missed"
        }
      ]
    }

## Next steps

We are really excited about the Outlook Notification REST API and webhooks for mail, calendar and contacts. Try them out and send us feedback on [UserVoice](https://officespdev.uservoice.com). This will help us improve our APIs and let you create delightful experiences in your apps.