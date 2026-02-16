This document defines the data types supported for PostgreSQL-dialect databases.

## Supported PostgreSQL data types

All types except `  NUMERIC  ` , `  FLOAT4  ` , and `  JSONB  ` are valid as primary keys, foreign keys, and secondary indexes. `  FLOAT8  ` columns used as a key column cannot store `  NaN  ` values.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Supported PostgreSQL data types</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       array      </code></td>
<td>Ordered list of zero or more elements of any non-array type. The PostgreSQL interface doesn't support multi-dimensional arrays. For user-defined indexes, the lower bound must be one and the upper bound must be the length of the array. For more information, see <a href="#array">Array type</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       bool      </code> / <code dir="ltr" translate="no">       boolean      </code></td>
<td>Logical boolean (true/false).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       bytea      </code></td>
<td>Binary data ("byte array").</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       date      </code></td>
<td>Dates ranging from 0001-01-01 to 9999-12-31. The <code dir="ltr" translate="no">       DATE      </code> type represents a logical calendar date, independent of time zone. A <code dir="ltr" translate="no">       DATE      </code> value doesn't represent a specific 24-hour time period. Rather, a given <code dir="ltr" translate="no">       DATE      </code> value represents a different 24-hour period when interpreted in different time zones, and might represent a shorter or longer day during daylight savings time transitions. To represent an absolute point in time, use the <code dir="ltr" translate="no">       timestamptz      </code> data type.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       float4      </code> / <code dir="ltr" translate="no">       real      </code></td>
<td>IEEE-754 single-precision binary floating-point format number (4 bytes) ( <a href="https://en.wikipedia.org/wiki/Single-precision_floating-point_format#IEEE_754_standard:_binary32">Wikipedia link</a> ). <code dir="ltr" translate="no">       float4      </code> type follows PostgreSQL semantics:
<ul>
<li>NaNs are greater than all non-null values.</li>
<li>NaNs are considered equal.</li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       float8      </code> / <code dir="ltr" translate="no">       double precision      </code></td>
<td>IEEE-754 double-precision binary floating-point format number (8 bytes) ( <a href="https://en.wikipedia.org/wiki/Double-precision_floating-point_format#IEEE_754_double-precision_binary_floating-point_format:_binary64">Wikipedia link</a> ). <code dir="ltr" translate="no">       float8      </code> type follows PostgreSQL semantics:
<ul>
<li>NaNs are greater than all non-null values.</li>
<li>NaNs are considered equal.</li>
</ul></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       int      </code></td>
<td>An alias for <code dir="ltr" translate="no">       int8      </code> . In open source PostgreSQL, <code dir="ltr" translate="no">       int      </code> is a four-byte integer, but in PostgreSQL interface for Spanner <code dir="ltr" translate="no">       int      </code> maps to <code dir="ltr" translate="no">       int8      </code> , a signed eight-byte integer.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       int8      </code> / <code dir="ltr" translate="no">       bigint      </code></td>
<td>Signed eight-byte (64-bit) integer.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       interval      </code></td>
<td>Data type used to represent duration or amount of time, without referring to any specific point in time.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb      </code></td>
<td>Data type used for holding JSON data. It maps to the <a href="https://www.postgresql.org/docs/current/datatype-json.html">PostgreSQL JSONB</a> data type. For more information, see <a href="/spanner/docs/working-with-jsonb">Work with JSONB data</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric      </code> / <code dir="ltr" translate="no">       decimal      </code></td>
<td>Arbitrary length numeric data. Optional precision and scale type modifiers (for example, <code dir="ltr" translate="no">       numeric(18,4)      </code> ) are supported in DQL/DML statements (for example, <code dir="ltr" translate="no">       SELECT      </code> or <code dir="ltr" translate="no">       INSERT      </code> ), but not in DDL statements (for example, <code dir="ltr" translate="no">       CREATE      </code> or <code dir="ltr" translate="no">       ALTER      </code> ). For more information, see <a href="/spanner/docs/working-with-numerics">Work with NUMERIC data</a> . <code dir="ltr" translate="no">       NUMERIC      </code> follows PostgreSQL semantics:
<ul>
<li>NaNs are greater than all non-null values.</li>
<li>NaNs are considered equal.</li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       timestamptz      </code> / <code dir="ltr" translate="no">       timestamp with time zone      </code></td>
<td>Date and time, including time zone. <code dir="ltr" translate="no">       timestamp with time zone      </code> can be expressed with a UTC offset. The following timestamp literal specifies America/Los_Angeles Pacific Standard Time <code dir="ltr" translate="no">       2016-06-22 19:10:25-08      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       varchar      </code> / <code dir="ltr" translate="no">       text      </code> / <code dir="ltr" translate="no">       character varying      </code></td>
<td>Variable-length character string. Optional type modifier (not applicable to the <code dir="ltr" translate="no">       text      </code> type) specifies a column size limit in characters (for example, <code dir="ltr" translate="no">       varchar(64)      </code> ). Maximum column size limit for these types is 2621440 characters.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       smallserial      </code> / <code dir="ltr" translate="no">       serial      </code> / <code dir="ltr" translate="no">       bigserial      </code> / <code dir="ltr" translate="no">       serial2      </code> / <code dir="ltr" translate="no">       serial4      </code> / <code dir="ltr" translate="no">       serial8      </code></td>
<td>Aliases that map to identity columns with the data type <code dir="ltr" translate="no">       bigint      </code> . The database option <code dir="ltr" translate="no">       default_sequence_kind      </code> must be set before using serial types.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       uuid      </code></td>
<td>Data type used for universally unique identifier (UUID) values. UUIDs are represented as a 128-bit number.</td>
</tr>
</tbody>
</table>

### Supported formats for `     date    ` data type

The following table shows the supported input formats for the `  date  ` data type. Note that the date interpretation is month-day-year similar to the case when the `  DateStyle  ` parameter is set to `  MDY  ` in open source PostgreSQL.

<table>
<thead>
<tr class="header">
<th>Example</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1999-01-08</td>
<td>ISO 8601; January 8 (recommended format)</td>
</tr>
<tr class="even">
<td>January 8, 1999</td>
<td>Unambiguous</td>
</tr>
<tr class="odd">
<td>1/8/1999</td>
<td>January 8</td>
</tr>
<tr class="even">
<td>1/18/1999</td>
<td>January 18</td>
</tr>
<tr class="odd">
<td>01/02/03</td>
<td>January 2, 2003</td>
</tr>
<tr class="even">
<td>1999-Jan-08</td>
<td>January 8</td>
</tr>
<tr class="odd">
<td>Jan-08-1999</td>
<td>January 8</td>
</tr>
<tr class="even">
<td>08-Jan-1999</td>
<td>January 8</td>
</tr>
<tr class="odd">
<td>99-Jan-08</td>
<td>Returns error</td>
</tr>
<tr class="even">
<td>08-Jan-99</td>
<td>January 8</td>
</tr>
<tr class="odd">
<td>Jan-08-99</td>
<td>January 8</td>
</tr>
<tr class="even">
<td>19990108</td>
<td>ISO 8601; January 8, 1999</td>
</tr>
<tr class="odd">
<td>990108</td>
<td>ISO 8601; January 8, 1999</td>
</tr>
<tr class="even">
<td>1999.008</td>
<td>Year and day of year</td>
</tr>
<tr class="odd">
<td>J2451187</td>
<td>Julian date</td>
</tr>
</tbody>
</table>

### Supported formats for `     timestamptz    ` data type

See the following tables for the supported input formats for the `  timestamptz  ` data type.

Time input:

<table>
<thead>
<tr class="header">
<th>Example</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>04:05:06.789</td>
<td>ISO 8601</td>
</tr>
<tr class="even">
<td>04:05:06</td>
<td>ISO 8601</td>
</tr>
<tr class="odd">
<td>04:05</td>
<td>ISO 8601</td>
</tr>
<tr class="even">
<td>040506</td>
<td>ISO 8601</td>
</tr>
<tr class="odd">
<td>04:05 AM</td>
<td>Same as 04:05; AM does not affect value</td>
</tr>
<tr class="even">
<td>04:05 PM</td>
<td>Same as 16:05; input hour must be &lt;= 12</td>
</tr>
<tr class="odd">
<td>04:05:06.789-8</td>
<td>ISO 8601, with time zone as UTC offset</td>
</tr>
<tr class="even">
<td>04:05:06-08:00</td>
<td>ISO 8601, with time zone as UTC offset</td>
</tr>
<tr class="odd">
<td>04:05-08:00</td>
<td>ISO 8601, with time zone as UTC offset</td>
</tr>
<tr class="even">
<td>040506-08</td>
<td>ISO 8601, with time zone as UTC offset</td>
</tr>
<tr class="odd">
<td>040506+0730</td>
<td>ISO 8601, with fractional-hour time zone as UTC offset</td>
</tr>
<tr class="even">
<td>040506+07:30:00</td>
<td>UTC offset specified to seconds (not allowed in ISO 8601)</td>
</tr>
<tr class="odd">
<td>2003-04-12 04:05:06 America/New_York</td>
<td>Time zone specified by full name</td>
</tr>
<tr class="even">
<td>2003-04-12 04:05:06-8</td>
<td>Time zone specified with -8, UTC offset for PST</td>
</tr>
</tbody>
</table>

Time zone input:

<table>
<thead>
<tr class="header">
<th>Example</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>America/New_York</td>
<td>Full time zone name</td>
</tr>
<tr class="even">
<td>-8:00:00</td>
<td>UTC offset for PST</td>
</tr>
<tr class="odd">
<td>-8:00</td>
<td>UTC offset for PST (ISO 8601 extended format)</td>
</tr>
<tr class="even">
<td>-800</td>
<td>UTC offset for PST (ISO 8601 basic format)</td>
</tr>
<tr class="odd">
<td>-8</td>
<td>UTC offset for PST (ISO 8601 basic format)</td>
</tr>
<tr class="even">
<td>zulu</td>
<td>Military abbreviation for UTC</td>
</tr>
<tr class="odd">
<td>z</td>
<td>Abbreviation for zulu</td>
</tr>
</tbody>
</table>

Only a limited set of timezone abbreviations is supported. The supported abbreviations are based on the IANA Time Zone Database, but not all are supported. Use full timezone names (for example, `  America/Los_Angeles  ` ). For a comprehensive list of valid timezone names, see the TZ identifier column in the [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) .

Supported abbreviations include:

  - `  GMT  ` , `  UTC  ` , `  Z  ` , `  CET  ` , `  EET  ` , `  WET  `

**Unsupported formats for `  date  ` and `  timestamptz  ` data types**

The following special literal values are unsupported: `  now  ` , `  yesterday  ` , `  today  ` , `  tomorrow  ` , `  epoch  ` , `  -infinity  ` , and `  infinity  ` .

For example, the following query returns an error:

`  SELECT 'today'::timestamptz;  `

Unsupported timezone abbreviations include:

  - `  BST  ` , `  IST  ` , `  PST  ` , `  CEST  ` , `  EEST  ` , `  MSK  ` , `  FET  ` , `  WEST  `

For a comprehensive list of valid timezone names, see the TZ identifier column in the [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) .

### Supported formats for `     interval    ` type

The following table shows the supported input formats for the `  interval  ` data type.

<table>
<thead>
<tr class="header">
<th>Example</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       P1Y2M3DT4H5M6.5S      </code></td>
<td>ISO 8601, format with designators (recommended format). Represents interval (months: 14, days: 3, seconds: 14706.5).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       P0001-02-03T04:05:06.5      </code></td>
<td>ISO 8601, alternative format. Represents interval (months: 14, days: 3, seconds: 14706.5).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       1 year 2 months 3 days 4 hours 5 minutes 6 seconds      </code></td>
<td>Open source PostgreSQL format. Represents interval (months: 14, days: 3, seconds: 14706).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       -1 year -2 months 3 days 04:05:06.5      </code></td>
<td>PostgreSQL format. Represents interval (months: -14, days: 3, seconds: 14706.5).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       @ 1 year 2 months -3 days 4 hours 5 mins 6 secs ago      </code></td>
<td>PostgreSQL verbose format. Represents interval (months: -14, days: 3, seconds: -14706).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       1-2      </code></td>
<td>SQL standard; year-month interval. Represents interval (months: 14, days: 0, seconds: 0).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       3 4:05:06      </code></td>
<td>SQL standard; day-time interval. Represents interval (months:0, days: 3, seconds: 14706)</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       1-2 3 4:05:06      </code></td>
<td>Mixed interval (both year-month and day-time). Represents interval (months:14, days: 3, seconds: 14706)</td>
</tr>
</tbody>
</table>

## Unsupported PostgreSQL data types

All other open source PostgreSQL data types are not supported. The following common types are not supported:

  - `  TIMESTAMP WITHOUT TIME ZONE  `
  - `  CHAR  `

## Array type

Arrays in the PostgreSQL interface use the behavior and syntax described in the [PostgreSQL Declaration of Array types](https://www.postgresql.org/docs/current/arrays.html) , except for the specified [Array type limitations](/spanner/docs/reference/postgresql/data-types#array-limitations) and [Spanner extension to open source PostgreSQL](#array-extensions) .

For the PostgreSQL interface, an array is an ordered list of zero or more elements of non-array values. Elements in an array must share the same type.

Arrays of arrays are not allowed. Queries that would produce an array of arrays return an error. An empty array and a NULL array are two distinct values. Arrays can contain NULL elements.

### Declare an array type

The following example shows how to create a table that declares an array:

``` text
CREATE TABLE students_info (
    name             text PRIMARY KEY,
    phone_numbers    varchar[]
);
```

Array declaration includes a name and square brackets ( `  []  ` ) with the chosen array data type. In the previous example, the name is "phone\_numbers", and `  varchar[]  ` denotes a varchar array for the phone contacts for students. The previous example also adds a `  text  ` type column for student names.

#### Examples

<table>
<colgroup>
<col style="width: 35%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Type declarations</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       bigint[]      </code></td>
<td>Simple array of 64-bit integers.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       float4[]              float8[]      </code></td>
<td>Array of floats, either 4 bytes or 8 bytes.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       integer[3]      </code></td>
<td>DDL syntax allows the exact size of arrays to be specified. Note, however, that declaring an array size does not enforce a size limit. Array size can be modified after declaration.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       integer ARRAY[4]      </code><br />
<code dir="ltr" translate="no">       integer ARRAY      </code></td>
<td>An alternative syntax which is more similar to the SQL standard by using the keyword ARRAY. As before, the size restriction is not enforced in any case.</td>
</tr>
</tbody>
</table>

#### Construct an array

You can construct an array using array literals or array functions. To learn how, see [Working with arrays in PostgreSQL-dialect databases](/spanner/docs/reference/postgresql/arrays) .

### Array limitations for PostgreSQL-dialect databases

This section lists limitations for the array data type for PostgreSQL-dialect databases, as opposed to open source PostgreSQL.

#### Multidimensional arrays

The PostgreSQL interface does not support multi-dimensional arrays. For example, you cannot create the following array:

``` text
CREATE TABLE rectangle_grid (
    id          integer PRIMARY KEY,
    rectangle   integer[4][3]
);
```

#### Array slices

The PostgreSQL interface supports using the array slice syntax, as shown in the following:

``` text
SELECT (array[10, 20, 30, 40])[2:3] → {20, 30}
```

#### Array indexes

The PostgreSQL interface does not support arrays with indexes that are different from the default values. For user-defined indexes, the lower bound must be `  1  ` and the upper bound must be the length of the array.

### Spanner extension to open source PostgreSQL

Spanner extends the array data type with the `  VECTOR LENGTH  ` parameter. This optional parameter sets an array to a fixed size for use in a vector search. The length must be a non-negative number and zero is allowed. You can only apply this parameter on an array that uses the `  float8  ` or `  double precision  ` data types. The following example shows how to use `  VECTOR LENGTH  ` in a DDL statement for `  CREATE TABLE  ` :

``` text
CREATE TABLE Singers (
  id int8 NOT NULL PRIMARY KEY,
  singer_vector float[] NOT NULL VECTOR LENGTH 4
)
```

## Serial types

Spanner maps serial types to identity columns with the data type `  bigint  ` . Serial types are aliases but not true types so you won't see them when you serialize your schema. The following example shows how to use `  serial  ` in a DDL statement for `  CREATE TABLE  ` :

``` text
ALTER DATABASE db SET spanner.default_sequence_kind = 'bit_reversed_positive';

CREATE TABLE Singers (
  id serial PRIMARY KEY,
  name text
);
```

The sample output of the `  GetDatabaseDDL  ` command for this schema looks like the following:

``` text
ALTER DATABASE db SET "spanner.default_sequence_kind" = 'bit_reversed_positive';

CREATE TABLE singers (
  id bigint GENERATED BY DEFAULT AS IDENTITY NOT NULL,
  name character varying,
  PRIMARY KEY(id)
);
```

## Interval type

Interval is a query-only type, and can't be stored in a table. A database schema can't have interval columns. However, the database schema can have column expressions and views with an interval type.

Unlike open source PostgreSQL, Spanner always uses ISO 8601 date and time format for interval output.

The interval type supports the following range:

  - Min: `  interval(months: -120000, days: -3660000, microseconds: -316224000000000000)  `
  - Max: `  interval(months: 120000, days: 3660000, microseconds: 316224000000000000)  `
