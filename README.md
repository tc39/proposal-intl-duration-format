# Proposal for Intl.DurationFormat

**Stage**: 2

**Champion**: [Younies Mahmoud](https://github.com/younies), [Ujjwal Sharma](https://github.com/ryzokuken)

**Authors**: [Younies Mahmoud](https://github.com/younies), [Ujjwal Sharma](https://github.com/ryzokuken)

**Stage 3 Reviewers**
- Michael Ficarra [@michaelficarra](https://github.com/michaelficarra)
- Ron Buckton [@rbuckton](https://github.com/rbuckton)
- Ross Kirsling [@rkirsling](https://github.com/rkirsling)

**[slides](https://docs.google.com/presentation/d/1QmrhwsYwlsfe8FJqgGarCIAySWxeZzDqCrVN3-DWiGk/edit?usp=sharing)**

## Status

This proposal reached Stage 1 at the 2020 Feb TC39 meeting.
This proposal reached Stage 2 at the 2020 June TC39 meeting.

## Overview

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

## Quick Start

```javascript
new Intl.DurationFormat("fr-FR", { style: "long" }).format({
    hours: 1,
    minutes: 46,
    seconds: 40,
});
// => "1 heure, 46 minutes et 40 secondes"
```

## Motivation

* Users need all types of duration formatting depending on the requirements of their application. For example, to show how long a flight takes, the duration should be in Short or Narrow format
  + "1 hr 40 min 60 sec" → Short
  + "1 h 40 m 60 s"  → Narrow

## Requirements & Design

In this section, we are going to illustrate each user needs (requirements) and a design for each need (requirement)

### Input Value

+ Users need to identify how to input a duration value. For example, if a user needs to format `1000 seconds` , how could the user pass the value to the formatting function.

#### Design

* Input value will be an object of type `Temporal.Duration`
* Example: `new DurationFormat().format(Temporal.Duration.from({hours: 3, minutes: 4});`

### Formatting width

Users want to determine several types of the formatting width as following

| Format width | Example               |
|--------------|-----------------------|
| Long         | 1 hour and 50 minutes |
| Short        | 1 hr, 50 min          |
| Narrow       | 1h 50m                |
| Digital      | 1: 50: 00             |

#### Design

+ The user can determine the formatting width using a parameter `style` and the value of this parameter may be one of the following `string` s:
  - `"long"`
  - `"short"`
  - `"narrow"`
  - `"digital"`
+ The width of each field can be set separately in the options, for example: `{ years: "long", months: "short", days: "narrow" }`.

### Supported Duration Units

* Users need the following fields to be supported
  + Years
  + Months
  + Weeks
  + Days
  + Hours
  + Minutes
  + Seconds
  + Milliseconds
  + Microseconds
  + Nanoseconds

#### Design

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

### Determining Duration Units

`DurationFormat` does not do any arithmetic operations on the input implicitly, expecting the user to use functions on the prototype like `Temporal.Duration.prototype.round` in order to ensure that the input is within a certain range if needed.

To avoid accidentally truncating part of the value, `DurationFormat` doesn't allow hiding certain fields, so users expecting a specific result are highly recommended to call `round` in order to make sure that the input only includes necessary units.

#### Design

* Users can determine the time fields in array.
* Example:

``` javascript
    fields: [
        'hour',
        'minute',
        'second'
    ]
```

### Hide zero-value fields

In most cases, users want to avoid displaying zero-value fields. All zero-valued fields are hidden by default. If you specify the style for a specific field, then it is always displayed, but you can override that behavior by also setting the display option for that particular field to `"auto"` again explicitly.

#### Design

For each field `foo`, there is an option `fooDisplay` that is set to `"auto"` by default. If you specify the style for that field by setting the `foo` option, then the default for `fooDisplay` becomes `"always"`.

### Round

* Users wants to decide if they are going to round the smallest field or not.
* for example:
  + Without rounding option
    - 1 hour and 30.6 minutes.
  + With rounding option
    - 1 hour and 31 minutes.

#### Design

* Users could use `Temporal.Duration#round` function.

### Locale aware format

* Users needs the formatting to be dependent on the locale
* For example:
  + en-US
    - 1 hour, 46 minutes and 40 seconds
  + fr-FR
    - 1 heure, 46 minutes et 40 secondes

#### Design

Adding the locale in `string` format as a first argument, or specifying a ranked list of locales as an array of string locale values.

## API design

``` javascript
    new Intl.DurationFormat('en').format(Temporal.Duration.from('PT2H46M40S')); // => 2 hr 46 min 40 sec
    new Intl.DurationFormat('en', {
      hours: 'numeric',
      seconds: 'numeric',
    }).format(Temporal.Duration.from('PT2H40S')); // => 2:00:40
```

## Implementation Status

### V8 Prototypes
Three v8 prototypes (try to use two different possible ICU classes) were made, ALL are:
* Sync with ["Stage 1 Draft / June 1, 2020" version of spec](https://github.com/tc39/proposal-intl-duration-format/commit/fc8ff131cf7e688810b38d7e95d6fa44b1f1964e)
* Flag --harmony_intl_duration_format
* Have not implement any changes not yet spec out in https://tc39.es/proposal-intl-duration-format/ such as
  + hideZeroValued
  + smallestUnit / largestUnit

1. Base the implementation on [icu::MeasureFormat::formatMeasures()](https://unicode-org.github.io/icu-docs/apidoc/dev/icu4c/classicu_1_1MeasureFormat.html#ade104d40578223bd194050914b090904)
   * https://chromium-review.googlesource.com/c/v8/v8/+/2762664
   * Not yet implment formatToParts
   * Need solution of [ICU-21543 "Add methods to return FormattedValue to MeasureFormat "](https://unicode-org.atlassian.net/browse/ICU-21543) to implement formatToParts().
2. Based on [the support of "-and-" unit in LocalizedNumberFormatter](https://unicode-org.github.io/icu-docs/apidoc/dev/icu4c/classicu_1_1number_1_1LocalizedNumberFormatter.html)
   * https://chromium-review.googlesource.com/c/v8/v8/+/2775300
   * Not yet implment formatToParts
   * Not yet implement style:"digital"
   * Need solution of the following to implement formatToParts():
     + [ICU-21544 "unit format in number formatter return U_INTERNAL_PROGRAM_ERROR with "year-and-" (except month) and  "month-and-""](https://unicode-org.atlassian.net/browse/ICU-21544)
     + [ICU-21547 "nextPosition() of FormattedNumber of unit with "-and-" is buggy"](https://unicode-org.atlassian.net/browse/ICU-21547)
3. Based on icu::ListFormatter and icu::number::LocalizedNumberFormatter w/o depend on the "X-and-Y" units.
   * https://chromium-review.googlesource.com/c/v8/v8/+/2776518
   * Implements formatToParts
    * Not yet implement style:"digital"
