---
name: documents/docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions
uri: https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions
title: Conversion functions in GoogleSQL
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

GoogleSQL for Spanner supports conversion functions. These data type conversions are explicit, but some conversions can happen implicitly. You can learn more about implicit and explicit conversion [here](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_rules) .

## Function list

| Name                                                                                                                                              | Summary                                                                                                                                                                                                                                                                     |
|---------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`ARRAY_TO_STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_to_string)                            | Produces a concatenation of the elements in an array as a `STRING` value. For more information, see [Array functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions) .                                                                  |
| [`BOOL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#bool_for_json)                                          | Converts a JSON boolean to a SQL `BOOL` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                               |
| [`BOOL_ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#bool_array_for_json)                              | Converts a JSON array of booleans to a SQL `ARRAY<BOOL>` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                              |
| [`CAST`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast)                                             | Convert the results of an expression to the given type.                                                                                                                                                                                                                     |
| [`CODE_POINTS_TO_BYTES`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#code_points_to_bytes)                 | Converts an array of extended ASCII code points to a `BYTES` value. For more information, see [String aggregate functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                            |
| [`CODE_POINTS_TO_STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#code_points_to_string)               | Converts an array of extended ASCII code points to a `STRING` value. For more information, see [String aggregate functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                           |
| [`DATE_FROM_UNIX_DATE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions#date_from_unix_date)                     | Interprets an `INT64` expression as the number of days since 1970-01-01. For more information, see [Date functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions) .                                                                     |
| [`FROM_BASE32`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#from_base32)                                   | Converts a base32-encoded `STRING` value into a `BYTES` value. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                                           |
| [`FROM_BASE64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#from_base64)                                   | Converts a base64-encoded `STRING` value into a `BYTES` value. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                                           |
| [`FROM_HEX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#from_hex)                                         | Converts a hexadecimal-encoded `STRING` value into a `BYTES` value. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                                      |
| [`INT64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#int64_for_json)                                        | Converts a JSON number to a SQL `INT64` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                               |
| [`INT64_ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#int64_array_for_json)                            | Converts a JSON array of numbers to a SQL `ARRAY<INT64>` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                              |
| [`LAX_BOOL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#lax_bool)                                           | Attempts to convert a JSON value to a SQL `BOOL` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                      |
| [`LAX_FLOAT64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#lax_double)                                      | Attempts to convert a JSON value to a SQL `FLOAT64` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                   |
| [`LAX_INT64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#lax_int64)                                         | Attempts to convert a JSON value to a SQL `INT64` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                     |
| [`LAX_STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#lax_string)                                       | Attempts to convert a JSON value to a SQL `STRING` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                    |
| [`PARSE_DATE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions#parse_date)                                       | Converts a `STRING` value to a `DATE` value. For more information, see [Date functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions) .                                                                                                 |
| [`PARSE_JSON`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#parse_json)                                       | Converts a JSON-formatted `STRING` value to a `JSON` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                  |
| [`PARSE_TIMESTAMP`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#parse_timestamp)                        | Converts a `STRING` value to a `TIMESTAMP` value. For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                                                                  |
| [`SAFE_CAST`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#safe_casting)                                | Similar to the `CAST` function, but returns `NULL` when a runtime error is produced.                                                                                                                                                                                        |
| [`SAFE_CONVERT_BYTES_TO_STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#safe_convert_bytes_to_string) | Converts a `BYTES` value to a `STRING` value and replace any invalid UTF-8 characters with the Unicode replacement character, `U+FFFD` . For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) . |
| [`STRING` (JSON)](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#string_for_json)                               | Converts a JSON string to a SQL `STRING` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                              |
| [`STRING_ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#string_array_for_json)                          | Converts a JSON array of strings to a SQL `ARRAY<STRING>` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                             |
| [`STRING` (Timestamp)](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#string)                              | Converts a `TIMESTAMP` value to a `STRING` value. For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                                                                  |
| [`TIMESTAMP_MICROS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_micros)                      | Converts the number of microseconds since 1970-01-01 00:00:00 UTC to a `TIMESTAMP` . For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                               |
| [`TIMESTAMP_MILLIS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_millis)                      | Converts the number of milliseconds since 1970-01-01 00:00:00 UTC to a `TIMESTAMP` . For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                               |
| [`TIMESTAMP_SECONDS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_seconds)                    | Converts the number of seconds since 1970-01-01 00:00:00 UTC to a `TIMESTAMP` . For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                                    |
| [`TO_BASE32`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_base32)                                       | Converts a `BYTES` value to a base32-encoded `STRING` value. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                                             |
| [`TO_BASE64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_base64)                                       | Converts a `BYTES` value to a base64-encoded `STRING` value. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                                             |
| [`TO_CODE_POINTS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_code_points)                             | Converts a `STRING` or `BYTES` value into an array of extended ASCII code points. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                        |
| [`TO_HEX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_hex)                                             | Converts a `BYTES` value to a hexadecimal `STRING` value. For more information, see [String functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions) .                                                                                |
| [`TO_JSON`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#to_json)                                             | Converts a SQL value to a JSON value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                                                        |
| [`TO_JSON_STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#to_json_string)                               | Converts a `JSON` value to a SQL JSON-formatted `STRING` value. For more information, see [JSON functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) .                                                                              |
| [`UNIX_DATE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions#unix_date)                                         | Converts a `DATE` value to the number of days since 1970-01-01. For more information, see [Date functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions) .                                                                              |
| [`UNIX_MICROS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#unix_micros)                                | Converts a `TIMESTAMP` value to the number of microseconds since 1970-01-01 00:00:00 UTC. For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                          |
| [`UNIX_MILLIS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#unix_millis)                                | Converts a `TIMESTAMP` value to the number of milliseconds since 1970-01-01 00:00:00 UTC. For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                          |
| [`UNIX_SECONDS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#unix_seconds)                              | Converts a `TIMESTAMP` value to the number of seconds since 1970-01-01 00:00:00 UTC. For more information, see [Timestamp functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions) .                                               |

## `CAST`

```
CAST(expression AS typename)
```

**Description**

Cast syntax is used in a query to indicate that the result type of an expression should be converted to some other type.

When using `CAST` , a query can fail if GoogleSQL is unable to perform the cast. If you want to protect your queries from these types of errors, you can use [SAFE_CAST](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#safe_casting) .

Casts between supported types that don't successfully map from the original value to the target domain produce runtime errors. For example, casting `BYTES` to `STRING` where the byte sequence isn't valid UTF-8 results in a runtime error.

**Examples**

The following query results in `"true"` if `x` is `1` , `"false"` for any other non- `NULL` value, and `NULL` if `x` is `NULL` .

```
CAST(x=1 AS STRING)
```

### CAST AS ARRAY

```
CAST(expression AS ARRAY<element_type>)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `ARRAY` . The `expression` parameter can represent an expression for these data types:

- `ARRAY`

**Conversion rules**

| From    | To      | Rule(s) when casting `x`                                                                                                                          |
|---------|---------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `ARRAY` | `ARRAY` | Must be the exact same array type.                                                                                                                |
| `NULL`  | `ARRAY` | Casts a `NULL` value to a specific type of array. For example, `CAST(NULL AS ARRAY<FLOAT64>)` returns a `NULL` literal of type `ARRAY<FLOAT64>` . |

### CAST AS BOOL

```
CAST(expression AS BOOL)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `BOOL` . The `expression` parameter can represent an expression for these data types:

- `INT64`
- `BOOL`
- `STRING`

**Conversion rules**

| From     | To     | Rule(s) when casting `x`                                                                                                                                                                                             |
|----------|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| INT64    | `BOOL` | Returns `FALSE` if `x` is `0` , `TRUE` otherwise.                                                                                                                                                                    |
| `STRING` | `BOOL` | Returns `TRUE` if `x` is `"true"` and `FALSE` if `x` is `"false"` All other values of `x` are invalid and throw an error instead of casting to a boolean. A string is case-insensitive when converting to a boolean. |

### CAST AS BYTES

```
CAST(expression AS BYTES)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `BYTES` . The `expression` parameter can represent an expression for these data types:

- `BYTES`
- `STRING`
- `PROTO`

**Conversion rules**

| From     | To      | Rule(s) when casting `x`                                                                                                                                       |
|----------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STRING` | `BYTES` | Strings are cast to bytes using UTF-8 encoding. For example, the string "©", when cast to bytes, would become a 2-byte sequence with the hex values C2 and A9. |
| `PROTO`  | `BYTES` | Returns the proto2 wire format bytes of `x` .                                                                                                                  |

### CAST AS DATE

```
CAST(expression AS DATE)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `DATE` . The `expression` parameter can represent an expression for these data types:

- `STRING`
- `TIMESTAMP`

**Conversion rules**

| From        | To     | Rule(s) when casting `x`                                                                                                                                                                                                                                            |
|-------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STRING`    | `DATE` | When casting from string to date, the string must conform to the supported date literal format, and is independent of time zone. If the string expression is invalid or represents a date that's outside of the supported min/max range, then an error is produced. |
| `TIMESTAMP` | `DATE` | Casting from a timestamp to date effectively truncates the timestamp as of the default time zone.                                                                                                                                                                   |

### CAST AS ENUM

```
CAST(expression AS ENUM)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `ENUM` . The `expression` parameter can represent an expression for these data types:

- `INT64`
- `STRING`
- `ENUM`

**Conversion rules**

| From   | To     | Rule(s) when casting `x`      |
|--------|--------|-------------------------------|
| `ENUM` | `ENUM` | Must have the same enum name. |

### CAST AS Floating Point

```
CAST(expression AS FLOAT64)
```

```
CAST(expression AS FLOAT32)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to floating point types. The `expression` parameter can represent an expression for these data types:

- `INT64`
- `FLOAT32`
- `FLOAT64`
- `NUMERIC`
- `STRING`

**Conversion rules**

| From      | To             | Rule(s) when casting `x`                                                                                                                                                                                                                                                        |
|-----------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| INT64     | Floating Point | Returns a close but potentially not exact floating point value.                                                                                                                                                                                                                 |
| `NUMERIC` | Floating Point | `NUMERIC` will convert to the closest floating point number with a possible loss of precision.                                                                                                                                                                                  |
| `STRING`  | Floating Point | Returns `x` as a floating point value, interpreting it as having the same form as a valid floating point literal. Also supports casts from `"[+,-]inf"` to `[,-]Infinity` , `"[+,-]infinity"` to `[,-]Infinity` , and `"[+,-]nan"` to `NaN` . Conversions are case-insensitive. |

### CAST AS INT64

```
CAST(expression AS INT64)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to integer types. The `expression` parameter can represent an expression for these data types:

- `INT64`
- `FLOAT32`
- `FLOAT64`
- `NUMERIC`
- `ENUM`
- `BOOL`
- `STRING`

**Conversion rules**

| From           | To    | Rule(s) when casting `x`                                                                      |
|----------------|-------|-----------------------------------------------------------------------------------------------|
| Floating Point | INT64 | Returns the closest integer value. Halfway cases such as 1.5 or -0.5 round away from zero.    |
| `BOOL`         | INT64 | Returns `1` if `x` is `TRUE` , `0` otherwise.                                                 |
| `STRING`       | INT64 | A hex string can be cast to an integer. For example, `0x123` to `291` or `-0x123` to `-291` . |

**Examples**

If you are working with hex strings ( `0x123` ), you can cast those strings as integers:

```
SELECT '0x123' as hex_value, CAST('0x123' as INT64) as hex_to_int;

/*-----------+------------+
 | hex_value | hex_to_int |
 +-----------+------------+
 | 0x123     | 291        |
 +-----------+------------*/
```

```
SELECT '-0x123' as hex_value, CAST('-0x123' as INT64) as hex_to_int;

/*-----------+------------+
 | hex_value | hex_to_int |
 +-----------+------------+
 | -0x123    | -291       |
 +-----------+------------*/
```

### CAST AS JSON

```
CAST(expression AS JSON)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `JSON` . The `expression` parameter can represent an expression for these data types:

- `INT64`
- `FLOAT32`
- `FLOAT64`
- `NUMERIC`
- `BOOL`
- `STRING`
- `BYTES`
- `DATE`
- `TIMESTAMP`
- `INTERVAL`
- `ARRAY`
- `STRUCT`
- `PROTO`
- `ENUM`
- `UUID`
- `JSON`
- `GRAPH_ELEMENT`
- `GRAPH_PATH`

**Conversion rules**

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>From</th>
<th>To</th>
<th>Rule(s) when casting <code>x</code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Any supported type</td>
<td><code>JSON</code></td>
<td>Casting to <code>JSON</code> generally follows the same rules as the <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#to_json"><code>TO_JSON</code></a> function, with the following differences:
<ul>
<li>Casting a SQL <code>NULL</code> to <code>JSON</code> returns a SQL <code>NULL</code> , not a JSON <code>null</code> . This distinction doesn't extend to SQL <code>NULL</code> values nested with a <code>STRUCT</code> or an <code>ARRAY</code> , which still convert to JSON <code>null</code> .</li>
<li>Casting <code>+/-inf</code> or <code>NaN</code> floating point values to <code>JSON</code> returns an error.</li>
<li><code>CAST</code> doesn't support the <code>stringify_wide_numbers</code> optional parameter and always behaves as if <code>stringify_wide_numbers</code> is disabled.</li>
</ul></td>
</tr>
</tbody>
</table>

**Examples**

The following example casts various SQL types to `JSON` , representing the different JSON types (number, string, boolean, array, and object):

```
SELECT
  CAST(1 AS JSON) AS json_number,
  CAST('foo' AS JSON) AS json_string,
  CAST(TRUE AS JSON) AS json_boolean,
  CAST([1, 2, 3] AS JSON) AS json_array,
  CAST(STRUCT('foo' AS a, TRUE AS b) AS JSON) AS json_object;

/*-------------+-------------+--------------+------------+----------------------+
 | json_number | json_string | json_boolean | json_array | json_object          |
 +-------------+-------------+--------------+------------+----------------------+
 | 1           | "foo"       | true         | [1,2,3]    | {"a":"foo","b":true} |
 +-------------+-------------+--------------+------------+----------------------*/
```

The following example demonstrates how `NULL` values are handled:

- Casting a SQL `NULL` to `JSON` returns a SQL `NULL` instead of a JSON `null` .
- Casting a struct with a `NULL` field or an array with a `NULL` element to `JSON` converts the `NULL` to a JSON `null` .

```
SELECT
  CAST(NULL AS JSON) AS json_null,
  CAST(STRUCT(1 AS a, NULL AS b) AS JSON) AS json_object_with_null,
  CAST([1, NULL, 3] AS JSON) AS json_array_with_null;

/*-----------+-----------------------+----------------------+
 | json_null | json_object_with_null | json_array_with_null |
 +-----------+-----------------------+----------------------+
 | NULL      | {"a":1,"b":null}      | [1,null,3]           |
 +-----------+-----------------------+----------------------*/
```

### CAST AS INTERVAL

```
CAST(expression AS INTERVAL)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `INTERVAL` . The `expression` parameter can represent an expression for these data types:

- `STRING`

**Conversion rules**

| From     | To         | Rule(s) when casting `x`                                                                                                                                                                                                                                                                                                                                                                                                                           |
|----------|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STRING` | `INTERVAL` | When casting from string to interval, the string must conform to either [ISO 8601 Duration](https://en.wikipedia.org/wiki/ISO_8601#Durations) standard or to interval literal format 'Y-M D H:M:S.F'. Partial interval literal formats are also accepted when they aren't ambiguous, for example 'H:M:S'. If the string expression is invalid or represents an interval that is outside of the supported min/max range, then an error is produced. |

**Examples**

```
SELECT input, CAST(input AS INTERVAL) AS output
FROM UNNEST([
  '1-2 3 10:20:30.456',
  '1-2',
  '10:20:30',
  'P1Y2M3D',
  'PT10H20M30,456S'
]) input

/*--------------------+--------------------+
 | input              | output             |
 +--------------------+--------------------+
 | 1-2 3 10:20:30.456 | 1-2 3 10:20:30.456 |
 | 1-2                | 1-2 0 0:0:0        |
 | 10:20:30           | 0-0 0 10:20:30     |
 | P1Y2M3D            | 1-2 3 0:0:0        |
 | PT10H20M30,456S    | 0-0 0 10:20:30.456 |
 +--------------------+--------------------*/
```

### CAST AS NUMERIC

```
CAST(expression AS NUMERIC)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `NUMERIC` . The `expression` parameter can represent an expression for these data types:

- `INT64`
- `FLOAT32`
- `FLOAT64`
- `NUMERIC`
- `STRING`

**Conversion rules**

| From             | To        | Rule(s) when casting `x`                                                                                                                                                                                                                                                                                                                                                                     |
|------------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Floating Point` | `NUMERIC` | The floating point number will round [half away from zero](https://en.wikipedia.org/wiki/Rounding#Round_half_away_from_zero) . Casting a `NaN` , `+inf` or `-inf` will return an error. Casting a value outside the range of `NUMERIC` returns an overflow error.                                                                                                                            |
| `STRING`         | `NUMERIC` | The numeric literal contained in the string must not exceed the maximum precision or range of the `NUMERIC` type, or an error will occur. If the number of digits after the decimal point exceeds nine, then the resulting `NUMERIC` value will round [half away from zero](https://en.wikipedia.org/wiki/Rounding#Round_half_away_from_zero) . to have nine digits after the decimal point. |

### CAST AS PROTO

```
CAST(expression AS PROTO)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `PROTO` . The `expression` parameter can represent an expression for these data types:

- `STRING`
- `BYTES`
- `PROTO`

**Conversion rules**

| From     | To      | Rule(s) when casting `x`                                                                                                                                           |
|----------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STRING` | `PROTO` | Returns the protocol buffer that results from parsing from proto2 text format. Throws an error if parsing fails, e.g., if not all required fields are set.         |
| `BYTES`  | `PROTO` | Returns the protocol buffer that results from parsing `x` from the proto2 wire format. Throws an error if parsing fails, e.g., if not all required fields are set. |
| `PROTO`  | `PROTO` | Must have the same protocol buffer name.                                                                                                                           |

**Example**

This example references a protocol buffer called `Award` .

```
message Award {
  required int32 year = 1;
  optional int32 month = 2;
  repeated Type type = 3;

  message Type {
    optional string award_name = 1;
    optional string category = 2;
  }
}
```

```
SELECT
  CAST(
    '''
    year: 2001
    month: 9
    type { award_name: 'Best Artist' category: 'Artist' }
    type { award_name: 'Best Album' category: 'Album' }
    '''
    AS googlesql.examples.music.Award)
  AS award_col

/*---------------------------------------------------------+
 | award_col                                               |
 +---------------------------------------------------------+
 | {                                                       |
 |   year: 2001                                            |
 |   month: 9                                              |
 |   type { award_name: "Best Artist" category: "Artist" } |
 |   type { award_name: "Best Album" category: "Album" }   |
 | }                                                       |
 +---------------------------------------------------------*/
```

### CAST AS STRING

```
CAST(expression AS STRING)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `STRING` . The `expression` parameter can represent an expression for these data types:

- `INT64`
- `FLOAT32`
- `FLOAT64`
- `NUMERIC`
- `ENUM`
- `BOOL`
- `BYTES`
- `PROTO`
- `DATE`
- `TIMESTAMP`
- `INTERVAL`
- `STRING`

**Conversion rules**

| From           | To       | Rule(s) when casting `x`                                                                                                                                                                                                                                                                        |
|----------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Floating Point | `STRING` | Returns an approximate string representation. A returned `NaN` or `0` will not be signed.                                                                                                                                                                                                       |
| `BOOL`         | `STRING` | Returns `"true"` if `x` is `TRUE` , `"false"` otherwise.                                                                                                                                                                                                                                        |
| `BYTES`        | `STRING` | Returns `x` interpreted as a UTF-8 string. For example, the bytes literal `b'\xc2\xa9'` , when cast to a string, is interpreted as UTF-8 and becomes the unicode character "©". An error occurs if `x` isn't valid UTF-8.                                                                       |
| `ENUM`         | `STRING` | Returns the canonical enum value name of `x` . If an enum value has multiple names (aliases), the canonical name/alias for that value is used.                                                                                                                                                  |
| `PROTO`        | `STRING` | Returns the proto2 text format representation of `x` .                                                                                                                                                                                                                                          |
| `DATE`         | `STRING` | Casting from a date type to a string is independent of time zone and is of the form `YYYY-MM-DD` .                                                                                                                                                                                              |
| `TIMESTAMP`    | `STRING` | When casting from timestamp types to string, the timestamp is interpreted using the default time zone, America/Los_Angeles. The number of subsecond digits produced depends on the number of trailing zeroes in the subsecond part: the CAST function will truncate zero, three, or six digits. |
| `INTERVAL`     | `STRING` | Casting from an interval to a string is of the form `Y-M D H:M:S` .                                                                                                                                                                                                                             |

**Examples**

```
SELECT CAST(CURRENT_DATE() AS STRING) AS current_date

/*---------------+
 | current_date  |
 +---------------+
 | 2021-03-09    |
 +---------------*/
```

```
SELECT CAST(INTERVAL 3 DAY AS STRING) AS interval_to_string

/*--------------------+
 | interval_to_string |
 +--------------------+
 | 0-0 3 0:0:0        |
 +--------------------*/
```

```
SELECT CAST(
  INTERVAL "1-2 3 4:5:6.789" YEAR TO SECOND
  AS STRING) AS interval_to_string

/*--------------------+
 | interval_to_string |
 +--------------------+
 | 1-2 3 4:5:6.789    |
 +--------------------*/
```

### CAST AS STRUCT

```
CAST(expression AS STRUCT)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `STRUCT` . The `expression` parameter can represent an expression for these data types:

- `STRUCT`

**Conversion rules**

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>From</th>
<th>To</th>
<th>Rule(s) when casting <code>x</code></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>STRUCT</code></td>
<td><code>STRUCT</code></td>
<td>Allowed if the following conditions are met:<br />

<ol>
<li>The two structs have the same number of fields.</li>
<li>The original struct field types can be explicitly cast to the corresponding target struct field types (as defined by field order, not field name).</li>
</ol></td>
</tr>
</tbody>
</table>

### CAST AS TIMESTAMP

```
CAST(expression AS TIMESTAMP)
```

**Description**

GoogleSQL supports [casting](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#cast) to `TIMESTAMP` . The `expression` parameter can represent an expression for these data types:

- `DATE`
- `STRING`
- `TIMESTAMP`

**Conversion rules**

| From     | To          | Rule(s) when casting `x`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|----------|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STRING` | `TIMESTAMP` | When casting from string to a timestamp, `string_expression` must conform to the supported timestamp literal formats, or else a runtime error occurs. The `string_expression` may itself contain a time zone. If there is a time zone in the `string_expression` , that time zone is used for conversion, otherwise the default time zone, America/Los_Angeles, is used. If the string has fewer than six digits, then it's implicitly widened. An error is produced if the `string_expression` is invalid, has more than six subsecond digits (i.e., precision greater than microseconds), or represents a time outside of the supported timestamp range. |
| `DATE`   | `TIMESTAMP` | Casting from a date to a timestamp interprets `date_expression` as of midnight (start of the day) in the default time zone, America/Los_Angeles.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

**Examples**

The following example casts a string-formatted timestamp as a timestamp:

```
SELECT CAST("2020-06-02 17:00:53.110+00:00" AS TIMESTAMP) AS as_timestamp

-- Results depend upon where this query was executed.
/*-------------------------+
 | as_timestamp            |
 +-------------------------+
 | 2020-06-03T00:00:53.11Z |
 +-------------------------*/
```

## `SAFE_CAST`

```
SAFE_CAST(expression AS typename)
```

**Description**

When using `CAST` , a query can fail if GoogleSQL is unable to perform the cast. For example, the following query generates an error:

```
SELECT CAST("apple" AS INT64) AS not_a_number;
```

If you want to protect your queries from these types of errors, you can use `SAFE_CAST` . `SAFE_CAST` replaces runtime errors with `NULL` s. However, during static analysis, impossible casts between two non-castable types still produce an error because the query is invalid.

```
SELECT SAFE_CAST("apple" AS INT64) AS not_a_number;

/*--------------+
 | not_a_number |
 +--------------+
 | NULL         |
 +--------------*/
```

If you are casting from bytes to strings, you can also use the function, [`SAFE_CONVERT_BYTES_TO_STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#safe_convert_bytes_to_string) . Any invalid UTF-8 characters are replaced with the unicode replacement character, `U+FFFD` .
