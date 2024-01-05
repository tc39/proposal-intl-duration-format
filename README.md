# Proposal for Intl.DurationFormat

**Stage**: 3

**Champion**: [Younies Mahmoud](https://github.com/younies), [Ujjwal Sharma](https://github.com/ryzokuken)

**Authors**: [Younies Mahmoud](https://github.com/younies), [Ujjwal Sharma](https://github.com/ryzokuken)

**Stage 3 Reviewers**
- Michael Ficarra [@michaelficarra](https://github.com/michaelficarra)
- Ron Buckton [@rbuckton](https://github.com/rbuckton)
- Ross Kirsling [@rkirsling](https://github.com/rkirsling)

**Resources**
 - Spec text in [ecmarkup output format](https://tc39.es/proposal-intl-duration-format/)
 - [Stage 2 slides](https://docs.google.com/presentation/d/1QmrhwsYwlsfe8FJqgGarCIAySWxeZzDqCrVN3-DWiGk/edit?usp=sharing)
 - [Stage 3 slides](https://ryzokuken.dev/slides/2021-10-df#/)
 - [Polyfill from formatjs](https://formatjs.io/docs/polyfills/intl-durationformat)


## Status

- This proposal reached Stage 1 at the 2020 Feb TC39 meeting.
- This proposal reached Stage 2 at the 2020 June TC39 meeting.
- This proposal reached Stage 3 at the 2021 October TC39 meeting.

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
  + "1h 40m 60s"  → Narrow

## Requirements & Design

In this section, we are going to illustrate each user needs (requirements) and a design for each need (requirement)

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

* Duration objects support the same fields as in [ `Temporal.Duration` ](https://github.com/tc39/proposal-temporal/blob/main/docs/duration.md), which contains:
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

### Input Value

+ Users need to identify how to input a duration value. For example, if a user needs to format `1000 seconds` , how could the user pass the value to the formatting function.

#### Design

* Input value will be a duration object, containing fields for some number of the supported duration units fields.
* Example: `new DurationFormat().format({hours: 3, minutes: 4});`

### Formatting width

Users want to determine several types of the formatting width as following

| Format width | Example               |
|--------------|-----------------------|
| Long         | 1 hour and 50 minutes |
| Short        | 1 hr, 50 min          |
| Narrow       | 1h 50m                |
| Digital      | 1:50:00               |

#### Design

+ The user can determine the formatting width using a parameter `style` and the value of this parameter may be one of the following strings:
  - `"long"`
  - `"short"`
  - `"narrow"`
  - `"digital"`
+ The width of each field can be set separately in the options, for example: `{ years: "long", months: "short", days: "narrow" }`.

### Determining Duration Units

`DurationFormat` does not do any arithmetic operations nor rounding on the input implicitly.
Instead, the user must edit the input to ensure the input is within the desired range.

To avoid accidentally omitting part of the duration, `DurationFormat` always outputs all nonzero fields (except sub-second fields truncated by `fractionalDigits`).
Callers who want to omit nonzero fields (for example, only showing the date or time portion of the duration) should edit the input duration.

#### Design

* Durations are stored in duration objects containing the fields above. 
* Example:

``` javascript
    const duration = { hours: 1, minutes: 2, seconds: 33 };
```

### Hide zero-value fields

In most cases, users want to avoid displaying zero-value fields. All zero-valued fields are hidden by default. If you specify the style for a specific field, then it is always displayed, but you can override that behavior by also setting the display option for that particular field to `"auto"` again explicitly.

#### Design

For each field `foo`, there is an option `fooDisplay` that is set to `"auto"` by default. Setting that option to `"always"` causes that field to be displayed even if it is zero; for example, to always show the "day" field, set `{ dayDisplay: "always" }`. If you specify the style for that field by setting the `foo` option, then the default for `fooDisplay` becomes `"always"`; for example, `{ day: "short" }` implies `{ day: "short", dayDisplay: "always" }`.

### Locale-aware format

* Users needs the formatting to be dependent on the locale
* For example:
  + en-US
    - 1 hour, 46 minutes and 40 seconds
  + fr-FR
    - 1 heure, 46 minutes et 40 secondes

#### Design

Adding the locale in string format as a first argument, or specifying a ranked list of locales as an array of string values.

### Display fractional values

Sometimes it is desirable to display the smallest sub-second unit not by itself but as a fraction of the immediately larger unit.

#### Design

We allow users to specify a `fractionalDigits` option that will display the smallest sub-second unit with display set to `"auto"` as a fraction of the previous unit if it is non-zero and if these values have style set to `"numeric"`. The number of digits used will be the value passed to this option. By default `fractionalDigits` is undefined. In this case, exactly as many fractional digits as needed to display the whole duration are included. If rounding is necessary, we round toward 0.

#### Example

```javascript

const duration = { seconds: 12, milliseconds: 345, microseconds: 600 } ;

new Intl.DurationFormat('en', { style: "digital", fractionalDigits: 2 }).format(duration); 
// "0:00:12.35"

new Intl.DurationFormat('en', { seconds: "numeric", fractionalDigits: 2 }).format(duration); 
// "12.35"

> new Intl.DurationFormat('en', { seconds: "numeric", fractionalDigits: 5 }).format(duration); 
// "12.34560"

> new Intl.DurationFormat('en', { seconds: "numeric"}).format(duration); 
// "12.3456"

```

## API design

### Constructor

#### Syntax

```javascript
new Intl.DurationFormat(locales, options)
```

#### Parameters

* `locales: Array<string> | string`: A locale string or a list of locale strings in decreasing order of preference.
* `options?: object`: An object for configuring the behavior of the instance. It may have some or all of the following properties:
  * `localeMatcher: "best fit" | "lookup"`: A string denoting which locale matching algorithm to use. Defaults to `"best fit"`.
  * `numberingSystem: string`: A string containing the name of the numbering system to be used for number formatting.
  * `style: "long" | "short" | "narrow" | "digital"`: The base style to be used for formatting. This can be overriden per-unit by setting the more granular options. Defaults to `"short"`.
  * `years: "long" | "short" | "narrow"`: The style to be used for formatting years.
  * `yearsDisplay: "always" | "auto"`: Whether to always display years, or only if nonzero.
  * `months: "long" | "short" | "narrow"`: The style to be used for formatting months.
  * `monthsDisplay: "always" | "auto"`: Whether to always display months, or only if nonzero.
  * `weeks: "long" | "short" | "narrow"`: The style to be used for formatting weeks.
  * `weeksDisplay: "always" | "auto"`: Whether to always display weeks, or only if nonzero.
  * `days: "long" | "short" | "narrow"`: The style to be used for formatting days.
  * `daysDisplay: "always" | "auto"`: Whether to always display days, or only if nonzero.
  * `hours: "long" | "short" | "narrow" | "numeric" | "2-digit"`: The style to be used for formatting hours.
  * `hoursDisplay: "always" | "auto"`: Whether to always display hours, or only if nonzero.
  * `minutes: "long" | "short" | "narrow" | "numeric" | "2-digit"`: The style to be used for formatting minutes.
  * `minutesDisplay: "always" | "auto"`: Whether to always display minutes, or only if nonzero.
  * `seconds: "long" | "short" | "narrow" | "numeric" | "2-digit"`: The style to be used for formatting seconds.
  * `secondsDisplay: "always" | "auto"`: Whether to always display seconds, or only if nonzero.
  * `milliseconds: "long" | "short" | "narrow" | "numeric"`: The style to be used for formatting milliseconds.
  * `millisecondsDisplay: "always" | "auto"`: Whether to always display milliseconds, or only if nonzero.
  * `microseconds: "long" | "short" | "narrow" | "numeric"`: The style to be used for formatting microseconds.
  * `microsecondsDisplay: "always" | "auto"`: Whether to always display microseconds, or only if nonzero.
  * `nanoseconds: "long" | "short" | "narrow" | "numeric"`: The style to be used for formatting nanoseconds.
  * `nanosecondsDisplay: "always" | "auto"`: Whether to always display nanoseconds, or only if nonzero.
  * `fractionalDigits: number`: How many fractional digits to display in the output. 
    Additional decimal places will be truncated towards zero.
    (`Temporal.Duration.prototype.round` can be used to obtain different rounding behavior.)
    Normally this option applies to fractional seconds, but this option actually applies to the largest seconds-or-smaller unit that uses the `"numeric"` or `"2-digit"` style.
    For example, if options are `{ seconds: "narrow", milliseconds: "numeric", fractionalDigits: 4}` then the output is "12.3456 seconds".
    If this option is omitted, only nonzero decimals will be displayed and trailing zeroes will be omitted.

##### Default values

* The per-unit style options default to the value of `style` for all styles except `"digital"`, for which units years till days default to `"short"` and hours till nanoseconds default to `"numeric"`.
* If `style` is `undefined`, all values default to `"short"`
* The per-unit display options default to `"auto"` if the corresponding style option is `undefined` and `"always"` otherwise.

#### Notes

* Some locales may share the same representation between `"long"` and `"short"` or between `"narrow"` and `"short"`.  Others may use different representations for each one, e.g. "3 seconds", "3 secs", "3s".
* Any unit with the style `"numeric"` preceded by a unit of style `"numeric"` or `"2-digit"` should act like the style `"2-digit"` was used instead.  For example, `{hours: 'numeric', minutes: 'numeric'}` can produce output like "3:08".

### `Intl.DurationFormat#format`

#### Syntax

```javascript
new Intl.DurationFormat('en').format(duration)
```

#### Parameters

* `duration` (`Temporal.Duration` | `string` | `object`): The duration to be formatted. This could either be a `Temporal.Duration` object or a string or options bag that can be converted into one.

#### Returns

A `string` containing the formatted duration.

### `Intl.DurationFormat#formatToParts`

#### Syntax

```javascript
new Intl.DurationFormat('en').formatToParts(duration)
```

#### Parameters

* `duration` (`Temporal.Duration` | `string` | `object`): The duration to be formatted. This could either be a `Temporal.Duration` object or a string or options bag that can be converted into one.

#### Returns

An `Array<{type: string, value: string}>` containing the formatted duration in parts.

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
