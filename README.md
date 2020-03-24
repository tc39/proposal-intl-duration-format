# Proposal for Int Duration Format

**Stage**: 1

**Champion**: Younies Mahmoud [@younies](https://github.com/younies)

**Authors**: Younies Mahmoud [@younies](https://github.com/younies)

**[slide](https://docs.google.com/presentation/d/1IfyFfYBROZJkODWWHudBKuGkKvTzN_Pwn2iq1ZM4_JY/edit?usp=sharing)**

This proposal reached Stage 1 at the 2020 Feb TC39 meeting.

## Overview

* Time Duration is how long something lasts, from the start to end. It can be represented by a single time unit or multiple ones. 
  - For example,
    - 10000 seconds
    - 2 hours 46 minutes 40 seconds

* Every locale has its own way to format duration. 
  - For example:
    - en-US: 1 hour, 46 minutes and 40 seconds
    - fr-FR: 1 heure, 46 minutes et 40 secondes

* There are multiple widths for the time duration.
  - For example, wide and short
    - 1 hour, 46 minutes and 40 seconds → Wide
    - 1 hr, 46 min, 40 sec → Short

## Motivation

* Users need all types of duration format depending on the requirements of their application. For example, in the airlines websites, to show how long a flight takes, the duration should be in Short or Narrow format
  - 1 hr 40 min 60 sec → Short 
  - 1 h 40 m 60 sec  → Narrow

* Furthermore, for TTS (text to speech) applications, Wide format is needed
  - 1 hour, 40 minutes and 60 seconds.


## API design

  ```javascript
    const formatter =  new Intl.DurationFormatter('en-US' , { style: 'short', fields: 'hours-minutes-seconds'});

    const duration =  Temporal.Duration.from({
                        hours: 2, minutes: 46, seconds: 40
                      });

    console.log(formatter.format(duration));
    // output: 2 hr 46 min 40 sec
  ```
