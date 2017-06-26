---
title: Get or set the location of an appointment in an Outlook add-in | Microsoft Docs
description: Learn how to get or set the location of an appointment in an Outlook add-in.
author: jasonjoh

ms.topic: article
ms.technology: office-add-ins
ms.date: 06/13/2017
ms.author: jasonjoh
---

# Get or set the location when composing an appointment in Outlook

The JavaScript API for Office provides asynchronous methods ([getAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5) and [setAsync](https://dev.office.com/reference/add-ins/outlook/1.5/Location?product=outlook&version=v1.5)) to get and set the location of an appointment that the user is composing. These asynchronous methods are available to only compose add-ins. To use these methods, make sure you have set up the add-in manifest appropriately for Outlook to activate the add-in in compose forms, as described [Create Outlook add-ins for compose forms](compose-scenario.md).

The [location](https://dev.office.com/reference/add-ins/outlook/1.5/Office.context.mailbox.item?product=outlook&version=v1.5) property is available for read access in both compose and read forms of appointments. In a read form, you can access the property directly from the parent object, as in:




```js
item.location
```

But in a compose form, because both the user and your add-in can be inserting or changing the location at the same time, you must use the asynchronous method  **getAsync** to get the location, as shown below:




```js
item.location.getAsync
```

The  **location** property is available for write access in only compose forms of appointments, but not in read forms.

As with most asynchronous methods in the JavaScript API for Office,  **getAsync** and **setAsync** take optional input parameters. For more information about specifying these optional input parameters, see [Asynchronous programming in Office Add-ins](https://dev.office.com/docs/add-ins/develop/asynchronous-programming-in-office-add-ins).


## To get the location


This section shows a code sample that gets the location of the appointment that the user is composing, and displays the location. This code sample assumes a rule in the add-in manifest that activates the add-in in a compose form for an appointment, as shown below.


```XML
<Rule xsi:type="ItemIs" ItemType="Appointment" FormType="Edit"/>

```

To use  **item.location.getAsync**, provide a callback method that checks for the status and result of the asynchronous call. You can provide any necessary arguments to the callback method through the  _asyncContext_ optional parameter. You can obtain status, results and any error using the output parameter _asyncResult_ of the callback. If the asynchronous call is successful, you can get the location as a string using the [AsyncResult.value](https://dev.office.com/reference/add-ins/outlook/1.5/simple-types?product=outlook&version=v1.5) property.




```js
var item;

Office.initialize = function () {
    item = Office.context.mailbox.item;
    // Checks for the DOM to load using the jQuery ready function.
    $(document).ready(function () {
        // After the DOM is loaded, app-specific code can run.
        // Get the location of the item being composed.
        getLocation();
    });
}

// Get the location of the item that the user is composing.
function getLocation() {
    item.location.getAsync(
        function (asyncResult) {
            if (asyncResult.status == Office.AsyncResultStatus.Failed){
                write(asyncResult.error.message);
            }
            else {
                // Successfully got the location, display it.
                write ('The location is: ' + asyncResult.value);
            }
        });
}

// Write to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```


## To set the location


This section shows a code sample that sets the location of the appointment that the user is composing. Similar to the previous example, this code sample assumes a rule in the add-in manifest that activates the add-in in a compose form for an appointment.

To use  **item.location.setAsync**, specify a string of up to 255 characters in the data parameter. Optionally, you can provide a callback method and any arguments for the callback method in the  _asyncContext_ parameter. You should check the status, result and any error message in the _asyncResult_ output parameter of the callback. If the asynchronous call is successful, **setAsync** inserts the specified location string as plain text, overwriting any existing location for that item.




```js
var item;

Office.initialize = function () {
    item = Office.context.mailbox.item;
    // Check for the DOM to load using the jQuery ready function.
    $(document).ready(function () {
        // After the DOM is loaded, app-specific code can run.
        // Set the location of the item being composed.
        setLocation();
    });
}

// Set the location of the item that the user is composing.
function setLocation() {
    item.location.setAsync(
        'Conference room A',
        { asyncContext: { var1: 1, var2: 2 } },
        function (asyncResult) {
            if (asyncResult.status == Office.AsyncResultStatus.Failed){
                write(asyncResult.error.message);
            }
            else {
                // Successfully set the location.
                // Do whatever appropriate for your scenario
                // using the arguments var1 and var2 as applicable.
            }
        });
}

// Write to a div with id='message' on the page.
function write(message){
    document.getElementById('message').innerText += message; 
}
```


## Additional resources



- [Get and set item data in a compose form in Outlook](get-and-set-item-data-in-a-compose-form.md)
    
- [Get and set Outlook item data in read or compose forms](item-data.md)
    
- [Create Outlook add-ins for compose forms](compose-scenario.md)
    
- [Asynchronous programming in Office Add-ins](https://dev.office.com/docs/add-ins/develop/asynchronous-programming-in-office-add-ins)
    
- [Get, set, or add recipients when composing an appointment or message in Outlook](get-set-or-add-recipients.md)
    
- [Get or set the subject when composing an appointment or message in Outlook](get-or-set-the-subject.md)
    
- [Insert data in the body when composing an appointment or message in Outlook](insert-data-in-the-body.md)
    
- [Get or set the time when composing an appointment in Outlook](get-or-set-the-time-of-an-appointment.md)
    
