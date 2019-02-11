---
title: Enable an Outlook add-in to support delegate access scenarios
description: Briefly describes delegate access and discusses how to configure add-in support.
ms.topic: article
ms.date: 02/11/2019
localization_priority: Normal
---

# How to enable delegate access scenarios in an Outlook add-in (preview)

There are a few steps you should follow to configure your Outlook add-in to support delegate access. These include enabling the `SupportsSharedFolders` property in the manifest and using the `SharedProperties` object.

## What is delegate access?

Consider a business setting where an executive enables their assistant to manage the executive's mail and calendar. Depending on the level of access, the assistant or delegate can respond on the mailbox owner's behalf and create new appointments, among other tasks. To learn more about delegate access and how to give delegates permission, see [Allow someone else to manage your mail and calendar](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926).

## Supported permissions

You should know which delegate permissions the JavaScript for Office API supports.

**Table 1 Supported permissions for delegate access**

|Permission|Value|Description|Bit position<br>from the right|
|---|:---:|---|:---:|
|Read|1|Can read items.|1|
|Write|2|Can create items.|2|
|DeleteOwn|4|Can delete only the items they created.|3|
|DeleteAll|8|Can delete any items.|4|
|EditOwn|16|Can edit only the items they created.|5|
|EditAll|32|Can edit any items.|6|

> [!NOTE]
> Using the API, you can only get existing delegate permissions.

Also, the [DelegatePermissions](/javascript/api/outlook/office.mailboxenums.delegatepermissions) object is implemented using a bit mask to indicate all the delegate's permissions. This means that each position represents a particular permission and if it is set to `1` (i.e., it's on) then the delegate has the respective permission. For example, if the second bit from the right is `1`, then the delegate has **Write** permission. You can see an example of how to check for a specific permission in the [Use in messages and appointments for compose and read modes](#use-in-messages-and-appointments-for-compose-and-read-modes) section later in this article.

## Configure the manifest

You should set the [SupportsSharedFolders](/office/dev/add-ins/reference/manifest/supportssharedfolders) element to `true` under the parent element `DesktopFormFactor` so that your add-in is available in delegate access scenarios.

> [!IMPORTANT]
> The `SupportsSharedFolders` element is only available in the [Outlook add-ins preview requirement set](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview) against Exchange Online.
>
> Add-ins that use this element cannot be published to AppSource or deployed via centralized deployment.

The following example shows the `SupportsSharedFolders` element set to `true` in a section of the manifest.

```XML
<DesktopFormFactor>
  <FunctionFile resid="residDesktopFuncUrl" />
  <SupportsSharedFolders>true</SupportsSharedFolders>
  <ExtensionPoint xsi:type="PrimaryCommandSurface">
    <!-- information about this extension point -->
  </ExtensionPoint>
</DesktopFormFactor>
```

## Use in messages and appointments for compose and read modes

You can get the item's shared properties by calling the [item.getSharedPropertiesAsync](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/office.context.mailbox.item#getsharedpropertiesasyncoptions-callback) method. This returns a [SharedProperties](/javascript/api/outlook/office.sharedproperties) object that currently provides the delegate's permissions, the owner's email address, and the REST URL of the owner's mailbox.

The following example shows how to get an item's shared properties and check if the delegate has **Write** permission.

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

## Limitations and restrictions

At present, delegate access is only available in the [Outlook add-ins preview requirement set](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview) against Exchange Online. Add-ins that use the `SupportsSharedFolders` element in their manifest cannot be published to AppSource or deployed via centralized deployment as the delegate access feature APIs should not be used in production environments yet.

## See also

- [Allow someone else to manage your mail and calendar](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926)
- [How to order manifest elements](/office/dev/add-ins/develop/manifest-element-ordering)
- [Mask (computing)](https://en.wikipedia.org/wiki/Mask_(computing))
- [JavaScript bitwise operators](https://www.w3schools.com/js/js_bitwise.asp)