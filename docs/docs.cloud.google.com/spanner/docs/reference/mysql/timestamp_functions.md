Spanner supports the following MySQL timestamp functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

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
<td><a href="#datediff"><code dir="ltr" translate="no">        mysql.DATEDIFF       </code></a></td>
<td>Subtracts two dates, returns the number of days between them.</td>
</tr>
<tr class="even">
<td><a href="#localtime"><code dir="ltr" translate="no">        mysql.LOCALTIME       </code></a></td>
<td>Alias for <a href="#now"><code dir="ltr" translate="no">        mysql.NOW       </code></a> .</td>
</tr>
<tr class="odd">
<td><a href="#localtimestamp"><code dir="ltr" translate="no">        mysql.LOCALTIMESTAMP       </code></a></td>
<td>Alias for <a href="#now"><code dir="ltr" translate="no">        mysql.NOW       </code></a> .</td>
</tr>
<tr class="even">
<td><a href="#now"><code dir="ltr" translate="no">        mysql.NOW       </code></a></td>
<td>Returns the TIMESTAMP at which the query statement that contains this function started to run.</td>
</tr>
</tbody>
</table>

## `     mysql.DATEDIFF    `

``` text
mysql.DATEDIFF(timestamp_expression1, timestamp_expression2)
```

**Description**

Calculates the number of days between two `  TIMESTAMP  ` values ( `  timestamp_expression1  ` - `  timestamp_expression2  ` ).

This function supports the following arguments:

  - `  timestamp_expression1  ` : The first `  TIMESTAMP  ` value (minuend).
  - `  timestamp_expression2  ` : The second `  TIMESTAMP  ` value (subtrahend).

**Return data type**

`  INT64  `

**Differences from MySQL**

This function only accepts `  TIMESTAMP  ` values. The MySQL version also accepts `  DATE  ` or `  DATETIME  ` values.

**Example**

The following example calculates the difference in days between two timestamps:

``` text
SELECT mysql.DATEDIFF(TIMESTAMP '2025-01-10 10:00:00', TIMESTAMP '2025-01-01 05:00:00')
as days_difference;

/*
+-----------------+
| days_difference |
+-----------------+
| 9               |
+-----------------+
*/
```

## `     mysql.LOCALTIME    `

``` text
mysql.LOCALTIME()
```

**Description**

Returns the `  TIMESTAMP  ` when the current query statement started to run. This function is an alias for `  mysql.NOW()  ` and `  mysql.LOCALTIMESTAMP()  ` .

This function doesn't support any arguments.

**Return data type**

`  TIMESTAMP  `

**Example**

The following example returns the start time of the current query:

``` text
SELECT mysql.LOCALTIME() as current_query_time;

/*
+-------------------------------+
| current_query_time            |
+-------------------------------+
| 2025-06-03 12:28:32.123456+00 |
+-------------------------------+
*/
```

## `     mysql.LOCALTIMESTAMP    `

``` text
mysql.LOCALTIMESTAMP()
```

**Description**

Alias for [`  NOW  `](#now) .

## `     mysql.NOW    `

``` text
mysql.NOW()
```

**Description**

Returns the `  TIMESTAMP  ` at which the current query statement started to run. This function is an alias for `  mysql.LOCALTIME()  ` and `  mysql.LOCALTIMESTAMP()  ` .

This function doesn't support any arguments.

**Return data type**

`  TIMESTAMP  `

**Example**

The following example returns the start time of the current query:

``` text
SELECT mysql.NOW() as current_query_time;

/*
+-------------------------------+
| current_query_time            |
+-------------------------------+
| 2025-06-03 12:28:32.123456+00 |
+-------------------------------+
*/
```

## `     mysql.UTC_TIMESTAMP    `

``` text
mysql.UTC_TIMESTAMP()
```

**Description**

Returns the current Coordinated Universal Time (UTC) `  TIMESTAMP  ` at which the query statement started to run. In this implementation, it behaves like `  mysql.NOW()  ` .

This function doesn't support any arguments.

**Return data type**

`  TIMESTAMP  `

**Differences from MySQL**

While MySQL's `  UTC_TIMESTAMP()  ` always returns a UTC timestamp regardless of the session timezone, this function, as implemented, returns the query start time, which is inherently UTC in GoogleSQL.

**Example**

The following example returns the current UTC timestamp at the start of the query:

``` text
SELECT mysql.UTC_TIMESTAMP() as current_utc_ts;

/*
+-------------------------------+
| current_utc_ts                |
+-------------------------------+
| 2025-06-03 12:28:32.123456+00 |
+-------------------------------+
*/
```
