Spanner supports the following MySQL string functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

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
<td><a href="#bit_length"><code dir="ltr" translate="no">        mysql.BIT_LENGTH       </code></a></td>
<td>Returns the length of a string in bits.</td>
</tr>
<tr class="even">
<td><a href="#char"><code dir="ltr" translate="no">        mysql.CHAR       </code></a></td>
<td>Interprets the input parameter as an integer and returns a byte string consisting of the character given by the code value of that integer.</td>
</tr>
<tr class="odd">
<td><a href="#concat_ws"><code dir="ltr" translate="no">        mysql.CONCAT_WS       </code></a></td>
<td>Concatenates two string with a specified separator string.</td>
</tr>
<tr class="even">
<td><a href="#hex"><code dir="ltr" translate="no">        mysql.HEX       </code></a></td>
<td>Returns the hexadecimal representation of a string.</td>
</tr>
<tr class="odd">
<td><a href="#insert"><code dir="ltr" translate="no">        mysql.INSERT       </code></a></td>
<td>Inserts a substring into a string at a specified position, replacing a specified number of characters.</td>
</tr>
<tr class="even">
<td><a href="#locate"><code dir="ltr" translate="no">        mysql.LOCATE       </code></a></td>
<td>Returns the position of the first occurrence of substring.</td>
</tr>
<tr class="odd">
<td><a href="#mid"><code dir="ltr" translate="no">        mysql.MID       </code></a></td>
<td>Alias for <a href="/spanner/docs/reference/standard-sql/string_functions#substring"><code dir="ltr" translate="no">        SUBSTRING       </code></a> .</td>
</tr>
<tr class="even">
<td><a href="#oct"><code dir="ltr" translate="no">        mysql.OCT       </code></a></td>
<td>Returns a string containing an octal representation of the number.</td>
</tr>
<tr class="odd">
<td><a href="#ord"><code dir="ltr" translate="no">        mysql.ORD       </code></a></td>
<td>Returns the ASCII code of the leftmost character in a string.</td>
</tr>
<tr class="even">
<td><a href="#position"><code dir="ltr" translate="no">        mysql.POSITION       </code></a></td>
<td>Alias for <a href="#locate"><code dir="ltr" translate="no">        mysql.LOCATE       </code></a> .</td>
</tr>
<tr class="odd">
<td><a href="#quote"><code dir="ltr" translate="no">        mysql.QUOTE       </code></a></td>
<td>Sanitize a string for use in a SQL statement.</td>
</tr>
<tr class="even">
<td><a href="#regexp_like"><code dir="ltr" translate="no">        mysql.REGEXP_LIKE       </code></a></td>
<td>Returns whether the string matches a regular expression.</td>
</tr>
<tr class="odd">
<td><a href="#regexp_substr"><code dir="ltr" translate="no">        mysql.REGEXP_SUBSTR       </code></a></td>
<td>Returns the first substring that matches a regular expression pattern.</td>
</tr>
<tr class="even">
<td><a href="#space"><code dir="ltr" translate="no">        mysql.SPACE       </code></a></td>
<td>Returns a string of the specified number of spaces.</td>
</tr>
<tr class="odd">
<td><a href="#strcmp"><code dir="ltr" translate="no">        mysql.STRCMP       </code></a></td>
<td>Compares two strings for equality.</td>
</tr>
<tr class="even">
<td><a href="#substring_index"><code dir="ltr" translate="no">        mysql.SUBSTRING_INDEX       </code></a></td>
<td>Returns a substring from before or after a specified number of delimiter occurrences.</td>
</tr>
<tr class="odd">
<td><a href="#unhex"><code dir="ltr" translate="no">        mysql.UNHEX       </code></a></td>
<td>Converts a string of hexadecimal characters into its byte equivalent.</td>
</tr>
</tbody>
</table>

## `     mysql.BIT_LENGTH    `

``` text
mysql.BIT_LENGTH(string_expression)
```

**Description**

Returns the length of a given string in bits.

This function supports the following argument:

  - `  string_expression  ` : The input `  STRING  ` value.

**Return data type**

`  INT64  `

**Example**

The following example returns the bit length of the string 'google':

``` text
SELECT mysql.BIT_LENGTH('google') as bit_len;

/*
+---------+
| bit_len |
+---------+
| 48      |
+---------+
*/
```

## `     mysql.CHAR    `

``` text
mysql.CHAR(numeric_expression)
```

**Description**

Interprets an integer argument as a code value and returns a `  BYTES  ` string consisting of the character for that code value. Arguments larger than 255 are converted into multiple result bytes. This function emulates MySQL's multi-argument behavior for values by taking the input modulo 4294967296 and then interpreting the resulting integer's bytes. For example, `  CHAR(256)  ` is equivalent to `  CHAR(1,0)  ` .

This function supports the following argument:

  - `  numeric_expression  ` : The `  INT64  ` value to convert to a byte character.

**Return data type**

`  BYTES  `

**Differences from MySQL**

This function does not support the `  USING  ` clause that MySQL's `  CHAR()  ` function offers for specifying character sets. It also only accepts a single integer argument, whereas MySQL's `  CHAR()  ` can accept multiple integer arguments to produce a multi-character string.

**Limitations**

This function only handles a single integer argument and does not support the `  USING  ` clause.

**Example**

The following example returns the byte string for the character code 65:

``` text
SELECT mysql.CHAR(65) AS char_from_code;

/*
+----------------+
| char_from_code |
+----------------+
| A              |
+----------------+
*/
```

## `     mysql.CONCAT_WS    `

``` text
mysql.CONCAT_WS(separator, value1, value2)
```

**Description**

Concatenates two strings with a specified separator string.

This function supports the following argument:

  - `  separator  ` : The `  STRING  ` to use as a separator.
  - `  value1  ` : The first `  STRING  ` .
  - `  value2  ` : The second `  STRING  ` .

**Return data type**

`  STRING  `

**Differences from MySQL**

Similar to `  CONCAT  ` , MySQL converts arguments to strings if they are not already, but this implementation expects `  STRING  ` arguments.

**Limitations**

This function only supports concatenating two strings with a separator. MySQL's `  CONCAT_WS  ` function can take a variable number of string arguments.

**Example**

The following example concatenates two strings using a hyphen as a separator:

``` text
SELECT mysql.CONCAT_WS('-', 'google', 'cloud') as concatenated_string;

/*
+---------------------+
| concatenated_string |
+---------------------+
| google-cloud        |
+---------------------+
*/
```

## `     mysql.HEX    `

``` text
mysql.HEX(string_expression)
```

**Description**

Returns the hexadecimal representation of a string.

This function supports the following arguments:

  - `  string_expression  ` : The input `  STRING  ` .

**Return data type**

`  STRING  `

**Differences from MySQL**

Only the `  STRING  ` input version is provided. MySQL's `  HEX()  ` function can also accept numeric arguments.

**Limitations**

This function only handles `  STRING  ` input and does not support numeric input.

**Example**

The following example returns the hexadecimal representation of the string "SQL":

``` text
SELECT mysql.HEX('SQL') AS hex_string;

/*
+------------+
| hex_string |
+------------+
| 53514C     |
+------------+
*/
```

## `     mysql.INSERT    `

``` text
mysql.INSERT(original_value, position, length, new_value)
```

**Description**

Inserts a substring into a string at a specified position, replacing a specified number of characters.

This function supports the following arguments:

  - `  original_value  ` : The original `  STRING  ` .
  - `  position  ` : The starting position for insertion (1-based). If `  pos  ` is outside the length of `  str  ` , the original string is returned.
  - `  length  ` : The number of characters in the original string to replace.
  - `  new_value  ` : The `  STRING  ` to insert.

**Return data type**

`  STRING  `

**Limitations**

`  INSERT  ` is a reserved keyword. If you use this function in Data Definition Language (DDL) statements, such as in generated column definitions, you must enclose the function name in backticks (for example, ``  mysql.`INSERT`  `` ).

**Example**

The following example inserts "Google" into "Hello World" at position 7, replacing 0 characters:

``` text
SELECT mysql.`INSERT`('Hello World', 7, 0, 'Google ') as inserted_string;

/*
+------------------------+
| inserted_string        |
+------------------------+
| Hello Google World     |
+------------------------+
*/
```

## `     mysql.LOCATE    `

``` text
mysql.LOCATE(substring, string)
```

**Description**

Returns the starting position (1-based) of the first occurrence of a substring within a string. The search is case-insensitive. Returns 0 if the substring is not found.

This function supports the following arguments:

  - `  substring  ` : The `  STRING  ` to search for.
  - `  string  ` : The `  STRING  ` to be searched.

**Return data type**

`  INT64  `

**Differences from MySQL**

MySQL's `  LOCATE()  ` function also has a three-argument version that allows specifying a starting position for the search. This function only supports the two-argument version.

**Limitations**

This function does not support the three-argument version of MySQL's `  LOCATE()  ` (which includes a starting position).

**Example**

The following example finds the position of "Cloud" in "Google Cloud":

``` text
SELECT mysql.LOCATE('Cloud', 'Google Cloud') as position_val;

/*
+--------------+
| position_val |
+--------------+
| 8            |
+--------------+
*/
```

## `     mysql.MID    `

``` text
mysql.MID(value, position, length)
```

**Description**

Alias for [`  SUBSTRING  `](/spanner/docs/reference/standard-sql/string_functions#substring) . For more information, see the [`  SUBSTRING  `](/spanner/docs/reference/standard-sql/string_functions#substring) entry.

## `     mysql.OCT    `

``` text
mysql.OCT(numeric_expression)
```

**Description**

Returns a string containing the octal (base-8) representation of a number.

This function supports the following argument:

  - `  numeric_expression  ` : The input `  INT64  ` number.

**Return data type**

`  STRING  `

**Example**

The following example returns the octal representation of the number 10:

``` text
SELECT mysql.OCT(10) as octal_value;

/*
+-------------+
| octal_value |
+-------------+
| 12          |
+-------------+
*/
```

## `     mysql.ORD    `

``` text
mysql.ORD(string_expression)
```

**Description**

Returns the numeric code of the leftmost character in a string. If the string is empty, the function returns the ASCII null character `  0  ` .

This function supports the following argument:

  - `  string_expression  ` : The input `  STRING  ` .

**Return data type**

`  INT64  `

**Example**

The following example returns the character code for 'G':

``` text
SELECT mysql.ORD('Google') as char_code;

/*
+-----------+
| char_code |
+-----------+
| 71        |
+-----------+
*/
```

## `     mysql.POSITION    `

``` text
mysql.POSITION(substring, string)
```

**Description**

Alias for [`  LOCATE  `](#locate) . For more information, see the [`  LOCATE  `](#locate) entry.

## `     mysql.QUOTE    `

``` text
mysql.QUOTE(string_expression)
```

**Description**

Escapes a string for safe use as a string literal in a SQL statement by enclosing it in quotes and escaping special characters within the string.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` to quote. If the input is `  NULL  ` , the result is `  NULL  ` .

**Return data type**

`  STRING  `

**Differences from MySQL**

This function encloses the string in double quotes ( `  "  ` ), while MySQL typically uses single quotes ( `  '  ` ) for this purpose. Both forms generally result in valid SQL string literals.

**Example**

The following example quotes a string containing a single quote and backslash:

``` text
SELECT mysql.QUOTE("Don't \do it!") as quoted_string;

/*
+------------------------+
| quoted_string          |
+------------------------+
| "Don't \\do it!"       |
+------------------------+
*/
```

## `     mysql.REGEXP_LIKE    `

``` text
mysql.REGEXP_LIKE(string_expression, regular_expression[, match_type])
```

**Description**

Checks if a string matches a regular expression pattern.

This function supports the following arguments:

  - `  string_expression  ` : The input `  STRING  ` .
  - `  regular_expression  ` : The regular expression `  STRING  ` pattern.
  - `  match_type  ` (Optional): A `  STRING  ` specifying the match behavior. Defaults to `  'i'  ` (case-insensitive). Supported values:
      - `  'i'  ` : Case-insensitive matching.
      - `  'c'  ` : Case-sensitive matching.
      - `  'u'  ` or `  'mu'  ` or `  'um'  ` : Multi-line mode (lines split by `  \n  ` ), case-insensitive.
      - `  'un'  ` or `  'nu'  ` : The `  .  ` character matches newlines, case-insensitive.

**Return data type**

`  BOOL  `

**Limitations**

  - The `  match_type  ` 'm' (multiline supporting any Unicode line-separating character) is not supported.
  - Except as listed for `  'u'  ` and `  'un'  ` (and their permutations), different match types cannot be combined by concatenating their characters.

**Example**

The following example checks if the string "New day" starts with "new" case-insensitively:

``` text
SELECT mysql.REGEXP_LIKE('New day', '^new', 'i') as is_match;

/*
+----------+
| is_match |
+----------+
| true     |
+----------+
*/
```

## `     mysql.REGEXP_SUBSTR    `

``` text
mysql.REGEXP_SUBSTR(string_expression, regular_expression)
```

**Description**

Returns the substring that matches a regular expression pattern within an input string.

This function supports the following arguments:

  - `  string_expression  ` : The input `  STRING  ` .
  - `  regular_expression  ` : The regular expression `  STRING  ` pattern.

**Return data type**

`  STRING  `

**Differences from MySQL**

This function uses an underlying regular expression engine based on the re2 library, which may have minor differences in behavior compared to MySQL's regular expression implementation.

**Limitations**

This function does not support the optional `  pos  ` (position), `  occurrence  ` , and `  match_type  ` arguments that MySQL's `  REGEXP_SUBSTR  ` supports. The matching is implicitly case-insensitive and extracts the first occurrence.

**Example**

The following example extracts the first word starting with 'C' from a string:

``` text
SELECT mysql.REGEXP_SUBSTR('Google Cloud Platform', 'C\\w*') as substring_match;

/*
+-----------------+
| substring_match |
+-----------------+
| Cloud           |
+-----------------+
*/
```

## `     mysql.SPACE    `

``` text
mysql.SPACE(numeric_expression)
```

**Description**

Returns a string consisting of a specified number of space characters.

This function supports the following argument:

  - `  numeric_expression  ` : The `  INT64  ` number of spaces to return. If `  numeric_expression  ` is less than 0, an empty string is returned.

**Return data type**

`  STRING  `

**Limitations**

This function can produce a string of spaces up to approximately 1MB in size. Requesting a number of spaces that would exceed this limit may result in an error.

**Example**

The following example returns a string of 5 spaces:

``` text
SELECT CONCAT('Hello', mysql.SPACE(3), 'World') as three_spaces;

/*
+---------------+
| three_spaces  |
+---------------+
| Hello   World |
+---------------+
*/
```

## `     mysql.STRCMP    `

``` text
mysql.STRCMP(string_expression1, string_expression2)
```

**Description**

Compares two strings lexicographically. Returns 0 if the strings are identical, -1 if the first string is less than the second, and 1 if the first string is greater than the second. Returns `  NULL  ` if either string is `  NULL  ` .

This function supports the following arguments:

  - `  string_expression1  ` : The first `  STRING  ` to compare.
  - `  string_expression2  ` : The second `  STRING  ` to compare.

**Return data type**

`  INT64  `

**Differences from MySQL**

MySQL's `  STRCMP  ` supports comparison of both string and binary types.

**Limitations**

This function only supports `  STRING  ` type inputs.

**Example**

The following example compares "apple" and "banana":

``` text
SELECT mysql.STRCMP('apple', 'banana') as comparison_result;

/*
+-------------------+
| comparison_result |
+-------------------+
| -1                |
+-------------------+
*/
```

## `     mysql.SUBSTRING_INDEX    `

``` text
mysql.SUBSTRING_INDEX(string_expression, delimiter, count)
```

**Description**

Returns a substring from a string before or after a specified number of occurrences of a delimiter. The match for the delimiter is case-sensitive.

This function supports the following arguments:

  - `  string_expression  ` : The input `  STRING  ` .
  - `  delimiter  ` : The delimiter `  STRING  ` . If `  delimiter  ` is an empty string, the function returns an empty string.
  - `  count  ` : An `  INT64  ` specifying the number of occurrences of `  delimiter  ` . If `  count  ` is positive, everything to the left of the final delimiter (counting from the left) is returned. If `  count  ` is negative, everything to the right of the final delimiter (counting from the right) is returned.

**Return data type**

`  STRING  `

**Example**

The following example extracts parts of a string using different counts:

``` text
SELECT
  mysql.SUBSTRING_INDEX('[www.google.com](https://www.google.com)', '.', 2) as part1,
  mysql.SUBSTRING_INDEX('[www.google.com](https://www.google.com)', '.', -2) as part2;

/*
+--------------+-------------+
| part1        | part2       |
+--------------+-------------+
| www.google   | google.com  |
+--------------+-------------+
*/
```

## `     mysql.UNHEX    `

``` text
mysql.UNHEX(string_expression)
```

**Description**

Converts a string containing a hexadecimal representation of characters back to the original characters (as `  BYTES  ` ). Each pair of hexadecimal digits in the input string is interpreted as a number, which is then converted to its character equivalent.

This function supports the following argument:

  - `  string_expression  ` : The input `  STRING  ` representing a hexadecimal number.

**Return data type**

`  BYTES  `

**Limitations**

If the input string contains any non-hexadecimal characters, the behavior might result in an error or partial conversion, depending on the underlying `  FROM_HEX  ` implementation.

**Example**

The following example converts the hexadecimal string "53514C" back to characters:

``` text
SELECT mysql.UNHEX('53514C') as original_bytes;

/*
+----------------+
| original_bytes |
+----------------+
| SQL            |
+----------------+
*/
```
