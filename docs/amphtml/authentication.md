---
title: AMP Authentication
description: AMP Authentication
author: LezaMax
ms.topic: article
ms.technology: o365-amp
ms.author: humlezg
localization_priority: Priority
---
# AMP for Email Authentication

If your AMP email performs an authenticated action, such as accessing a protected Web API, you must supply access tokens as query string parameters in the URL of your actions and verify those tokens on your service. The following example illustrates this with an `amp-list`.

```html
<amp-list src="https://contoso.com/order-status?id=123&exampletoken=BB34X21F" height="200">
  <template type="amp-mustache">
    <p>{{order}}</p>
  </template>
</amp-list>
```

AMP for Email doesn't prescribe how access tokens should be created or validated. It is up to each email sender to ensure that the actions they allow users to take on email are scoped and secured. For example, if you email allows the user to update information, the access token should be scoped to only allow updates to the record and fields that the email is relevant to. You can also consider limiting the validity of an access tokens, for example, have an expiration date on them or limit the number of times the same access token can be used.
