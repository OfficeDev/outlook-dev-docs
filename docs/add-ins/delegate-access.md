---
title: Enable delegate access scenarios in an Outlook add-in
description: Briefly describes delegate access and discusses how to configure add-in support.
ms.topic: article
ms.date: 02/21/2019
localization_priority: Normal
---

# Enable delegate access scenarios in an Outlook add-in (preview)

A mailbox owner can use the delegate access feature to [allow someone else to manage their mail and calendar](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926). This article specifies which delegate permissions the Office JavaScript API supports and describes how to enable delegate access scenarios in your Outlook add-in.

> [!IMPORTANT]
> Delegate access for Outlook add-ins is currently in preview and only supported in clients that run against Exchange Online. Delegate access APIs should not yet be used in production environments.

## Supported permissions for delegate access

The following table describes the delegate permissions that the Office JavaScript API supports.

|Permission|Value|Description|Bit position<br>from the right|
|---|:---:|---|:---:|
|Read|1|Can read items.|1|
|Write|2|Can create items.|2|
|DeleteOwn|4|Can delete only the items they created.|3|
|DeleteAll|8|Can delete any items.|4|
|EditOwn|16|Can edit only the items they created.|5|
|EditAll|32|Can edit any items.|6|

> [!NOTE]
> Currently the API supports getting existing delegate permissions, but not setting delegate permissions.

The [DelegatePermissions](/javascript/api/outlook/office.mailboxenums.delegatepermissions) object is implemented using a bitmask to indicate the delegate's permissions. Each position in the bitmask represents a particular permission and if it's set to `1` then the delegate has the respective permission. For example, if the second bit from the right is `1`, then the delegate has **Write** permission. You can see an example of how to check for a specific permission in the [Get delegate permissions](#get-delegate-permissions) section later in this article.

## Configure the manifest

To enable delegate access scenarios in your add-in, you must set the [SupportsSharedFolders](/office/dev/add-ins/reference/manifest/supportssharedfolders) element to `true` in the manifest (under the parent element `DesktopFormFactor`).

> [!IMPORTANT]
> Because delegate access for Outlook add-ins is currently in preview, add-ins that use the `SupportSharedFolders` element cannot be published to AppSource or deployed via centralized deployment.

The following example shows the `SupportsSharedFolders` element set to `true` in a section of the manifest.

```XML
<DesktopFormFactor>
  <FunctionFile resid="residDesktopFuncUrl" />
  <SupportsSharedFolders>true</SupportsSharedFolders>
  <ExtensionPoint xsi:type="PrimaryCommandSurface">
    <!-- configure this extension point -->
  </ExtensionPoint>
</DesktopFormFactor>
```

## Get delegate permissions

You can get an item's shared properties in Compose or Read mode by calling the [item.getSharedPropertiesAsync](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/office.context.mailbox.item#getsharedpropertiesasyncoptions-callback) method. This returns a [SharedProperties](/javascript/api/outlook/office.sharedproperties) object that currently provides the delegate's permissions, the owner's email address, and the REST URL of the owner's mailbox.

The following example shows how to get the shared properties of a message or appointment and check if the delegate has **Write** permission.

```js
Office.context.mailbox.item.getSharedPropertiesAsync(callback);

function callback (asyncResult) {
  var sharedProperties = asyncResult.value;

  // Check if user is a delegate before proceeding.
  if (sharedProperties != null) {
    console.log(JSON.stringify(sharedProperties));

    var delegatePermissions = sharedProperties.delegatePermissions;

    // Determine if user can do the expected action.
    // E.g., do they have Write permission?
    if ((delegatePermissions & MailboxEnums.Write) != 0 ) {
      // Expected action...
    }
  }
}
```

## See also

- [Allow someone else to manage your mail and calendar](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926)
- [How to order manifest elements](/office/dev/add-ins/develop/manifest-element-ordering)
- [Mask (computing)](https://en.wikipedia.org/wiki/Mask_(computing))
- [JavaScript bitwise operators](https://www.w3schools.com/js/js_bitwise.asp)