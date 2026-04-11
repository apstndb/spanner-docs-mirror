Spanner supports the following MySQL date and time functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](https://docs.cloud.google.com/spanner/docs/install-mysql-functions) .

## Function list

| Name                                                                                                                    | Summary                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| [`mysql.DATE_FORMAT`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#date_format)       | Formats a date as specified.                                                                       |
| [`mysql.DAY`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#day)                       | Alias for the `DAYOFMONTH` function. Returns the day of the month (1-31) from a `TIMESTAMP` value. |
| [`mysql.DAYNAME`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#dayname)               | Returns the name of the weekday.                                                                   |
| [`mysql.DAYOFMONTH`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#dayofmonth)         | Returns the day of the month (1-31).                                                               |
| [`mysql.DAYOFWEEK`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#dayofweek)           | Returns the weekday index (1-7) of the input parameter.                                            |
| [`mysql.DAYOFYEAR`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#dayofyear)           | Returns the day of the year (1-366).                                                               |
| [`mysql.FROM_DAYS`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#from_days)           | Converts a day number to a date.                                                                   |
| [`mysql.FROM_UNIXTIME`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#from_unixtime)   | Formats Unix timestamp as a date.                                                                  |
| [`mysql.HOUR`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#hour)                     | Returns the hour.                                                                                  |
| [`mysql.MAKEDATE`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#makedate)             | Creates a `DATE` value from a specified year and day of the year.                                  |
| [`mysql.MICROSECOND`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#microsecond)       | Returns the microseconds from the input parameter.                                                 |
| [`mysql.MINUTE`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#minute)                 | Returns the minute from the input parameter.                                                       |
| [`mysql.MONTH`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#month)                   | Returns the month from the date passed.                                                            |
| [`mysql.MONTHNAME`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#monthname)           | Returns the name of the month.                                                                     |
| [`mysql.PERIOD_ADD`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#period_add)         | Adds a specified number of months to a period of time.                                             |
| [`mysql.PERIOD_DIFF`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#period_diff)       | Returns the number of months between two periods.                                                  |
| [`mysql.QUARTER`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#quarter)               | Returns the quarter from a date input parameter.                                                   |
| [`mysql.SECOND`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#second)                 | Returns the second (0-59).                                                                         |
| [`mysql.STR_TO_DATE`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#str_to_date)       | Converts a string to a date.                                                                       |
| [`mysql.SYSDATE`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#sysdate)               | Returns the `TIMESTAMP` at which the query statement that contains this function started to run.   |
| [`mysql.TIME`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#time)                     | Extracts the time portion of the expression passed.                                                |
| [`mysql.TO_DAYS`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#to_days)               | Returns the date input parameter converted to days.                                                |
| [`mysql.TO_SECONDS`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#to_seconds)         | Returns the date or datetime input parameter converted to seconds since year zero.                 |
| [`mysql.UNIX_TIMESTAMP`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#unix_timestamp) | Returns a Unix timestamp.                                                                          |
| [`mysql.UTC_DATE`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#utc_date)             | Returns the current UTC date.                                                                      |
| [`mysql.WEEK`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#week)                     | Returns the week number (1-53).                                                                    |
| [`mysql.WEEKDAY`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#weekday)               | Returns the weekday index (0-6).                                                                   |
| [`mysql.WEEKOFYEAR`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#weekofyear)         | Returns the calendar week of the date (1-53).                                                      |
| [`mysql.YEAR`](https://docs.cloud.google.com/spanner/docs/reference/mysql/date_time_functions#year)                     | Returns the year.                                                                                  |

## `mysql.DATE_FORMAT`

    mysql.DATE_FORMAT(timestamp_expression, format_string)

**Description**

Formats a `TIMESTAMP` value according to a specified format string.

This function supports the following arguments:

  - `timestamp_expression` : The `TIMESTAMP` value to format.
  - `format_string` : A `STRING` value that contains [format elements](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/format-elements#format_elements_date_time) to use with `timestamp_expression` .

**Return data type**

`STRING`

**Differences from MySQL**

This function accepts only `TIMESTAMP` values, while the MySQL version also accepts `DATE` values depending on the format string. This implementation also supports a smaller subset of the format specifiers available in conventional MySQL.

**Limitations**

  - The following format specifiers are not supported: `%c, %D, %f, %h, %i, %M, %r, %s, %u, %V, %W, %X, %x` .
  - When you apply time-related format specifiers to a `DATE` object, this function ignores them. In contrast, MySQL substitutes values from a default time.

**Example**

The following example formats a `TIMESTAMP` value:

    SELECT mysql.DATE_FORMAT(TIMESTAMP '2023-10-27', '%Y-%d-%m') as formatted_date;
    
    /*
    +----------------+
    | formatted_date |
    +----------------+
    | 2023-27-10     |
    +----------------+
    */

## `mysql.DAY`

    mysql.DAY(timestamp_expression)

**Description**

Returns the day of the month for a `TIMESTAMP` value, from 1 to 31. This is an alias for `DAYOFMONTH` .

This function supports the following argument:

  - `timestamp_expression` : The `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

  - If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .
  - This function doesn't support the "zero date" ( `0000-00-00` ). Providing this value causes an error, while MySQL returns `NULL` .

**Example**

The following example gets the day of the month from a `TIMESTAMP` value:

    SELECT mysql.DAY(TIMESTAMP '2025-05-30') AS day_of_month;
    
    /*
    +--------------+
    | day_of_month |
    +--------------+
    | 30           |
    +--------------+
    */

## `mysql.DAYNAME`

    mysql.DAYNAME(timestamp_expression)

**Description**

Returns the full name of the weekday in English for a given `TIMESTAMP` value.

This function supports the following argument:

  - `timestamp_expression` : The `TIMESTAMP` value from which to extract the weekday name.

**Return data type**

`STRING`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values. In MySQL, the output language is controlled by the `lc_time_names` system variable; in GoogleSQL, the output is always in English.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `NULL` .

**Example**

The following example returns the name of the weekday from a `TIMESTAMP` value:

    SELECT mysql.DAYNAME(TIMESTAMP '2025-05-30') as day_name;
    
    /*
    +----------+
    | day_name |
    +----------+
    | Friday   |
    +----------+
    */

## `mysql.DAYOFMONTH`

    mysql.DAYOFMONTH(timestamp_expression)

**Description**

Returns the day of the month for a `TIMESTAMP` value, from 1 to 31.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the day of the month from a `TIMESTAMP` value:

    SELECT mysql.DAYOFMONTH(TIMESTAMP '2025-05-30') as dayofmonth;
    
    /*
    +------------------+
    | dayofmonth       |
    +------------------+
    | 30               |
    +------------------+
    */

## `mysql.DAYOFWEEK`

    mysql.DAYOFWEEK(timestamp_expression)

**Description**

Returns the weekday index for a `TIMESTAMP` value. The index uses Sunday as the first day of the week (Sunday = 1, Saturday = 7).

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `NULL` .

**Example**

The following example returns the weekday index for a given timestamp:

    SELECT mysql.DAYOFWEEK(TIMESTAMP '2025-05-30') AS day_of_week;
    
    /*
    +-------------+
    | day_of_week |
    +-------------+
    | 6           |
    +-------------+
    */

## `mysql.DAYOFYEAR`

    mysql.DAYOFYEAR(timestamp_expression)

**Description**

Returns the day of the year for a `TIMESTAMP` value, from 1 to 366.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `NULL` .

**Example**

The following example returns the day of the year from a `TIMESTAMP` value:

    SELECT mysql.DAYOFYEAR(TIMESTAMP '2025-05-30') AS day_of_year;
    
    /*
    +-------------+
    | day_of_year |
    +-------------+
    | 150         |
    +-------------+
    */

## `mysql.FROM_DAYS`

    mysql.FROM_DAYS(day_number)

**Description**

Converts an `INT64` day number into a `DATE` value.

This function supports the following argument:

  - `day_number` : The number of days.

**Return data type**

`DATE`

**Differences from MySQL**

  - This function does not support dates before `0001-01-01` .

  - Dates that precede the Gregorian calendar (1582), might vary from the MySQL version.

**Example**

The following example converts a day number to a `DATE` value:

    SELECT mysql.FROM_DAYS(739765) AS date_from_days;
    
    /*
    +----------------+
    | date_from_days |
    +----------------+
    | 2025-05-29     |
    +----------------+
    */

## `mysql.FROM_UNIXTIME`

    mysql.FROM_UNIXTIME(unix_timestamp)

**Description**

Converts a Unix timestamp (seconds since the epoch) into a `TIMESTAMP` value.

This function supports the following argument:

  - `unix_timestamp` : The number of seconds since the Unix epoch (1970-01-01 00:00:00 UTC).

**Return data type**

`TIMESTAMP`

**Differences from MySQL**

This function supports a wider range of timestamps than the MySQL version, including negative timestamps. The output is always in UTC, while the MySQL version output depends on the session time zone.

**Limitations**

This function only supports the single-argument version of `FROM_UNIXTIME` .

**Example**

The following example converts a Unix timestamp to a `TIMESTAMP` value:

    SELECT mysql.FROM_UNIXTIME(1748601000) AS timestamp_from_unix;
    
    /*
    +------------------------+
    | timestamp_from_unix    |
    +------------------------+
    | 2025-05-30 10:30:00+00 |
    +------------------------+
    */

## `mysql.HOUR`

    mysql.HOUR(timestamp_expression)

**Description**

Returns the hour from a `TIMESTAMP` value, from 0 to 23.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example gets the hour from a `TIMESTAMP` value:

    SELECT mysql.HOUR(TIMESTAMP '2025-05-30 14:30:45.123456') as hour;
    
    /*
    +------+
    | hour |
    +------+
    | 14   |
    +------+
    */

## `mysql.MAKEDATE`

    mysql.MAKEDATE(year, day_of_year)

**Description**

Creates a `DATE` value from a specified year and day of the year. The day of the year value is from 1 to 366.

This function supports the following argument:

  - `year` : The year ( `INT64` ).
  - `day_of_year` : The day of the year ( `INT64` ).

**Return data type**

`DATE`

**Example**

The following example creates a `DATE` value from the input parameters provided:

    SELECT mysql.MAKEDATE(2025, 150) AS date_from_year_day;
    
    /*
    +--------------------+
    | date_from_year_day |
    +--------------------+
    | 2025-05-30         |
    +--------------------+
    */

## `mysql.MICROSECOND`

    mysql.MICROSECOND(timestamp_expression)

**Description**

Returns the microsecond component from a `TIMESTAMP` value, from 0 to 999999.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the microsecond from a `TIMESTAMP` value:

    SELECT mysql.MICROSECOND(TIMESTAMP '2025-05-30 14:30:45.123456') as microsecond;
    
    /*
    +-------------+
    | microsecond |
    +-------------+
    | 123456      |
    +-------------+
    */

## `mysql.MINUTE`

    mysql.MINUTE(timestamp_expression)

**Description**

Returns the minute from a `TIMESTAMP` value, from 0 to 59.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the minute from a `TIMESTAMP` value:

    SELECT mysql.MINUTE(TIMESTAMP '2025-05-30 14:30:45.123456') as minute;
    
    /*
    +--------+
    | minute |
    +--------+
    | 30     |
    +--------+
    */

## `mysql.MONTH`

    mysql.MONTH(timestamp_expression)

**Description**

Returns the month from a `TIMESTAMP` value, from 1 to 12.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the month from a `TIMESTAMP` value:

    SELECT mysql.MONTH(TIMESTAMP '2025-05-30') as month_num;
    
    /*
    +-----------+
    | month_num |
    +-----------+
    | 5         |
    +-----------+
    */

## `mysql.MONTHNAME`

    mysql.MONTHNAME(timestamp_expression)

**Description**

Returns the full name of the month in English for a `TIMESTAMP` value.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`STRING`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values. In MySQL, the output language is controlled by the `lc_time_names` system variable; in GoogleSQL, the output is always in English.

**Limitations**

This function has no direct limitations. However, if you provide the timestamp as a string literal that is not a valid timestamp, this function returns an error. The MySQL version returns `NULL` .

**Example**

The following example returns the month name from a `TIMESTAMP` value:

    SELECT mysql.MONTHNAME(TIMESTAMP '2025-05-30') AS month_name;
    
    /*
    +------------+
    | month_name |
    +------------+
    | May        |
    +------------+
    */

## `mysql.PERIOD_ADD`

    mysql.PERIOD_ADD(period, months_to_add)

**Description**

Adds a specified number of months to a period (formatted as `YYYYMM` or `YYMM` ).

This function supports the following arguments:

  - `period` : The period, formatted as an integer (for example, `202505` ).
  - `months_to_add` : The number of months to add.

**Return data type**

`INT64`

**Example**

The following example adds 3 months to the period `202505` :

    SELECT mysql.PERIOD_ADD(202505, 3) AS period_plus_3_months;
    
    /*
    +----------------------+
    | period_plus_3_months |
    +----------------------+
    | 202508               |
    +----------------------+
    */

## `mysql.PERIOD_DIFF`

    mysql.PERIOD_DIFF(period1, period2)

**Description**

Returns the number of months between two periods (formatted as `YYYYMM` or `YYMM` ).

This function supports the following arguments:

  - `period1` : The first period, formatted as an integer.
  - `period2` : The second period, formatted as an integer.

**Return data type**

`INT64`

**Example**

The following example returns the difference in months between two periods:

    SELECT mysql.PERIOD_DIFF(202508, 202505) as months_diff;
    
    /*
    +-------------+
    | months_diff |
    +-------------+
    | 3           |
    +-------------+
    */

## `mysql.QUARTER`

    mysql.QUARTER(timestamp_expression)

**Description**

Returns the quarter of the year for a `TIMESTAMP` value, from 1 to 4.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the quarter of the year from a `TIMESTAMP` value:

    SELECT mysql.QUARTER(TIMESTAMP '2025-05-30') as quarter_of_year;
    
    /*
    +-----------------+
    | quarter_of_year |
    +-----------------+
    | 2               |
    +-----------------+
    */

## `mysql.SECOND`

    mysql.SECOND(timestamp_expression)

**Description**

Returns the second from a `TIMESTAMP` value, from 0 to 59.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the second from a `TIMESTAMP` value:

    SELECT mysql.SECOND(TIMESTAMP '2025-05-30 14:30:45.123456') as second;
    
    /*
    +--------+
    | second |
    +--------+
    | 45     |
    +--------+
    */

## `mysql.STR_TO_DATE`

    mysql.STR_TO_DATE(string_expression, format_string)

**Description**

Converts a string into a `TIMESTAMP` value based on a specified format string.

This function supports the following argument:

  - `string_expression` : The date string.
  - `format_string` : A `STRING` value that contains [format elements](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/format-elements#format_elements_date_time) to use with `timestamp_expression` .

**Return data type**

`TIMESTAMP`

**Differences from MySQL**

This function supports a wider range of timestamps than the MySQL version.

**Limitations**

  - The following format specifiers are not supported: `%c, %D, %f, %h, %i, %M, %r,` `%s, %u, %V, %W, %X, %x` .
  - This function always returns a `TIMESTAMP` , even if the format string does not contain time-related specifiers.

**Example**

The following example converts a string to a `TIMESTAMP` value:

    SELECT mysql.STR_TO_DATE('May 30, 2025', '%M %e, %Y') as date_from_string;
    
    /*
    +------------------------+
    | date_from_string       |
    +------------------------+
    | 2025-05-30 00:00:00+00 |
    +------------------------+
    */

## `mysql.SYSDATE`

    mysql.SYSDATE()

**Description**

Returns the `TIMESTAMP` at which the current query began to run.

This function doesn't support any arguments.

**Return data type**

`TIMESTAMP`

**Differences from MySQL**

This function is not an exact match for MySQL's `SYSDATE()` . This function returns the start time of the entire query statement, so multiple calls within the same query return the same value. In contrast, MySQL's `SYSDATE()` returns the time at which the function itself runs.

**Example**

The following example returns the current query's start timestamp:

    SELECT mysql.SYSDATE() AS start_time;
    
    /*
    +------------------------+
    | start_time             |
    +------------------------+
    | 2025-06-03 12:12:33+00 |
    +------------------------+
    */

## `mysql.TIME`

    mysql.TIME(timestamp_expression)

**Description**

Extracts the time portion from a `TIMESTAMP` value and returns it as a string.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`STRING`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values.

**Example**

The following example extracts the time from a `TIMESTAMP` value:

    SELECT mysql.TIME(TIMESTAMP '2025-05-30 14:30:45.123') AS time_part;
    
    /*
    +-----------------+
    | time_part       |
    +-----------------+
    | 14:30:45.123000 |
    +-----------------+
    */

## `mysql.TO_DAYS`

    mysql.TO_DAYS(date_expression)

**Description**

Converts a `DATE` value to the number of days since year zero. Year zero starts at 0000-00-00.

This function supports the following argument:

  - `date_expression` : The input `DATE` value.

**Return data type**

`INT64`

**Differences from MySQL**

The epoch (day zero) is different from what MySQL uses.

**Limitations**

Use this function with caution for dates that precede 1970-01-01, as behavior may vary from MySQL.

**Example**

The following example converts a `DATE` value to a number of days:

    SELECT mysql.TO_DAYS(DATE '2025-05-30') as days_since_year_0;
    
    /*
    +-------------------+
    | days_since_year_0 |
    +-------------------+
    | 739765            |
    +-------------------+
    */

## `mysql.TO_SECONDS`

    mysql.TO_SECONDS(timestamp_expression)

**Description**

Converts a `TIMESTAMP` value to the number of seconds since `0000-01-01 00:00:00` .

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` .

**Return data type**

`INT64`

**Limitations**

Use this function with caution on dates before the year 1901.

**Example**

The following example converts a `TIMESTAMP` to a number of seconds:

    SELECT mysql.TO_SECONDS(TIMESTAMP '2025-05-30 00:00:00') AS seconds_since_day_0;
    
    /*
    +----------------------+
    | seconds_since_day_0  |
    +----------------------+
    | 63915807600          |
    +----------------------+
    */

## `mysql.UNIX_TIMESTAMP`

    mysql.UNIX_TIMESTAMP(timestamp_expression)

**Description**

Returns the number of seconds from the Unix epoch (1970-01-01 00:00:00 UTC) to a specified `TIMESTAMP` value.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

Supports a wider range of `TIMESTAMP` values than the MySQL version.

**Limitations**

The zero-argument version of `UNIX_TIMESTAMP()` is not supported.

**Example**

The following example returns a Unix timestamp:

    SELECT mysql.UNIX_TIMESTAMP(TIMESTAMP '2025-05-30 14:30:00') AS unix_ts;
    
    /*
    +------------+
    | unix_ts    |
    +------------+
    | 1748640600 |
    +------------+
    */

## `mysql.UTC_DATE`

    mysql.UTC_DATE()

**Description**

Returns the current Coordinated Universal Time (UTC) date formatted as YYYY-MM-DD.

This function doesn't support any arguments.

**Return data type**

`DATE`

**Example**

The following example returns the current UTC date:

    SELECT mysql.UTC_DATE() AS current_utc_date;
    
    /*
    +------------------+
    | current_utc_date |
    +------------------+
    | 2025-06-03       |
    +------------------+
    */

## `mysql.WEEK`

    mysql.WEEK(timestamp_expression)

**Description**

Returns the week number for a `TIMESTAMP` value, from 1 to 53.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values, while the MySQL version also accepts `DATE` and `DATETIME` values. The MySQL version also has a `mode` argument to control the week's start day and range. This function does not support the `mode` argument and corresponds to MySQL's default mode (mode 0).

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the week number from a `TIMESTAMP` value:

    SELECT mysql.WEEK(TIMESTAMP '2025-05-30') as week_num;
    
    /*
    +----------+
    | week_num |
    +----------+
    | 21       |
    +----------+
    */

## `mysql.WEEKDAY`

    mysql.WEEKDAY(timestamp_expression)

**Description**

Returns the weekday index for a `TIMESTAMP` value. The index uses Monday as the first day of the week (Monday = 0, Sunday = 6).

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values. The underlying day-of-week logic differs, but this function adjusts the result to match MySQL's `WEEKDAY()` output.

**Example**

The following example returns the weekday index for a given timestamp:

    SELECT mysql.WEEKDAY(TIMESTAMP '2025-05-30') as weekday_index;
    
    /*
    +---------------+
    | weekday_index |
    +---------------+
    | 4             |
    +---------------+
    */

## `mysql.WEEKOFYEAR`

    mysql.WEEKOFYEAR(timestamp_expression)

**Description**

Returns the calendar week of the year for a `TIMESTAMP` value, from 1 to 53. This function uses the ISO 8601 standard for week numbering.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the week of the year from a `TIMESTAMP` value:

    SELECT mysql.WEEKOFYEAR(TIMESTAMP '2025-05-30') as weekofyear_iso;
    
    /*
    +----------------+
    | weekofyear_iso |
    +----------------+
    | 22             |
    +----------------+
    */

## `mysql.YEAR`

    mysql.YEAR(timestamp_expression)

**Description**

Returns the year from a `TIMESTAMP` value.

This function supports the following argument:

  - `timestamp_expression` : The input `TIMESTAMP` value.

**Return data type**

`INT64`

**Differences from MySQL**

This function only accepts `TIMESTAMP` values. The MySQL version also accepts `DATE` and `DATETIME` values.

**Limitations**

If you provide an invalid timestamp, this function returns an error. In contrast, MySQL returns `NULL` .

**Example**

The following example returns the year from a `TIMESTAMP` value:

    SELECT mysql.YEAR(TIMESTAMP '2025-05-30') as year_value;
    
    /*
    +------------+
    | year_value |
    +------------+
    | 2025       |
    +------------+
    */
