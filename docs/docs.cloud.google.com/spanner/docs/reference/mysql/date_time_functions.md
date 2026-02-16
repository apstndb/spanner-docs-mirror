Spanner supports the following MySQL date and time functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

## Function list

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="#date_format"><code dir="ltr" translate="no">        mysql.DATE_FORMAT       </code></a></td>
<td>Formats a date as specified.</td>
</tr>
<tr class="even">
<td><a href="#day"><code dir="ltr" translate="no">        mysql.DAY       </code></a></td>
<td>Alias for the <code dir="ltr" translate="no">       DAYOFMONTH      </code> function. Returns the day of the month (1-31) from a <code dir="ltr" translate="no">       TIMESTAMP      </code> value.</td>
</tr>
<tr class="odd">
<td><a href="#dayname"><code dir="ltr" translate="no">        mysql.DAYNAME       </code></a></td>
<td>Returns the name of the weekday.</td>
</tr>
<tr class="even">
<td><a href="#dayofmonth"><code dir="ltr" translate="no">        mysql.DAYOFMONTH       </code></a></td>
<td>Returns the day of the month (1-31).</td>
</tr>
<tr class="odd">
<td><a href="#dayofweek"><code dir="ltr" translate="no">        mysql.DAYOFWEEK       </code></a></td>
<td>Returns the weekday index (1-7) of the input parameter.</td>
</tr>
<tr class="even">
<td><a href="#dayofyear"><code dir="ltr" translate="no">        mysql.DAYOFYEAR       </code></a></td>
<td>Returns the day of the year (1-366).</td>
</tr>
<tr class="odd">
<td><a href="#from_days"><code dir="ltr" translate="no">        mysql.FROM_DAYS       </code></a></td>
<td>Converts a day number to a date.</td>
</tr>
<tr class="even">
<td><a href="#from_unixtime"><code dir="ltr" translate="no">        mysql.FROM_UNIXTIME       </code></a></td>
<td>Formats Unix timestamp as a date.</td>
</tr>
<tr class="odd">
<td><a href="#hour"><code dir="ltr" translate="no">        mysql.HOUR       </code></a></td>
<td>Returns the hour.</td>
</tr>
<tr class="even">
<td><a href="#makedate"><code dir="ltr" translate="no">        mysql.MAKEDATE       </code></a></td>
<td>Creates a <code dir="ltr" translate="no">       DATE      </code> value from a specified year and day of the year.</td>
</tr>
<tr class="odd">
<td><a href="#microsecond"><code dir="ltr" translate="no">        mysql.MICROSECOND       </code></a></td>
<td>Returns the microseconds from the input parameter.</td>
</tr>
<tr class="even">
<td><a href="#minute"><code dir="ltr" translate="no">        mysql.MINUTE       </code></a></td>
<td>Returns the minute from the input parameter.</td>
</tr>
<tr class="odd">
<td><a href="#month"><code dir="ltr" translate="no">        mysql.MONTH       </code></a></td>
<td>Returns the month from the date passed.</td>
</tr>
<tr class="even">
<td><a href="#monthname"><code dir="ltr" translate="no">        mysql.MONTHNAME       </code></a></td>
<td>Returns the name of the month.</td>
</tr>
<tr class="odd">
<td><a href="#period_add"><code dir="ltr" translate="no">        mysql.PERIOD_ADD       </code></a></td>
<td>Adds a specified number of months to a period of time.</td>
</tr>
<tr class="even">
<td><a href="#period_diff"><code dir="ltr" translate="no">        mysql.PERIOD_DIFF       </code></a></td>
<td>Returns the number of months between two periods.</td>
</tr>
<tr class="odd">
<td><a href="#quarter"><code dir="ltr" translate="no">        mysql.QUARTER       </code></a></td>
<td>Returns the quarter from a date input parameter.</td>
</tr>
<tr class="even">
<td><a href="#second"><code dir="ltr" translate="no">        mysql.SECOND       </code></a></td>
<td>Returns the second (0-59).</td>
</tr>
<tr class="odd">
<td><a href="#str_to_date"><code dir="ltr" translate="no">        mysql.STR_TO_DATE       </code></a></td>
<td>Converts a string to a date.</td>
</tr>
<tr class="even">
<td><a href="#sysdate"><code dir="ltr" translate="no">        mysql.SYSDATE       </code></a></td>
<td>Returns the <code dir="ltr" translate="no">       TIMESTAMP      </code> at which the query statement that contains this function started to run.</td>
</tr>
<tr class="odd">
<td><a href="#time"><code dir="ltr" translate="no">        mysql.TIME       </code></a></td>
<td>Extracts the time portion of the expression passed.</td>
</tr>
<tr class="even">
<td><a href="#to_days"><code dir="ltr" translate="no">        mysql.TO_DAYS       </code></a></td>
<td>Returns the date input parameter converted to days.</td>
</tr>
<tr class="odd">
<td><a href="#to_seconds"><code dir="ltr" translate="no">        mysql.TO_SECONDS       </code></a></td>
<td>Returns the date or datetime input parameter converted to seconds since year zero.</td>
</tr>
<tr class="even">
<td><a href="#unix_timestamp"><code dir="ltr" translate="no">        mysql.UNIX_TIMESTAMP       </code></a></td>
<td>Returns a Unix timestamp.</td>
</tr>
<tr class="odd">
<td><a href="#utc_date"><code dir="ltr" translate="no">        mysql.UTC_DATE       </code></a></td>
<td>Returns the current UTC date.</td>
</tr>
<tr class="even">
<td><a href="#week"><code dir="ltr" translate="no">        mysql.WEEK       </code></a></td>
<td>Returns the week number (1-53).</td>
</tr>
<tr class="odd">
<td><a href="#weekday"><code dir="ltr" translate="no">        mysql.WEEKDAY       </code></a></td>
<td>Returns the weekday index (0-6).</td>
</tr>
<tr class="even">
<td><a href="#weekofyear"><code dir="ltr" translate="no">        mysql.WEEKOFYEAR       </code></a></td>
<td>Returns the calendar week of the date (1-53).</td>
</tr>
<tr class="odd">
<td><a href="#year"><code dir="ltr" translate="no">        mysql.YEAR       </code></a></td>
<td>Returns the year.</td>
</tr>
</tbody>
</table>

## `     mysql.DATE_FORMAT    `

``` text
mysql.DATE_FORMAT(timestamp_expression, format_string)
```

**Description**

Formats a `  TIMESTAMP  ` value according to a specified format string.

This function supports the following arguments:

  - `  timestamp_expression  ` : The `  TIMESTAMP  ` value to format.
  - `  format_string  ` : A `  STRING  ` value that contains [format elements](/spanner/docs/reference/standard-sql/format-elements#format_elements_date_time) to use with `  timestamp_expression  ` .

**Return data type**

`  STRING  `

**Differences from MySQL**

This function accepts only `  TIMESTAMP  ` values, while the MySQL version also accepts `  DATE  ` values depending on the format string. This implementation also supports a smaller subset of the format specifiers available in conventional MySQL.

**Limitations**

  - The following format specifiers are not supported: `  %c, %D, %f, %h, %i, %M, %r, %s, %u, %V, %W, %X, %x  ` .
  - When you apply time-related format specifiers to a `  DATE  ` object, this function ignores them. In contrast, MySQL substitutes values from a default time.

**Example**

The following example formats a `  TIMESTAMP  ` value:

``` text
SELECT mysql.DATE_FORMAT(TIMESTAMP '2023-10-27', '%Y-%d-%m') as formatted_date;

/*
+----------------+
| formatted_date |
+----------------+
| 2023-27-10     |
+----------------+
*/
```

## `     mysql.DAY    `

``` text
mysql.DAY(timestamp_expression)
```

**Description**

Returns the day of the month for a `  TIMESTAMP  ` value, from 1 to 31. This is an alias for `  DAYOFMONTH  ` .

This function supports the following argument:

  - `  timestamp_expression  ` : The `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

  - If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .
  - This function doesn't support the "zero date" ( `  0000-00-00  ` ). Providing this value causes an error, while MySQL returns `  NULL  ` .

**Example**

The following example gets the day of the month from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.DAY(TIMESTAMP '2025-05-30') AS day_of_month;

/*
+--------------+
| day_of_month |
+--------------+
| 30           |
+--------------+
*/
```

## `     mysql.DAYNAME    `

``` text
mysql.DAYNAME(timestamp_expression)
```

**Description**

Returns the full name of the weekday in English for a given `  TIMESTAMP  ` value.

This function supports the following argument:

  - `  timestamp_expression  ` : The `  TIMESTAMP  ` value from which to extract the weekday name.

**Return data type**

`  STRING  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values. In MySQL, the output language is controlled by the `  lc_time_names  ` system variable; in GoogleSQL, the output is always in English.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `  NULL  ` .

**Example**

The following example returns the name of the weekday from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.DAYNAME(TIMESTAMP '2025-05-30') as day_name;

/*
+----------+
| day_name |
+----------+
| Friday   |
+----------+
*/
```

## `     mysql.DAYOFMONTH    `

``` text
mysql.DAYOFMONTH(timestamp_expression)
```

**Description**

Returns the day of the month for a `  TIMESTAMP  ` value, from 1 to 31.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the day of the month from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.DAYOFMONTH(TIMESTAMP '2025-05-30') as dayofmonth;

/*
+------------------+
| dayofmonth       |
+------------------+
| 30               |
+------------------+
*/
```

## `     mysql.DAYOFWEEK    `

``` text
mysql.DAYOFWEEK(timestamp_expression)
```

**Description**

Returns the weekday index for a `  TIMESTAMP  ` value. The index uses Sunday as the first day of the week (Sunday = 1, Saturday = 7).

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `  NULL  ` .

**Example**

The following example returns the weekday index for a given timestamp:

``` text
SELECT mysql.DAYOFWEEK(TIMESTAMP '2025-05-30') AS day_of_week;

/*
+-------------+
| day_of_week |
+-------------+
| 6           |
+-------------+
*/
```

## `     mysql.DAYOFYEAR    `

``` text
mysql.DAYOFYEAR(timestamp_expression)
```

**Description**

Returns the day of the year for a `  TIMESTAMP  ` value, from 1 to 366.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `  NULL  ` .

**Example**

The following example returns the day of the year from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.DAYOFYEAR(TIMESTAMP '2025-05-30') AS day_of_year;

/*
+-------------+
| day_of_year |
+-------------+
| 150         |
+-------------+
*/
```

## `     mysql.FROM_DAYS    `

``` text
mysql.FROM_DAYS(day_number)
```

**Description**

Converts an `  INT64  ` day number into a `  DATE  ` value.

This function supports the following argument:

  - `  day_number  ` : The number of days.

**Return data type**

`  DATE  `

**Differences from MySQL**

  - This function does not support dates before `  0001-01-01  ` .

  - Dates that precede the Gregorian calendar (1582), might vary from the MySQL version.

**Example**

The following example converts a day number to a `  DATE  ` value:

``` text
SELECT mysql.FROM_DAYS(739765) AS date_from_days;

/*
+----------------+
| date_from_days |
+----------------+
| 2025-05-29     |
+----------------+
*/
```

## `     mysql.FROM_UNIXTIME    `

``` text
mysql.FROM_UNIXTIME(unix_timestamp)
```

**Description**

Converts a Unix timestamp (seconds since the epoch) into a `  TIMESTAMP  ` value.

This function supports the following argument:

  - `  unix_timestamp  ` : The number of seconds since the Unix epoch (1970-01-01 00:00:00 UTC).

**Return data type**

`  TIMESTAMP  `

**Differences from MySQL**

This function supports a wider range of timestamps than the MySQL version, including negative timestamps. The output is always in UTC, while the MySQL version output depends on the session time zone.

**Limitations**

This function only supports the single-argument version of `  FROM_UNIXTIME  ` .

**Example**

The following example converts a Unix timestamp to a `  TIMESTAMP  ` value:

``` text
SELECT mysql.FROM_UNIXTIME(1748601000) AS timestamp_from_unix;

/*
+------------------------+
| timestamp_from_unix    |
+------------------------+
| 2025-05-30 10:30:00+00 |
+------------------------+
*/
```

## `     mysql.HOUR    `

``` text
mysql.HOUR(timestamp_expression)
```

**Description**

Returns the hour from a `  TIMESTAMP  ` value, from 0 to 23.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example gets the hour from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.HOUR(TIMESTAMP '2025-05-30 14:30:45.123456') as hour;

/*
+------+
| hour |
+------+
| 14   |
+------+
*/
```

## `     mysql.MAKEDATE    `

``` text
mysql.MAKEDATE(year, day_of_year)
```

**Description**

Creates a `  DATE  ` value from a specified year and day of the year. The day of the year value is from 1 to 366.

This function supports the following argument:

  - `  year  ` : The year ( `  INT64  ` ).
  - `  day_of_year  ` : The day of the year ( `  INT64  ` ).

**Return data type**

`  DATE  `

**Example**

The following example creates a `  DATE  ` value from the input parameters provided:

``` text
SELECT mysql.MAKEDATE(2025, 150) AS date_from_year_day;

/*
+--------------------+
| date_from_year_day |
+--------------------+
| 2025-05-30         |
+--------------------+
*/
```

## `     mysql.MICROSECOND    `

``` text
mysql.MICROSECOND(timestamp_expression)
```

**Description**

Returns the microsecond component from a `  TIMESTAMP  ` value, from 0 to 999999.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the microsecond from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.MICROSECOND(TIMESTAMP '2025-05-30 14:30:45.123456') as microsecond;

/*
+-------------+
| microsecond |
+-------------+
| 123456      |
+-------------+
*/
```

## `     mysql.MINUTE    `

``` text
mysql.MINUTE(timestamp_expression)
```

**Description**

Returns the minute from a `  TIMESTAMP  ` value, from 0 to 59.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the minute from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.MINUTE(TIMESTAMP '2025-05-30 14:30:45.123456') as minute;

/*
+--------+
| minute |
+--------+
| 30     |
+--------+
*/
```

## `     mysql.MONTH    `

``` text
mysql.MONTH(timestamp_expression)
```

**Description**

Returns the month from a `  TIMESTAMP  ` value, from 1 to 12.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the month from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.MONTH(TIMESTAMP '2025-05-30') as month_num;

/*
+-----------+
| month_num |
+-----------+
| 5         |
+-----------+
*/
```

## `     mysql.MONTHNAME    `

``` text
mysql.MONTHNAME(timestamp_expression)
```

**Description**

Returns the full name of the month in English for a `  TIMESTAMP  ` value.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  STRING  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values. In MySQL, the output language is controlled by the `  lc_time_names  ` system variable; in GoogleSQL, the output is always in English.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `  NULL  ` .

**Example**

The following example returns the month name from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.MONTHNAME(TIMESTAMP '2025-05-30') AS month_name;

/*
+------------+
| month_name |
+------------+
| May        |
+------------+
*/
```

## `     mysql.PERIOD_ADD    `

``` text
mysql.PERIOD_ADD(period, months_to_add)
```

**Description**

Adds a specified number of months to a period (formatted as `  YYYYMM  ` or `  YYMM  ` ).

This function supports the following arguments:

  - `  period  ` : The period, formatted as an integer (for example, `  202505  ` ).
  - `  months_to_add  ` : The number of months to add.

**Return data type**

`  INT64  `

**Example**

The following example adds 3 months to the period `  202505  ` :

``` text
SELECT mysql.PERIOD_ADD(202505, 3) AS period_plus_3_months;

/*
+----------------------+
| period_plus_3_months |
+----------------------+
| 202508               |
+----------------------+
*/
```

## `     mysql.PERIOD_DIFF    `

``` text
mysql.PERIOD_DIFF(period1, period2)
```

**Description**

Returns the number of months between two periods (formatted as `  YYYYMM  ` or `  YYMM  ` ).

This function supports the following arguments:

  - `  period1  ` : The first period, formatted as an integer.
  - `  period2  ` : The second period, formatted as an integer.

**Return data type**

`  INT64  `

**Example**

The following example returns the difference in months between two periods:

``` text
SELECT mysql.PERIOD_DIFF(202508, 202505) as months_diff;

/*
+-------------+
| months_diff |
+-------------+
| 3           |
+-------------+
*/
```

## `     mysql.QUARTER    `

``` text
mysql.QUARTER(timestamp_expression)
```

**Description**

Returns the quarter of the year for a `  TIMESTAMP  ` value, from 1 to 4.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the quarter of the year from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.QUARTER(TIMESTAMP '2025-05-30') as quarter_of_year;

/*
+-----------------+
| quarter_of_year |
+-----------------+
| 2               |
+-----------------+
*/
```

## `     mysql.SECOND    `

``` text
mysql.SECOND(timestamp_expression)
```

**Description**

Returns the second from a `  TIMESTAMP  ` value, from 0 to 59.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the second from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.SECOND(TIMESTAMP '2025-05-30 14:30:45.123456') as second;

/*
+--------+
| second |
+--------+
| 45     |
+--------+
*/
```

## `     mysql.STR_TO_DATE    `

``` text
mysql.STR_TO_DATE(string_expression, format_string)
```

**Description**

Converts a string into a `  TIMESTAMP  ` value based on a specified format string.

This function supports the following argument:

  - `  string_expression  ` : The date string.
  - `  format_string  ` : A `  STRING  ` value that contains [format elements](/spanner/docs/reference/standard-sql/format-elements#format_elements_date_time) to use with `  timestamp_expression  ` .

**Return data type**

`  TIMESTAMP  `

**Differences from MySQL**

This function supports a wider range of timestamps than the MySQL version.

**Limitations**

  - The following format specifiers are not supported: `  %c, %D, %f, %h, %i, %M, %r,  ` `  %s, %u, %V, %W, %X, %x  ` .
  - This function always returns a `  TIMESTAMP  ` , even if the format string does not contain time-related specifiers.

**Example**

The following example converts a string to a `  TIMESTAMP  ` value:

``` text
SELECT mysql.STR_TO_DATE('May 30, 2025', '%M %e, %Y') as date_from_string;

/*
+------------------------+
| date_from_string       |
+------------------------+
| 2025-05-30 00:00:00+00 |
+------------------------+
*/
```

## `     mysql.SYSDATE    `

``` text
mysql.SYSDATE()
```

**Description**

Returns the `  TIMESTAMP  ` at which the current query began to run.

This function doesn't support any arguments.

**Return data type**

`  TIMESTAMP  `

**Differences from MySQL**

This function is not an exact match for MySQL's `  SYSDATE()  ` . This function returns the start time of the entire query statement, so multiple calls within the same query return the same value. In contrast, MySQL's `  SYSDATE()  ` returns the time at which the function itself runs.

**Example**

The following example returns the current query's start timestamp:

``` text
SELECT mysql.SYSDATE() AS start_time;

/*
+------------------------+
| start_time             |
+------------------------+
| 2025-06-03 12:12:33+00 |
+------------------------+
*/
```

## `     mysql.TIME    `

``` text
mysql.TIME(timestamp_expression)
```

**Description**

Extracts the time portion from a `  TIMESTAMP  ` value and returns it as a string.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  STRING  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values.

**Example**

The following example extracts the time from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.TIME(TIMESTAMP '2025-05-30 14:30:45.123') AS time_part;

/*
+-----------------+
| time_part       |
+-----------------+
| 14:30:45.123000 |
+-----------------+
*/
```

## `     mysql.TO_DAYS    `

``` text
mysql.TO_DAYS(date_expression)
```

**Description**

Converts a `  DATE  ` value to the number of days since year zero. Year zero starts at 0000-00-00.

This function supports the following argument:

  - `  date_expression  ` : The input `  DATE  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

The epoch (day zero) is different from what MySQL uses.

**Limitations**

Use this function with caution for dates that precede 1970-01-01, as behavior may vary from MySQL.

**Example**

The following example converts a `  DATE  ` value to a number of days:

``` text
SELECT mysql.TO_DAYS(DATE '2025-05-30') as days_since_year_0;

/*
+-------------------+
| days_since_year_0 |
+-------------------+
| 739765            |
+-------------------+
*/
```

## `     mysql.TO_SECONDS    `

``` text
mysql.TO_SECONDS(timestamp_expression)
```

**Description**

Converts a `  TIMESTAMP  ` value to the number of seconds since `  0000-01-01 00:00:00  ` .

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` .

**Return data type**

`  INT64  `

**Limitations**

Use this function with caution on dates before the year 1901.

**Example**

The following example converts a `  TIMESTAMP  ` to a number of seconds:

``` text
SELECT mysql.TO_SECONDS(TIMESTAMP '2025-05-30 00:00:00') AS seconds_since_day_0;

/*
+----------------------+
| seconds_since_day_0  |
+----------------------+
| 63915807600          |
+----------------------+
*/
```

## `     mysql.UNIX_TIMESTAMP    `

``` text
mysql.UNIX_TIMESTAMP(timestamp_expression)
```

**Description**

Returns the number of seconds from the Unix epoch (1970-01-01 00:00:00 UTC) to a specified `  TIMESTAMP  ` value.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

Supports a wider range of `  TIMESTAMP  ` values than the MySQL version.

**Limitations**

The zero-argument version of `  UNIX_TIMESTAMP()  ` is not supported.

**Example**

The following example returns a Unix timestamp:

``` text
SELECT mysql.UNIX_TIMESTAMP(TIMESTAMP '2025-05-30 14:30:00') AS unix_ts;

/*
+------------+
| unix_ts    |
+------------+
| 1748640600 |
+------------+
*/
```

## `     mysql.UTC_DATE    `

``` text
mysql.UTC_DATE()
```

**Description**

Returns the current Coordinated Universal Time (UTC) date formatted as YYYY-MM-DD.

This function doesn't support any arguments.

**Return data type**

`  DATE  `

**Example**

The following example returns the current UTC date:

``` text
SELECT mysql.UTC_DATE() AS current_utc_date;

/*
+------------------+
| current_utc_date |
+------------------+
| 2025-06-03       |
+------------------+
*/
```

## `     mysql.WEEK    `

``` text
mysql.WEEK(timestamp_expression)
```

**Description**

Returns the week number for a `  TIMESTAMP  ` value, from 1 to 53.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values, while the MySQL version also accepts `  DATE  ` and `  DATETIME  ` values. The MySQL version also has a `  mode  ` argument to control the week's start day and range. This function does not support the `  mode  ` argument and corresponds to MySQL's default mode (mode 0).

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the week number from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.WEEK(TIMESTAMP '2025-05-30') as week_num;

/*
+----------+
| week_num |
+----------+
| 21       |
+----------+
*/
```

## `     mysql.WEEKDAY    `

``` text
mysql.WEEKDAY(timestamp_expression)
```

**Description**

Returns the weekday index for a `  TIMESTAMP  ` value. The index uses Monday as the first day of the week (Monday = 0, Sunday = 6).

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values. The underlying day-of-week logic differs, but this function adjusts the result to match MySQL's `  WEEKDAY()  ` output.

**Example**

The following example returns the weekday index for a given timestamp:

``` text
SELECT mysql.WEEKDAY(TIMESTAMP '2025-05-30') as weekday_index;

/*
+---------------+
| weekday_index |
+---------------+
| 4             |
+---------------+
*/
```

## `     mysql.WEEKOFYEAR    `

``` text
mysql.WEEKOFYEAR(timestamp_expression)
```

**Description**

Returns the calendar week of the year for a `  TIMESTAMP  ` value, from 1 to 53. This function uses the ISO 8601 standard for week numbering.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the week of the year from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.WEEKOFYEAR(TIMESTAMP '2025-05-30') as weekofyear_iso;

/*
+----------------+
| weekofyear_iso |
+----------------+
| 22             |
+----------------+
*/
```

## `     mysql.YEAR    `

``` text
mysql.YEAR(timestamp_expression)
```

**Description**

Returns the year from a `  TIMESTAMP  ` value.

This function supports the following argument:

  - `  timestamp_expression  ` : The input `  TIMESTAMP  ` value.

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` and `  DATETIME  ` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `  NULL  ` .

**Example**

The following example returns the year from a `  TIMESTAMP  ` value:

``` text
SELECT mysql.YEAR(TIMESTAMP '2025-05-30') as year_value;

/*
+------------+
| year_value |
+------------+
| 2025       |
+------------+
*/
```
