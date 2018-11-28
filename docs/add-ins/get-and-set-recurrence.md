---
title: Get and set recurrence in an Outlook add-in
description: Get and set various recurrence properties of an item in an Outlook add-in.
author: elizs
ms.topic: article
ms.technology: office-add-ins
ms.date: 11/28/2018
ms.author: elizs
---

# Get and set recurrence

Imagine that you need to schedule a meeting with your team to discuss the current status of a project. It turns out that you need the status repeatedly until the project is completed. Instead of scheduling a meeting each time, you can configure the meeting to include how often you want it to occur. Then let's say that subsequently one of the meetings is at a time where you or one of the key attendees can't make it. You can then update that particular instance or occurrence, or cancel it altogether, without affecting the remainder of the series. Note that you can also set up a recurrence for your repeat appointments.

**Table 1. Available item states**

|State|Editable?|Viewable?|
|---|---|---|
|Appointment organizer - compose series|Yes ([`setAsync`][setAsync link])|Yes ([`getAsync`][getAsync link])|
|Appointment organizer - compose instance|No (`setAsync` returns an error)|Yes ([`getAsync`][getAsync link])|
|Appointment attendee - read series|No (`setAsync` not available)|Yes ([`item.recurrence`][item.recurrence link])|
|Appointment attendee - read instance|No (`setAsync` not available)|Yes ([`item.recurrence`][item.recurrence link])|
|Meeting request - read series|No (`setAsync` not available)|Yes ([`item.recurrence`][item.recurrence link])|
|Meeting request - read instance|No (`setAsync` not available)|Yes ([`item.recurrence`][item.recurrence link])|

## Available recurrence patterns

To configure the recurrence pattern, you need to combine the recurrence type and its applicable recurrence properties (if any).

**Table 2. Recurrence types and their applicable properties**

|[Recurrence type](/javascript/api/outlook_1_7/office.mailboxenums.recurrencetype)|Valid [recurrence properties](/javascript/api/outlook_1_7/office.recurrenceproperties)|Usage|
|---|---|---|
|`daily`|- [`interval`][interval link]|An appointment occurs every *interval* days. Example: An appointment occurs every **_2_** days.|
|`weekday`||An appointment occurs every weekday.|
|`monthly`|- [`interval`][interval link]<br/>- [`dayOfMonth`][dayOfMonth link]<br/>- [`dayOfWeek`][dayOfWeek link]<br/>- [`weekNumber`][weekNumber link]|1. An appointment occurs on day *dayOfMonth* every *interval* months. Example: An appointment occurs on day **_5_** every **_4_** months.<br/>2. An appointment occurs on the *weekNumber* *dayOfWeek* every *interval* months. Example: An appointment occurs on the **_third_** **_Thursday_** every **_2_** months.|
|`weekly`|- [`interval`][interval link]<br/>- [`days`][days link]<br/>- [`firstDayOfWeek`][firstDayOfWeek link] (optional)|An appointment occurs on *days* every *interval* weeks. Example: An appointment occurs on **_Tuesday_ and _Thursday_** every **_2_** weeks.|
|`yearly`|- [`interval`][interval link]<br/>- [`dayOfMonth`][dayOfMonth link]<br/>- [`dayOfWeek`][dayOfWeek link]<br/>- [`weekNumber`][weekNumber link]<br/>- [`month`][month link]|1. An appointment occurs on day *dayOfMonth* of *month* every *interval* years. Example: An appointment occurs on day **_7_** of **_September_** every **_4_** years.<br/>2. An appointment occurs on the *weekNumber* *dayOfWeek* of *month* every *interval* years. Example: An appointment occurs on the **_first_** **_Thursday_** of **_September_** every **_2_** years.|

> [!NOTE]
> If a property is not listed then it is not applicable for that type.

## Set recurrence for a series as the organizer

Along with the recurrence pattern, you also need to determine the start and end dates/times of your appointment series. The [`SeriesTime`][SeriesTime link] object is used to manage that information.

The appointment organizer can set up the appointment series in compose mode only. In the following example, the appointment series is set to occur from 10:30 AM to 11:00 AM PST every Tuesday and Thursday during the period November 2, 2017 to December 2, 2017.

```js
var seriesTimeObject = new Office.SeriesTime();
seriesTimeObject.setStartDate(2017,10,2);
seriesTimeObject.setEndDate(2017,11,2);
seriesTimeObject.setStartTime(10,30);
seriesTimeObject.setDuration(30);

var pattern = {
    "seriesTime": seriesTimeObject,
    "recurrenceType": "weekly",
    "recurrenceProperties": {"interval": 1, "days": ["tue", "thu"], "firstDayOfWeek": "sun"},
    "recurrenceTimeZone": {"name": "Pacific Standard Time"}};

Office.context.mailbox.item.recurrence.setAsync(pattern, callback);

function callback(asyncResult)
{
    console.log(JSON.stringify(asyncResult));
}
```

> [!NOTE]
> Only the appointment organizer can set the recurrence.

## Get recurrence

### Get recurrence as the organizer

In the following example in compose mode, the appointment organizer can get the recurrence object of an appointment series given the series item or an instance of that series.

```js
Office.context.mailbox.item.recurrence.getAsync(callback);

function callback(asyncResult){
    var context = asyncResult.context;
    var recurrence = asyncResult.value;

    if (recurrence == null) {
        console.log("Non-recurring meeting");
        return;
    }

    console.log(JSON.stringify(recurrence));
}
```

The following is an example of the results of the `getAsync` call on a series.

> [!NOTE]
> In this example, the `seriesTimeObject` is a placeholder for the JSON representing the `recurrence.seriesTime` property. You should use the [`SeriesTime`][SeriesTime link] methods to get the recurrence date and time properties.

```js
Recurrence = {
    "recurrenceType": "weekly",
    "recurrenceProperties": {
        "interval": 1,
        "days": ["tue","thu"],
        "firstDayOfWeek": "sun"},
    "seriesTime": {seriesTimeObject},
    "recurrenceTimeZone": {
        "name": "Pacific Standard Time",
        "offset": -480}}
```

### Get recurrence as an attendee

In the following example, an appointment attendee can get the recurrence object of an appointment series given the series, an instance of that series, or a meeting request.

```js
outputRecurrence(Office.context.mailbox.item);

function outputRecurrence(item) {
    var recurrence = item.recurrence;
    var seriesId = item.seriesId;

    if (recurrence == null) {
        console.log("Non-recurring item");
        return;
    }

    console.log(JSON.stringify(recurrence));
}
```

The following is an example of the `item.recurrence` property.

> [!NOTE]
> In this example, the `seriesTimeObject` is a placeholder for the JSON representing the `recurrence.seriesTime` property. You should use the [`SeriesTime`][SeriesTime link] methods to get the recurrence date and time properties.

```js
Recurrence = {
    "recurrenceType": "weekly",
    "recurrenceProperties": {
        "interval": 1,
        "days": ["tue","thu"],
        "firstDayOfWeek": "sun"},
    "seriesTime": {seriesTimeObject},
    "recurrenceTimeZone": {
        "name": "Pacific Standard Time",
        "offset": -480}}
```

### Access the recurrence details

Now that you have the recurrence object (either from the callback or `item.recurrence`), you can get the specific properties you want to work with. For example, to get the series start and end dates and times, you use the methods on the `recurrence.seriesTime` property.

```js
// Get series date and time info
var seriesTime = recurrence.seriesTime;
var startTime = recurrence.seriesTime.getStartTime();
var endTime = recurrence.seriesTime.getEndTime();
var startDate = recurrence.seriesTime.getStartDate();
var endDate = recurrence.seriesTime.getEndDate();
var duration = recurrence.seriesTime.getDuration();

// Get series time zone
var timeZone = recurrence.recurrenceTimeZone;

// Get recurrence properties
var recurrenceProperties = recurrence.recurrenceProperties;

// Get recurrence type
var recurrenceType = recurrence.recurrenceType;
```

## See also

[`RecurrenceChanged` event](/javascript/api/office/office.eventtype)

[getAsync link]: /javascript/api/outlook_1_7/office.recurrence#getasync-options--callback-
[item.recurrence link]: /javascript/api/outlook_1_7/office.item#recurrence
[setAsync link]: /javascript/api/outlook_1_7/office.recurrence#setasync-recurrencepattern--options--callback-

[dayOfMonth link]: /javascript/api/outlook_1_7/office.recurrenceproperties#dayofmonth
[dayOfWeek link]: /javascript/api/outlook_1_7/office.recurrenceproperties#dayofweek
[days link]: /javascript/api/outlook_1_7/office.recurrenceproperties#days
[firstDayOfWeek link]: /javascript/api/outlook_1_7/office.recurrenceproperties#firstdayofweek
[interval link]: /javascript/api/outlook_1_7/office.recurrenceproperties#interval
[month link]: /javascript/api/outlook_1_7/office.recurrenceproperties#month
[weekNumber link]: /javascript/api/outlook_1_7/office.recurrenceproperties#weeknumber

[SeriesTime link]: /javascript/api/outlook_1_7/office.seriestime