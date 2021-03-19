# Proposal for Intl.DurationFormat

**Stage**: 2

**Champion**: [Younies Mahmoud](https://github.com/younies), [Ujjwal Sharma](https://github.com/ryzokuken)

**Authors**: [Younies Mahmoud](https://github.com/younies), [Ujjwal Sharma](https://github.com/ryzokuken)

**Stage 3 Reviewers**

- Michael Ficarra [@michaelficarra](https://github.com/michaelficarra)
- Ron Buckton [@rbuckton](https://github.com/rbuckton)
- Ross Kirsling [@rkirsling](https://github.com/rkirsling)

**[slide](https://docs.google.com/presentation/d/1QmrhwsYwlsfe8FJqgGarCIAySWxeZzDqCrVN3-DWiGk/edit?usp=sharing)**

This proposal reached Stage 1 at the 2020 Feb TC39 meeting.

# Overview

* Time Duration is how long something lasts, from the start to end. It can be represented by a single time unit or multiple ones.
  + For example,
    - 10000 seconds
    - 2 hours 46 minutes 40 seconds
* Every locale has its own way to format duration.
  + For example:
    - en-US: 1 hour, 46 minutes and 40 seconds
    - fr-FR: 1 heure, 46 minutes et 40 secondes
* There are multiple widths for the time duration.
  + For example, wide and short
    - 1 hour, 46 minutes and 40 seconds → Wide
    - 1 hr, 46 min, 40 sec → Short

# Quick Start Example

```javascript
new Intl.DurationFormat("fr-FR", { style: "long" }).format({
    hours: 1,
    minutes: 46,
    seconds: 40,
});
// "1 heure, 46 minutes et 40 secondes"
```

# V8 Prototypes
Two v8 prototypes (try to use two different possible ICU classes) were made, BOTH are
* Sync with ["Stage 1 Draft / June 1, 2020" version of spec](https://github.com/tc39/proposal-intl-duration-format/commit/fc8ff131cf7e688810b38d7e95d6fa44b1f1964e)
* Flag --harmony_intl_duration_format
* Not yet implment formatToParts
* Have not implement any changes not yet spec out in https://tc39.es/proposal-intl-duration-format/ such as 
  + hideZeroValued
  + smallestUnit / largestUnit

1. Base the implementation on MeasureFormat
   * https://chromium-review.googlesource.com/c/v8/v8/+/2762664
2. Based on LocalizedNumberFormatter
   * https://chromium-review.googlesource.com/c/v8/v8/+/2775300
   * Not yet implement style:"dotted"
   
# Motivation

* Users need all types of duration format depending on the requirements of their application. For example, to show how long a flight takes, the duration should be in Short or Narrow format
  + 1 hr 40 min 60 sec → Short
  + 1 h 40 m 60 sec  → Narrow

# Requirements & Design

In this section, we are going to illustrate each user needs (requirements) and a design for each need (requirement)

## Input Value

  + Users need to identify how to input a duration value. For example, if a user needs to format `1000 seconds` , how could the user pass the value to the formatting function.

### Design

* Input value will be an object of type `Temporal.Duration`
* Example:
    - `const d = new DurationFormat();`
    - `d.format(Temporal.Duration.from({hours: 3, minutes: 4});`

## Formatting width

Users want to determine several types of the formatting width as following

| Format width | Example               |
|--------------|-----------------------|
| Wide         | 1 hour and 50 minutes |
| Short        | 1 hr, 50 min          |
| Narrow       | 1h 50 m               |
| Dotted       | 1: 50: 00             |

### Design

  + The user can determine the formatting width using a parameter `style` and the value of this parameter may be one of the following `string` s:
    - `"wide"`
    - `"short"`
    - `"narrow"`
    - `"dotted"`

## Supported Time Fields

* Users need the following fields to be supported
  + Year
  + Month
  + Week
  + Day
  + Hour
  + Minute
  + Second
  + Millisecond
  + Microsecond
  + Nanosecond

### Design

* We are going to support the same fields as in [ `Temporal.Duration` ](https://github.com/tc39/proposal-temporal/blob/main/docs/duration.md), which contains:
  + years
  + months
  + weeks
  + days
  + hours
  + minutes
  + seconds
  + milliseconds
  + microseconds
  + nanoseconds

## Determining Time Fields

* Users need to determine the time fields,
* For example: 6059 seconds
  + Hours, Minutes, Seconds
    - 1 hours, 40 minutes and  59 seconds
  + Minutes and Seconds
    - 100 minutes and 59 seconds.

### Design

* Users can determine the time fields in array.
* Example:

``` javascript
    fields: [
        'hour',
        'minute',
        'second'
    ]
```

#### NOTE (For stage 3)

* We are discussing to use `largestField` and `smallestField` options to determine the time fields instead of an array.

## Hide zero-value fields

* Users needs to hide zero-value fields depends in some criteria
  + Hide all zero-value fields
    - Hours, Minutes, Seconds (without the `hide` option)
      * 1 hours, 0 minutes and 59 seconds.
    - Hours, Minutes, Seconds (with the `hide` option)
      * 1 hours and 59 seconds.
  + Hide-upper-limit with zero-valued.
    - Hours, Minutes, Seconds, MilliSeconds (without the `hide-upper-limit` option)
      * 0 hr, 20 min, 0 sec, 100 msec .
    - Hours, Minutes, Seconds MilliSeconds(with the `hide-upper-limit` option)
      * 20 min, 0 sec, 100 msec.
  + Hide-lower-limit with zero-valued.

### Design

* Users can set the `hideZeroValues` parameter  by one of the following values:
  + `"all"` // Hide all the fields that have a zero-valued.
  + `"leadingAndTrailing"` // Hide all the zero fields in the leading or the training
  + `"leadingOnly"`
  + `"trailingOnly"`
  + `"none"` // Do not hide any zero-valued fields.

* Default Value: `"none"`

## Round

* Users wants to decide if they are going to round the smallest field or not.
* for example:
  + Without rounding option
    - 1 hour and 30.6 minutes.
  + With rounding option
    - 1 hour and 31 minutes.

### Design

* Users could use `Temporal.Duration#round` function.

## Locale aware format

* Users needs the formatting to be dependent on the locale
* For example:
  + en-US
    - 1 hour, 46 minutes and 40 seconds
  + fr-FR
    - 1 heure, 46 minutes et 40 secondes

### Design

Adding the locale in `string` format as a first argument.

# API design

``` javascript
    const formatter =
        new Intl.DurationFormat('en-US', {
            fields: [
                'hour',
                'minute',
                'second'
            ],
            style: 'short',
            hideZeroValues: 'none',
        });

    const duration =
        Temporal.Duration.from({
            hours: 2,
            minutes: 46,
            seconds: 40
        });

    console.log(formatter.format(duration));
    // output: 2 hr 46 min 40 sec
```
