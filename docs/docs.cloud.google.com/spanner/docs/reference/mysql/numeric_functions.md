Spanner supports the following MySQL numeric functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

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
<td><a href="#degrees"><code dir="ltr" translate="no">        mysql.DEGREES       </code></a></td>
<td>Converts radians in to degrees.</td>
</tr>
<tr class="even">
<td><a href="#log2"><code dir="ltr" translate="no">        mysql.LOG2       </code></a></td>
<td>Returns the base-2 logarithm of the input parameter. Returns <code dir="ltr" translate="no">       NULL      </code> if the input parameter is out of range.</td>
</tr>
<tr class="odd">
<td><a href="#pi"><code dir="ltr" translate="no">        mysql.PI       </code></a></td>
<td>Returns the value of pi (π).</td>
</tr>
<tr class="even">
<td><a href="#radians"><code dir="ltr" translate="no">        mysql.RADIANS       </code></a></td>
<td>Converts degrees in to radians.</td>
</tr>
<tr class="odd">
<td><a href="#truncate"><code dir="ltr" translate="no">        mysql.TRUNCATE       </code></a></td>
<td>Truncates an input parameter to a specified number of decimal places.</td>
</tr>
</tbody>
</table>

## `     mysql.DEGREES    `

``` text
mysql.DEGREES(numeric_expression)
```

**Description**

Converts an angle value from radians to degrees.

This function supports the following argument:

  - `  numeric_expression  ` : The angle in radians, specified as a `  FLOAT64  ` value.

**Return data type**

`  FLOAT64  `

**Limitations**

If you provide a very large input value (greater than approximately 1e300), the function may result in an overflow error.

**Example**

The following example converts π radians to degrees:

``` text
SELECT mysql.DEGREES(ACOS(-1)) as pi_in_degrees;

/*
+---------------+
| pi_in_degrees |
+---------------+
| 180.0         |
+---------------+
*/
```

## `     mysql.LOG2    `

``` text
mysql.LOG2(numeric_expression)
```

**Description**

Calculates the base-2 logarithm of a numeric value.

This function supports the following argument:

  - `  numeric_expression  ` : The `  FLOAT64  ` value for which to calculate the base-2 logarithm.

**Return data type**

`  FLOAT64  `

**Limitations**

The input value `  numeric_expression  ` must be greater than zero. If `  numeric_expression  ` is zero or negative, the function returns `  NULL  ` .

**Example**

The following example calculates the base-2 logarithm of 8:

``` text
SELECT mysql.LOG2(8) AS log2_of_8;

/*
+-----------+
| log2_of_8 |
+-----------+
| 3.0       |
+-----------+
*/
```

## `     mysql.PI    `

``` text
mysql.PI()
```

**Description**

Returns the mathematical constant pi (π).

This function doesn't support any arguments.

**Return data type**

`  FLOAT64  `

**Example**

The following example returns the value of π:

``` text
SELECT mysql.PI() as pi_value;

/*
+-------------------+
| pi_value          |
+-------------------+
| 3.141592653589793 |
+-------------------+
*/
```

## `     mysql.RADIANS    `

``` text
mysql.RADIANS(numeric_expression)
```

**Description**

Converts an angle value from degrees to radians.

This function supports the following argument:

  - `  numeric_expression  ` : The angle in degrees, specified as a `  FLOAT64  ` value.

**Return data type**

`  FLOAT64  `

**Example**

The following example converts 180 degrees to radians:

``` text
SELECT mysql.RADIANS(180) as radians_value;

/*
+-------------------+
| radians_value     |
+-------------------+
| 3.141592653589793 |
+-------------------+
*/
```

## `     mysql.TRUNCATE    `

``` text
mysql.TRUNCATE(numeric_expression, precision)
```

**Description**

Truncates a number to a specified number of decimal places. This function does not perform rounding.

This function supports the following arguments:

  - `  numeric_expression  ` : The `  FLOAT64  ` value to truncate.
  - `  precision  ` : The `  INT64  ` value specifying the number of decimal places to preserve. If `  precision  ` is positive, it truncates to `  precision  ` decimal places. If `  precision  ` is zero, it truncates to the nearest whole number towards zero. If `  precision  ` is negative, it makes `  precision  ` digits to the left of the decimal point zero.

**Return data type**

`  FLOAT64  `

**Differences from MySQL**

This function's behavior with two arguments, `  numeric_expression  ` (the number) and `  precision  ` (the number of decimal places), is similar to MySQL's `  TRUNCATE(numeric_expression, precision)  ` function.

**Limitations**

`  TRUNCATE  ` is a reserved keyword. If you use this function in Data Definition Language (DDL) statements, such as in generated column definitions, you must enclose the function name in backticks (for example, ``  mysql.`TRUNCATE`  `` ).

**Example**

The following example demonstrates various uses of the `  TRUNCATE  ` function:

``` text
SELECT
  mysql.TRUNCATE(123.4567, 2) as truncate_2_decimals,
  mysql.TRUNCATE(123.987, 0) as truncate_to_integer,
  mysql.TRUNCATE(123.456, -1) as truncate_tens_place,
  mysql.TRUNCATE(-123.456, 1) as truncate_negative_num;

/*
+---------------------+---------------------+---------------------+-----------------------+
| truncate_2_decimals | truncate_to_integer | truncate_tens_place | truncate_negative_num |
+---------------------+---------------------+---------------------+-----------------------+
| 123.45              | 123.0               | 120.0               | -123.4                |
+---------------------+---------------------+---------------------+-----------------------+
*/
```
