---
description: >-
  Describes the placeholders you can use when formatting dates in PHP or with
  Carbon
---

# PHP DateTime formatting cribsheet

Usage with php datetime object or [Carbon](https://carbon.nesbot.com/)

{% tabs %}
{% tab title="Time" %}
### Hour

| Character | Description                                     | Example           |
| :-------: | ----------------------------------------------- | ----------------- |
|     H     | 24-hour format of an hour with leading zeros    | _00_ through _23_ |
|     G     | 24-hour format of an hour without leading zeros | _0_ through _23_  |
|     h     | 12-hour format of an hour with leading zeros    | _01_ through 12   |
|     g     | 12-hour format of an hour without leading zeros | _1_ through _12_  |
|     a     | Lowercase Ante meridiem and Post meridiem       | `am` or `pm`      |
|     A     | Uppercase Ante meridiem and Post meridiem       | `AM` or `PM`      |

### Minutes and Seconds

| Character | Description                | Example           |
| :-------: | -------------------------- | ----------------- |
|     i     | Minutes with leading zeros | _00_ through _59_ |
|     s     | Seconds with leading zeros | _00_ through _59_ |
|     u     | Microseconds               | eg _123456_       |
|     v     | Milliseconds               | eg _654_          |
{% endtab %}

{% tab title="Date" %}
### Day

| Character | Description                                                    | Example                                                  |
| :-------: | -------------------------------------------------------------- | -------------------------------------------------------- |
|     d     | Day of the month, 2 digits with leading zeros                  | _01_ through _31_                                        |
|     j     | Day of the month without leading zeros                         | _1_ through _31_                                         |
|     D     | Textual representation of a day, three letters                 | _Mon_ through _Sun_\*\*                                  |
|     l     | (lowercase L) Full textual representation of a day of the week | <p><em>Monday</em> through </p><p><em>Sunday</em> **</p> |
|     N     | ISO-8601 numeric representation of the day of the week         | <p><em>1</em> through <em>7</em> </p><p>(mon=1)</p>      |
|     S     | English ordinal suffix for the day of the month                | _st_, _nd_, _rd_, or _th_                                |
|     w     | Numeric representation of the weekday                          | _0_ (sun) through _6_                                    |
|     z     | The day of the year (zero index for Jan 1)                     | _0_ through _365_                                        |

### Week

| Character | Description                                            | Example                                                        |
| :-------: | ------------------------------------------------------ | -------------------------------------------------------------- |
|     W     | ISO-8601 week number of year, weeks starting on Monday | <p>Example: <em>42</em> </p><p>(the 42nd week in the year)</p> |

### Month

| Character | Description                                                        | Example                      |
| --------- | ------------------------------------------------------------------ | ---------------------------- |
| m         | Numeric representation of a month, with leading zeros              | _01_ through _12_            |
| n         | Numeric representation of a month, without leading zeros           | _1_ through _12_             |
| M         | A short textual representation of a month, three letters	          | _Jan_ through _Dec \*\*_     |
| F         | A full textual representation of a month, such as January or March | _January_ through _December_ |
| t         | The number of days in the given month                              | _28_ through _31_            |

### Year

| Character | Description                                                                                                                                                         | Example                                    |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| Y         | a four digit numeric representation of the year                                                                                                                     | _2020_ or _1999_                           |
| y         | a two digit numeric representation of the year                                                                                                                      | _20_ or _99_                               |
| L         | Whether it is a leap year                                                                                                                                           | <p>1 for leap year, </p><p>0 otherwise</p> |
| o         | ISO-8601 week-numbering year. This has the same value as Y, except that if the ISO week number (W) belongs to the previous or next year, that year is used instead. | 2020 or 1999                               |

\*\* can be varied through localisation
{% endtab %}

{% tab title="Other" %}
| Character | Description                                                             | Example                           |
| :-------: | ----------------------------------------------------------------------- | --------------------------------- |
|     c     | ISO 8601 Full Date Time                                                 | `2020-06-26T13:40:10+00:00`       |
|     r     | [ RFC 2822](http://www.faqs.org/rfcs/rfc2822) formatted date            | `Fri, 26 Jun 2020 13:41:32 +0100` |
|     U     | Seconds since the Unix Epoch (commonly referred to as timestamp format) | _1593175400_                      |
{% endtab %}
{% endtabs %}
