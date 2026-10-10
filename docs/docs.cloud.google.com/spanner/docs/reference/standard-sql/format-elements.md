---
name: documents/docs.cloud.google.com/spanner/docs/reference/standard-sql/format-elements
uri: https://docs.cloud.google.com/spanner/docs/reference/standard-sql/format-elements
title: Format elements in GoogleSQL
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

GoogleSQL for Spanner supports the following format elements.

## Format elements for date and time parts

Many GoogleSQL parsing and formatting functions rely on a format string to describe the format of parsed or formatted values. A format string represents the textual form of date and time and contains separate format elements that are applied left-to-right.

These functions use format strings:

- [`FORMAT_DATE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions#format_date)
- [`FORMAT_TIMESTAMP`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#format_timestamp)
- [`PARSE_DATE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions#parse_date)
- [`PARSE_TIMESTAMP`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#parse_timestamp)

Format strings generally support the following elements:

| Format element | Type               | Description                                                                                                                                                                                                                                                                                                                                | Example                    |
|----------------|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|
| `%A`           | `DATE` `TIMESTAMP` | The full weekday name (English).                                                                                                                                                                                                                                                                                                           | `Wednesday`                |
| `%a`           | `DATE` `TIMESTAMP` | The abbreviated weekday name (English).                                                                                                                                                                                                                                                                                                    | `Wed`                      |
| `%B`           | `DATE` `TIMESTAMP` | The full month name (English).                                                                                                                                                                                                                                                                                                             | `January`                  |
| `%b`           | `DATE` `TIMESTAMP` | The abbreviated month name (English).                                                                                                                                                                                                                                                                                                      | `Jan`                      |
| `%C`           | `DATE` `TIMESTAMP` | The century (a year divided by 100 and truncated to an integer) as a decimal number (00-99).                                                                                                                                                                                                                                               | `20`                       |
| `%c`           | `TIMESTAMP`        | The date and time representation (English).                                                                                                                                                                                                                                                                                                | `Wed Jan 20 21:47:00 2021` |
| `%D`           | `DATE` `TIMESTAMP` | The date in the format %m/%d/%y.                                                                                                                                                                                                                                                                                                           | `01/20/21`                 |
| `%d`           | `DATE` `TIMESTAMP` | The day of the month as a decimal number (01-31).                                                                                                                                                                                                                                                                                          | `20`                       |
| `%e`           | `DATE` `TIMESTAMP` | The day of month as a decimal number (1-31); single digits are preceded by a space.                                                                                                                                                                                                                                                        | `20`                       |
| `%F`           | `DATE` `TIMESTAMP` | The date in the format %Y-%m-%d. When formatting, this element removes leading zeros from the year.                                                                                                                                                                                                                                        | `2021-01-20`               |
| `%G`           | `DATE` `TIMESTAMP` | The [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) year with century as a decimal number. Each ISO year begins on the Monday before the first Thursday of the Gregorian calendar year. Note that %G and %Y may produce different results near Gregorian year boundaries, where the Gregorian year and ISO year can diverge.            | `2021`                     |
| `%g`           | `DATE` `TIMESTAMP` | The [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) year without century as a decimal number (00-99). Each ISO year begins on the Monday before the first Thursday of the Gregorian calendar year. Note that %g and %y may produce different results near Gregorian year boundaries, where the Gregorian year and ISO year can diverge. | `21`                       |
| `%H`           | `TIMESTAMP`        | The hour (24-hour clock) as a decimal number (00-23).                                                                                                                                                                                                                                                                                      | `21`                       |
| `%h`           | `DATE` `TIMESTAMP` | The abbreviated month name (English).                                                                                                                                                                                                                                                                                                      | `Jan`                      |
| `%I`           | `TIMESTAMP`        | The hour (12-hour clock) as a decimal number (01-12).                                                                                                                                                                                                                                                                                      | `09`                       |
| `%j`           | `DATE` `TIMESTAMP` | The day of the year as a decimal number (001-366).                                                                                                                                                                                                                                                                                         | `020`                      |
| `%k`           | `TIMESTAMP`        | The hour (24-hour clock) as a decimal number (0-23); single digits are preceded by a space.                                                                                                                                                                                                                                                | `21`                       |
| `%l`           | `TIMESTAMP`        | The hour (12-hour clock) as a decimal number (1-12); single digits are preceded by a space.                                                                                                                                                                                                                                                | `9`                        |
| `%M`           | `TIMESTAMP`        | The minute as a decimal number (00-59).                                                                                                                                                                                                                                                                                                    | `47`                       |
| `%m`           | `DATE` `TIMESTAMP` | The month as a decimal number (01-12).                                                                                                                                                                                                                                                                                                     | `01`                       |
| `%n`           | All                | A newline character.                                                                                                                                                                                                                                                                                                                       |                            |
| `%P`           | `TIMESTAMP`        | When formatting, this is either am or pm. This can't be used with parsing. Instead, use %p.                                                                                                                                                                                                                                                | `pm`                       |
| `%p`           | `TIMESTAMP`        | When formatting, this is either AM or PM. When parsing, this can be used with am, pm, AM, or PM.                                                                                                                                                                                                                                           | `PM`                       |
| `%Q`           | `DATE` `TIMESTAMP` | The quarter as a decimal number (1-4).                                                                                                                                                                                                                                                                                                     | `1`                        |
| `%R`           | `TIMESTAMP`        | The time in the format %H:%M.                                                                                                                                                                                                                                                                                                              | `21:47`                    |
| `%S`           | `TIMESTAMP`        | The second as a decimal number (00-60).                                                                                                                                                                                                                                                                                                    | `00`                       |
| `%s`           | `TIMESTAMP`        | The number of seconds since 1970-01-01 00:00:00. Always overrides all other format elements, independent of where %s appears in the string. If multiple %s elements appear, then the last one takes precedence.                                                                                                                            | `1611179220`               |
| `%T`           | `TIMESTAMP`        | The time in the format %H:%M:%S.                                                                                                                                                                                                                                                                                                           | `21:47:00`                 |
| `%t`           | All                | A tab character.                                                                                                                                                                                                                                                                                                                           |                            |
| `%U`           | `DATE` `TIMESTAMP` | The week number of the year (Sunday as the first day of the week) as a decimal number (00-53).                                                                                                                                                                                                                                             | `03`                       |
| `%u`           | `DATE` `TIMESTAMP` | The weekday (Monday as the first day of the week) as a decimal number (1-7).                                                                                                                                                                                                                                                               | `3`                        |
| `%V`           | `DATE` `TIMESTAMP` | The [ISO 8601](https://en.wikipedia.org/wiki/ISO_week_date) week number of the year (Monday as the first day of the week) as a decimal number (01-53). If the week containing January 1 has four or more days in the new year, then it's week 1; otherwise it's week 53 of the previous year, and the next week is week 1.                 | `03`                       |
| `%W`           | `DATE` `TIMESTAMP` | The week number of the year (Monday as the first day of the week) as a decimal number (00-53).                                                                                                                                                                                                                                             | `03`                       |
| `%w`           | `DATE` `TIMESTAMP` | The weekday (Sunday as the first day of the week) as a decimal number (0-6).                                                                                                                                                                                                                                                               | `3`                        |
| `%X`           | `TIMESTAMP`        | The time representation in HH:MM:SS format.                                                                                                                                                                                                                                                                                                | `21:47:00`                 |
| `%x`           | `DATE` `TIMESTAMP` | The date representation in MM/DD/YY format.                                                                                                                                                                                                                                                                                                | `01/20/21`                 |
| `%Y`           | `DATE` `TIMESTAMP` | The year with century as a decimal number. When formatting, this element removes leading zeros. To retain leading zeros for four-character years (0001 ... 9999), use `%E4Y` .                                                                                                                                                             | `2021`                     |
| `%y`           | `DATE` `TIMESTAMP` | The year without century as a decimal number (00-99), with an optional leading zero. Can be mixed with %C. If %C isn't specified, years 00-68 are 2000s, while years 69-99 are 1900s.                                                                                                                                                      | `21`                       |
| `%Z`           | `TIMESTAMP`        | The time zone name.                                                                                                                                                                                                                                                                                                                        | `UTC-5`                    |
| `%z`           | `TIMESTAMP`        | The offset from the Prime Meridian in the format +HHMM or -HHMM as appropriate, with positive values representing locations east of Greenwich.                                                                                                                                                                                             | `-0500`                    |
| `%%`           | All                | A single % character.                                                                                                                                                                                                                                                                                                                      | `%`                        |
| `%Ez`          | `TIMESTAMP`        | RFC 3339-compatible numeric time zone (+HH:MM or -HH:MM).                                                                                                                                                                                                                                                                                  | `-05:00`                   |
| `%E<number>S`  | `TIMESTAMP`        | Seconds with \<number\> digits of fractional precision.                                                                                                                                                                                                                                                                                    | `00.000 for %E3S`          |
| `%E*S`         | `TIMESTAMP`        | Seconds with full fractional precision (a literal '\*').                                                                                                                                                                                                                                                                                   | 00.123456789               |
| `%E4Y`         | `DATE` `TIMESTAMP` | Four-character years (0001 ... 9999) that include leading zeros, unlike `%Y` that only uses as many characters as needed to render the year without leading zeros.                                                                                                                                                                         | `2021`                     |

Examples:

```
SELECT FORMAT_DATE("%b-%d-%Y", DATE "2008-12-25") AS formatted;

/*-------------+
 | formatted   |
 +-------------+
 | Dec-25-2008 |
 +-------------*/
```

```
SELECT FORMAT_TIMESTAMP("%b %Y %Ez", TIMESTAMP "2008-12-25 15:30:00+00")
  AS formatted;

/*-----------------+
 | formatted       |
 +-----------------+
 | Dec 2008 +00:00 |
 +-----------------*/
```

```
SELECT PARSE_DATE("%Y%m%d", "20081225") AS parsed;

/*------------+
 | parsed     |
 +------------+
 | 2008-12-25 |
 +------------*/
```

```
SELECT PARSE_TIMESTAMP("%c", "Thu Dec 25 07:30:00 2008") AS parsed;

-- Display of results may differ, depending upon the environment and
-- time zone where this query was executed.
/*------------------------+
 | parsed                 |
 +------------------------+
 | 2008-12-25T15:30:00Z   |
 +------------------------*/
```
