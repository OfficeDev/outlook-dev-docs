---
title: Recurrence rules in Outlook REST APIs | Microsoft Docs
description: Learn how to use recurrence rules in the Outlook REST APIs to create recurring appointments and meetings.
author: jasonjoh

ms.topic: article
ms.technology: graph
ms.date: 08/25/2017
ms.author: jasonjoh
---

# Working with recurrence rules in the Outlook REST APIs

Recurring events are an important part of Outlook calendaring. Whether it's a weekly one-on-one meeting with your manager, or a division-wide review meeting that happens on the second Tuesday of each month, recurring events make it easy to create the event once, and let the server fill in the rest of the series.

The key bit of information that allows recurring events to "expand" into individial occurrences is the recurrence rule. The rule specifies both how often an event repeats, and for how long. The Outlook REST APIs model recurrence rules in the `recurrence` property of the [event resource](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/event). Each `recurrence` is made up of two parts: the recurrence pattern (how often), and the recurrence range (for how long).

## Patterns

The first part of a recurrence is the pattern. This specifies how often the event repeats. For example, an event could repeat "every 3 days", "every Thursday", or "on July 22 every year". A pattern is represented in the API by the [recurrencePattern resource](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/resources/recurrencepattern).

Depending on the type of pattern, certain fields of the `recurrencePattern` are required, optional, or ignored.

> [!NOTE]
> Even if a field is ignored, it is still validated. If a field has a set list of possible values, using a value outside the allowed set will cause an error, even if that field is ignored.

Let's take a look at each of the possible pattern types.

### Daily

The daily recurrence pattern is used when an event should repeat based on a number of days between each occurrence.

#### Relevant properties

| Property | Relevance | Description |
|----------|-----------|-------------|
| `interval` | **Required** | Specifies the number of days between each occurrence. |
| `type` | **Required** | Must be set to `daily`. |

#### Examples

- Repeat this event every day
    ```json
    "pattern": {
      "type": "daily",
      "interval": 1
    }
    ```
- Repeat this event every three days
    ```json
    "pattern": {
      "type": "daily",
      "interval": 3
    }
    ```

### Weekly

The weekly recurrence pattern is used when an event should repeat on the same day or days of the week, based on the number of weeks between each set of occurrences.

#### Relevant properties

| Property | Relevance | Description |
|----------|-----------|-------------|
| `daysOfWeek` | **Required** | Specifies on which days of the week the event occurs. |
| `firstDayOfWeek` | **Optional** | Specifies which day is considered the first day of the week. Default value: `Sunday`. |
| `interval` | **Required** | Specifies the number of weeks between each set of occurrences. |
| `type` | **Required** | Must be set to `weekly`. |

#### Examples

- Repeat this event every Thursday
    ```json
    "pattern": {
      "type": "weekly",
      "interval": 1,
      "daysOfWeek": [ "Thursday" ]
    }
    ```
- Repeat this event every other Monday and Tuesday
    ```json
    "pattern": {
      "type": "weekly",
      "interval": 2,
      "daysOfWeek": [
        "Monday",
        "Tuesday"
      ]
    }
    ```

### Absolute monthly

The absolute monthly pattern is used when an event should repeat on the same day of the month (e.g. the 15th), based on the number of months between each occurrence.

#### Relevant properties

| Property | Relevance | Description |
|----------|-----------|-------------|
| `dayOfMonth` | **Required** | Specifies on which day of the month the event occurs. |
| `interval` | **Required** | Specifies the number of months between each occurrence. |
| `type` | **Required** | Must be set to `absoluteMonthly`. |

#### Examples

- Repeat this event on the 15th of every month
    ```json
    "pattern": {
      "type": "absoluteMonthly",
      "interval": 1,
      "dayOfMonth": 15
    }
    ```
- Repeat this event quarterly (every 3 months) on the 7th
    ```json
    "pattern": {
      "type": "absoluteMonthly",
      "interval": 3,
      "dayOfMonth": 7
    }
    ```

### Relative monthly

The relative monthly pattern is used when an event should repeat on the same day of the week in the same relative position in the month, based on the number of months between each occurrence. For example, "every second Wednesday of the month".

#### Relevant properties

| Property | Relevance | Description |
|----------|-----------|-------------|
| `daysOfWeek` | **Required** | Specifies on which days of the week the event can occur. Relative monthly events only occur once per month, so if more than one value is specified, the event falls on the first day that satisfies the pattern. |
| `index` | **Optional** | Specifies on which instance of the allowed days specified in `daysOfsWeek` the event occurs, counted from the first instance in the month. Default value: `first`. |
| `interval` | **Required** | Specifies the number of months between each occurrence. |
| `type` | **Required** | Must be set to `relativeMonthly`. |

#### Examples

- Repeat this event on the second Wednesday of every month
    ```json
    "pattern": {
      "type": "relativeMonthly",
      "interval": 1,
      "daysOfWeek": [ "Wednesday" ],
      "index": "second"
    }
    ```
- Repeat this event on the first Thursday or Friday of every month
    ```json
    "pattern": {
      "type": "relativeMonthly",
      "interval": 1,
      "daysOfWeek": [ "Thursday", "Friday" ],
      "index": "second"
    }
    ```

### Absolute yearly

The absolute yearly pattern is used when an event should repeat on the same month and day (e.g. April 15th), based on the number of years between each occurrence.

#### Relevant properties

| Property | Relevance | Description |
|----------|-----------|-------------|
| `dayOfMonth` | **Required** | Specifies on which day of the month the event occurs. |
| `month` | **Required** | Specifies in which month the event occurs. |
| `interval` | **Required** | Specifies the number of years between each occurrence. |
| `type` | **Required** | Must be set to `absoluteYearly`. |

#### Example

- Repeat this event on April 15 every year
    ```json
    "pattern": {
      "type": "absoluteYearly",
      "interval": 1,
      "dayOfMonth": 15,
      "month": 4
    }
    ```

### Relative yearly

The relative yearly pattern is used when an event should repeat on the same day of the week in the same relative position in a specific month, based on the number of years between each occurrence. For example, "every last Wednesday of November".

#### Relevant properties

| Property | Relevance | Description |
|----------|-----------|-------------|
| `daysOfWeek` | **Required** | Specifies on which days of the week the event can occur. Relative yearly events only occur once per year, so if more than one value is specified, the event falls on the first day that satisfies the pattern. |
| `index` | **Optional** | Specifies on which instance of the allowed days specified in `daysOfsWeek` the event occurs, counted from the first instance in the month. Default value: `first`. |
| `month` | **Required** | Specifies in which month the event occurs. |
| `interval` | **Required** | Specifies the number of years between each occurrence. |
| `type` | **Required** | Must be set to `relativeMonthly`. |

#### Examples

- Repeat this event on the last Wednesday of November every year
    ```json
    "pattern": {
      "type": "relativeYearly",
      "interval": 1,
      "daysOfWeek": [ "Wednesday" ],
      "index": "last",
      "month": 11
    }
    ```