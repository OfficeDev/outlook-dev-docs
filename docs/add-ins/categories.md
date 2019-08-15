---
title: Get and set categories
description: How to manage categories on mailbox and item
ms.topic: article
ms.date: 08/15/2019
localization_priority: Normal
---

# Get and set categories (preview)

In Outlook, a user can group messages and appointments by using a category to color-code them. The user defines categories in a master list on their mailbox. They can then apply one or more categories to a message or appointment item. To create a new [category](/javascript/api/outlook/office.categorydetails), provide a name and assign a color based on the list of available [preset colors](/javascript/api/outlook/office.mailboxenums.categorycolor) (though the actual color displayed depends on how the Outlook client renders it). You can use the Office JavaScript API to manage the categories master list on the mailbox and the categories applied to an item.

> [!IMPORTANT]
> Categories API for Outlook add-ins are currently [in preview](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview#categories) in Outlook on Windows and Mac connected to an Office 365 subscription. These APIs shouldn't be used in production environments yet.

## Manage categories available for mailbox items

Only categories in the master list on your mailbox are available for you to apply to a message or appointment. You can use the API to add, get, and remove master categories.

### Add master categories

The following example shows how you can add a category named "Urgent!" to the master list by calling [addAsync](/javascript/api/outlook/office.mastercategories#addasync-categories--options--callback-) on [mailbox.masterCategories](/javascript/api/outlook/office.mailbox#mastercategories).

```javascript
var masterCategoriesToAdd = [
    {
        "displayName": "Urgent!",
        "color": Office.MailboxEnums.CategoryColor.Preset0
    }
];

Office.context.mailbox.masterCategories.addAsync(masterCategoriesToAdd, function (asyncResult) {
    if (asyncResult.status === Office.AsyncResultStatus.Succeeded) {
        console.log("Successfully added categories to master list");
    } else {
        console.log("masterCategories.addAsync call failed with error: " + asyncResult.error.message);
    }
});
```

### Get master categories

The following example shows how you can get the list of categories by calling [getAsync](/javascript/api/outlook/office.mastercategories#getasync-options--callback-) on [mailbox.masterCategories](/javascript/api/outlook/office.mailbox#mastercategories).

```javascript
Office.context.mailbox.masterCategories.getAsync(function (asyncResult) {
    if (asyncResult.status === Office.AsyncResultStatus.Failed) {
        console.log("Action failed with error: " + asyncResult.error.message);
    } else {
        var masterCategories = asyncResult.value;
        console.log("Master categories:");
        masterCategories.forEach(function (item) {
            console.log("-- " + JSON.stringify(item));
        });
    }
});
```

### Remove master categories

The following example shows how you can remove a category named "Urgent!" from the master list by calling [removeAsync](/javascript/api/outlook/office.mastercategories#removeasync-categories--options--callback-) on [mailbox.masterCategories](/javascript/api/outlook/office.mailbox#mastercategories).

```javascript
var masterCategoriesToRemove = ["Urgent!"];

Office.context.mailbox.masterCategories.removeAsync(masterCategoriesToRemove, function (asyncResult) {
    if (asyncResult.status === Office.AsyncResultStatus.Succeeded) {
        console.log("Successfully removed categories from master list");
    } else {
        console.log("masterCategories.removeAsync call failed with error: " + asyncResult.error.message);
    }
});
```

## Manage categories on a message or appointment

You can use the API to add, get, and remove categories for a message or appointment item.

> [!IMPORTANT]
> Only categories in the master list on your mailbox are available for you to apply to a message or appointment. See the earlier section [Manage categories available for mailbox items](#manage-categories-available-for-mailbox-items) for more information.

### Add categories to an item

The following example shows how you can apply a category named "Urgent!" to the current item by calling [addAsync](/javascript/api/outlook/office.categories#addasync-categories--options--callback-) on [item.categories](/javascript/api/outlook/office.item#categories).

```javascript
var categoriesToAdd = ["Urgent!"];

Office.context.mailbox.item.categories.addAsync(categoriesToAdd, function (asyncResult) {
    if (asyncResult.status === Office.AsyncResultStatus.Succeeded) {
        console.log("Successfully added categories");
    } else {
        console.log("categories.addAsync call failed with error: " + asyncResult.error.message);
    }
});
```

### Get an item's categories

The following example shows how you can get the categories applied to the current item by calling [getAsync](/javascript/api/outlook/office.categories#getasync-options--callback-) on [item.categories](/javascript/api/outlook/office.item#categories).

```javascript
Office.context.mailbox.item.categories.getAsync(function (asyncResult) {
    if (asyncResult.status === Office.AsyncResultStatus.Failed) {
        console.log("Action failed with error: " + asyncResult.error.message);
    } else {
        var categories = asyncResult.value;
        console.log("Categories:");
        categories.forEach(function (item) {
            console.log("-- " + JSON.stringify(item));
        });
    }
});
```

### Remove categories from an item

The following example shows how you can remove a category named "Urgent!" from the current item by calling [removeAsync](/javascript/api/outlook/office.categories#removeasync-categories--options--callback-) on [item.categories](/javascript/api/outlook/office.item#categories).

```javascript
var categoriesToRemove = ["Urgent!"];

Office.context.mailbox.item.categories.removeAsync(categoriesToRemove, function (asyncResult) {
    if (asyncResult.status === Office.AsyncResultStatus.Succeeded) {
        console.log("Successfully removed categories");
    } else {
        console.log("categories.removeAsync call failed with error: " + asyncResult.error.message);
    }
});
```

## See also

[Categories API in preview](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview#categories)