GoogleSQL for Spanner supports mathematical functions. All mathematical functions have the following behaviors:

  - They return `NULL` if any of the input parameters is `NULL` .
  - They return `NaN` if any of the arguments is `NaN` .

## Categories

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Category</th>
<th>Functions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Trigonometric</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#acos"><code dir="ltr" translate="no">ACOS</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#acosh"><code dir="ltr" translate="no">ACOSH</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#asin"><code dir="ltr" translate="no">ASIN</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#asinh"><code dir="ltr" translate="no">ASINH</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atan"><code dir="ltr" translate="no">ATAN</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atan2"><code dir="ltr" translate="no">ATAN2</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atanh"><code dir="ltr" translate="no">ATANH</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cos"><code dir="ltr" translate="no">COS</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cosh"><code dir="ltr" translate="no">COSH</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sin"><code dir="ltr" translate="no">SIN</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sinh"><code dir="ltr" translate="no">SINH</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#tan"><code dir="ltr" translate="no">TAN</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#tanh"><code dir="ltr" translate="no">TANH</code></a></td>
</tr>
<tr class="even">
<td>Exponential and<br />
logarithmic</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#exp"><code dir="ltr" translate="no">EXP</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ln"><code dir="ltr" translate="no">LN</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#log"><code dir="ltr" translate="no">LOG</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#log10"><code dir="ltr" translate="no">LOG10</code></a></td>
</tr>
<tr class="odd">
<td>Rounding and<br />
truncation</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ceil"><code dir="ltr" translate="no">CEIL</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ceiling"><code dir="ltr" translate="no">CEILING</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#floor"><code dir="ltr" translate="no">FLOOR</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#round"><code dir="ltr" translate="no">ROUND</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#trunc"><code dir="ltr" translate="no">TRUNC</code></a></td>
</tr>
<tr class="even">
<td>Power and<br />
root</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#pow"><code dir="ltr" translate="no">POW</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#power"><code dir="ltr" translate="no">POWER</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sqrt"><code dir="ltr" translate="no">SQRT</code></a></td>
</tr>
<tr class="odd">
<td>Sign</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#abs"><code dir="ltr" translate="no">ABS</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sign"><code dir="ltr" translate="no">SIGN</code></a></td>
</tr>
<tr class="even">
<td>Distance</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product"><code dir="ltr" translate="no">APPROX_DOT_PRODUCT</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance"><code dir="ltr" translate="no">APPROX_COSINE_DISTANCE</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance"><code dir="ltr" translate="no">APPROX_EUCLIDEAN_DISTANCE</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#dot_product"><code dir="ltr" translate="no">DOT_PRODUCT</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cosine_distance"><code dir="ltr" translate="no">COSINE_DISTANCE</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#euclidean_distance"><code dir="ltr" translate="no">EUCLIDEAN_DISTANCE</code></a></td>
</tr>
<tr class="odd">
<td>Comparison</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#greatest"><code dir="ltr" translate="no">GREATEST</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#least"><code dir="ltr" translate="no">LEAST</code></a></td>
</tr>
<tr class="even">
<td>Arithmetic and error handling</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#div"><code dir="ltr" translate="no">DIV</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ieee_divide"><code dir="ltr" translate="no">IEEE_DIVIDE</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#is_inf"><code dir="ltr" translate="no">IS_INF</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#is_nan"><code dir="ltr" translate="no">IS_NAN</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#mod"><code dir="ltr" translate="no">MOD</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_add"><code dir="ltr" translate="no">SAFE_ADD</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_divide"><code dir="ltr" translate="no">SAFE_DIVIDE</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_multiply"><code dir="ltr" translate="no">SAFE_MULTIPLY</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_negate"><code dir="ltr" translate="no">SAFE_NEGATE</code></a> <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_subtract"><code dir="ltr" translate="no">SAFE_SUBTRACT</code></a></td>
</tr>
</tbody>
</table>

## Function list

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#abs"><code dir="ltr" translate="no">ABS</code></a></td>
<td>Computes the absolute value of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#acos"><code dir="ltr" translate="no">ACOS</code></a></td>
<td>Computes the inverse cosine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#acosh"><code dir="ltr" translate="no">ACOSH</code></a></td>
<td>Computes the inverse hyperbolic cosine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance"><code dir="ltr" translate="no">APPROX_COSINE_DISTANCE</code></a></td>
<td>Computes the approximate cosine distance between two vectors.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product"><code dir="ltr" translate="no">APPROX_DOT_PRODUCT</code></a></td>
<td>Computes the approximate dot product of two vectors.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance"><code dir="ltr" translate="no">APPROX_EUCLIDEAN_DISTANCE</code></a></td>
<td>Computes the approximate Euclidean distance between two vectors.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#asin"><code dir="ltr" translate="no">ASIN</code></a></td>
<td>Computes the inverse sine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#asinh"><code dir="ltr" translate="no">ASINH</code></a></td>
<td>Computes the inverse hyperbolic sine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atan"><code dir="ltr" translate="no">ATAN</code></a></td>
<td>Computes the inverse tangent of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atan2"><code dir="ltr" translate="no">ATAN2</code></a></td>
<td>Computes the inverse tangent of <code dir="ltr" translate="no">X/Y</code> , using the signs of <code dir="ltr" translate="no">X</code> and <code dir="ltr" translate="no">Y</code> to determine the quadrant.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atanh"><code dir="ltr" translate="no">ATANH</code></a></td>
<td>Computes the inverse hyperbolic tangent of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions#avg"><code dir="ltr" translate="no">AVG</code></a></td>
<td>Gets the average of non- <code dir="ltr" translate="no">NULL</code> values.<br />
For more information, see <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions">Aggregate functions</a> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ceil"><code dir="ltr" translate="no">CEIL</code></a></td>
<td>Gets the smallest integral value that isn't less than <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ceiling"><code dir="ltr" translate="no">CEILING</code></a></td>
<td>Synonym of <code dir="ltr" translate="no">CEIL</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cos"><code dir="ltr" translate="no">COS</code></a></td>
<td>Computes the cosine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cosh"><code dir="ltr" translate="no">COSH</code></a></td>
<td>Computes the hyperbolic cosine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cosine_distance"><code dir="ltr" translate="no">COSINE_DISTANCE</code></a></td>
<td>Computes the cosine distance between two vectors.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#div"><code dir="ltr" translate="no">DIV</code></a></td>
<td>Divides integer <code dir="ltr" translate="no">X</code> by integer <code dir="ltr" translate="no">Y</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#dot_product"><code dir="ltr" translate="no">DOT_PRODUCT</code></a></td>
<td>Computes the dot product of two vectors.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#exp"><code dir="ltr" translate="no">EXP</code></a></td>
<td>Computes <code dir="ltr" translate="no">e</code> to the power of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#euclidean_distance"><code dir="ltr" translate="no">EUCLIDEAN_DISTANCE</code></a></td>
<td>Computes the Euclidean distance between two vectors.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#floor"><code dir="ltr" translate="no">FLOOR</code></a></td>
<td>Gets the largest integral value that isn't greater than <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#greatest"><code dir="ltr" translate="no">GREATEST</code></a></td>
<td>Gets the greatest value among <code dir="ltr" translate="no">X1,...,XN</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ieee_divide"><code dir="ltr" translate="no">IEEE_DIVIDE</code></a></td>
<td>Divides <code dir="ltr" translate="no">X</code> by <code dir="ltr" translate="no">Y</code> , but doesn't generate errors for division by zero or overflow.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#is_inf"><code dir="ltr" translate="no">IS_INF</code></a></td>
<td>Checks if <code dir="ltr" translate="no">X</code> is positive or negative infinity.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#is_nan"><code dir="ltr" translate="no">IS_NAN</code></a></td>
<td>Checks if <code dir="ltr" translate="no">X</code> is a <code dir="ltr" translate="no">NaN</code> value.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#least"><code dir="ltr" translate="no">LEAST</code></a></td>
<td>Gets the least value among <code dir="ltr" translate="no">X1,...,XN</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#ln"><code dir="ltr" translate="no">LN</code></a></td>
<td>Computes the natural logarithm of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#log"><code dir="ltr" translate="no">LOG</code></a></td>
<td>Computes the natural logarithm of <code dir="ltr" translate="no">X</code> or the logarithm of <code dir="ltr" translate="no">X</code> to base <code dir="ltr" translate="no">Y</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#log10"><code dir="ltr" translate="no">LOG10</code></a></td>
<td>Computes the natural logarithm of <code dir="ltr" translate="no">X</code> to base 10.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions#max"><code dir="ltr" translate="no">MAX</code></a></td>
<td>Gets the maximum non- <code dir="ltr" translate="no">NULL</code> value.<br />
For more information, see <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions">Aggregate functions</a> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#mod"><code dir="ltr" translate="no">MOD</code></a></td>
<td>Gets the remainder of the division of <code dir="ltr" translate="no">X</code> by <code dir="ltr" translate="no">Y</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#pow"><code dir="ltr" translate="no">POW</code></a></td>
<td>Produces the value of <code dir="ltr" translate="no">X</code> raised to the power of <code dir="ltr" translate="no">Y</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#power"><code dir="ltr" translate="no">POWER</code></a></td>
<td>Synonym of <code dir="ltr" translate="no">POW</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#round"><code dir="ltr" translate="no">ROUND</code></a></td>
<td>Rounds <code dir="ltr" translate="no">X</code> to the nearest integer or rounds <code dir="ltr" translate="no">X</code> to <code dir="ltr" translate="no">N</code> decimal places after the decimal point.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_add"><code dir="ltr" translate="no">SAFE_ADD</code></a></td>
<td>Equivalent to the addition operator ( <code dir="ltr" translate="no">X + Y</code> ), but returns <code dir="ltr" translate="no">NULL</code> if overflow occurs.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_divide"><code dir="ltr" translate="no">SAFE_DIVIDE</code></a></td>
<td>Equivalent to the division operator ( <code dir="ltr" translate="no">X / Y</code> ), but returns <code dir="ltr" translate="no">NULL</code> if an error occurs.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_multiply"><code dir="ltr" translate="no">SAFE_MULTIPLY</code></a></td>
<td>Equivalent to the multiplication operator ( <code dir="ltr" translate="no">X * Y</code> ), but returns <code dir="ltr" translate="no">NULL</code> if overflow occurs.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_negate"><code dir="ltr" translate="no">SAFE_NEGATE</code></a></td>
<td>Equivalent to the unary minus operator ( <code dir="ltr" translate="no">-X</code> ), but returns <code dir="ltr" translate="no">NULL</code> if overflow occurs.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_subtract"><code dir="ltr" translate="no">SAFE_SUBTRACT</code></a></td>
<td>Equivalent to the subtraction operator ( <code dir="ltr" translate="no">X - Y</code> ), but returns <code dir="ltr" translate="no">NULL</code> if overflow occurs.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sign"><code dir="ltr" translate="no">SIGN</code></a></td>
<td>Produces -1 , 0, or +1 for negative, zero, and positive arguments respectively.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sin"><code dir="ltr" translate="no">SIN</code></a></td>
<td>Computes the sine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sinh"><code dir="ltr" translate="no">SINH</code></a></td>
<td>Computes the hyperbolic sine of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sqrt"><code dir="ltr" translate="no">SQRT</code></a></td>
<td>Computes the square root of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions#sum"><code dir="ltr" translate="no">SUM</code></a></td>
<td>Gets the sum of non- <code dir="ltr" translate="no">NULL</code> values.<br />
For more information, see <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions">Aggregate functions</a> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#tan"><code dir="ltr" translate="no">TAN</code></a></td>
<td>Computes the tangent of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#tanh"><code dir="ltr" translate="no">TANH</code></a></td>
<td>Computes the hyperbolic tangent of <code dir="ltr" translate="no">X</code> .</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#trunc"><code dir="ltr" translate="no">TRUNC</code></a></td>
<td>Rounds a number like <code dir="ltr" translate="no">ROUND(X)</code> or <code dir="ltr" translate="no">ROUND(X, N)</code> , but always rounds towards zero and never overflows.</td>
</tr>
</tbody>
</table>

## `ABS`

    ABS(X)

**Description**

Computes absolute value. Returns an error if the argument is an integer and the output value can't be represented as the same type; this happens only for the largest negative input value, which has no positive representation.

| X      | ABS(X) |
| ------ | ------ |
| 25     | 25     |
| \-25   | 25     |
| `+inf` | `+inf` |
| `-inf` | `+inf` |

**Return Data Type**

| INPUT  | `INT64` | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | ------- | --------- | --------- | --------- |
| OUTPUT | `INT64` | `NUMERIC` | `FLOAT32` | `FLOAT64` |

## `ACOS`

    ACOS(X)

**Description**

Computes the principal value of the inverse cosine of X. The return value is in the range \[0,π\]. Generates an error if X is a value outside of the range \[-1, 1\].

If X is `NUMERIC` then, the output is `FLOAT64` .

| X       | ACOS(X) |
| ------- | ------- |
| `+inf`  | `NaN`   |
| `-inf`  | `NaN`   |
| `NaN`   | `NaN`   |
| X \< -1 | Error   |
| X \> 1  | Error   |

## `ACOSH`

    ACOSH(X)

**Description**

Computes the inverse hyperbolic cosine of X. Generates an error if X is a value less than 1.

If X is `NUMERIC` then, the output is `FLOAT64` .

| X      | ACOSH(X) |
| ------ | -------- |
| `+inf` | `+inf`   |
| `-inf` | `NaN`    |
| `NaN`  | `NaN`    |
| X \< 1 | Error    |

## `APPROX_COSINE_DISTANCE`

    APPROX_COSINE_DISTANCE(vector1, vector2, options=>value)

**Description**

Computes the approximate [cosine distance](https://en.wikipedia.org/wiki/Cosine_similarity#Cosine_distance) between two vectors.

**Definitions**

  - `vector1` : A vector that's represented by an `ARRAY<T>` value.

  - `vector2` : A vector that's represented by an `ARRAY<T>` value.

  - `options` : A named argument with a value that represents a Spanner-specific optimization. `value` must be the following:
    
      - `JSON'{"num_leaves_to_search": INT}'`
    
    This option specifies the approximate nearest neighbors (ANN) algorithm configuration used in your query. The total number of leaves is specified when you create your vector index. For this argument, we recommend using a number that's 1% the total number of leaves defined in the `CREATE VECTOR INDEX` statement. The number of leaves to search is defined by the `num_leaves_to_search` option for both 2-level and 3-level trees.
    
    If an unsupported option is provided, an error is produced.

**Details**

`APPROX_COSINE_DISTANCE` approximates the [`COSINE_DISTANCE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cosine_distance) between the given vectors. Approximation typically occurs when using specific indexing strategies that precompute clustering.

Query results across invocations aren't guaranteed to repeat.

You can add a filter such as `WHERE s.id = 42` to your query. However, that might lead to poor recall problems because the `WHERE` filter happens after internal limits are applied. To mitigate this issue, you can increase the value of the `num_of_leaves_to_search` option.

  - `ARRAY<T>` can be used to represent a vector. Each zero-based index in this array represents a dimension. The value for each element in this array represents a magnitude.
    
    `T` can represent the following and must be the same for both vectors:
    
      - `FLOAT32`
      - `FLOAT64`
    
    In the following example vector, there are four dimensions. The magnitude is `10.0` for dimension `0` , `55.0` for dimension `1` , `40.0` for dimension `2` , and `34.0` for dimension `3` :
    
        [10.0, 55.0, 40.0, 34.0]

  - Both vectors in this function must share the same dimensions, and if they don't, an error is produced.

  - A vector can't be a zero vector. A vector is a zero vector if it has no dimensions or all dimensions have a magnitude of `0` , such as `[]` or `[0.0, 0.0]` . If a zero vector is encountered, an error is produced.

  - An error is produced if a magnitude in a vector is `NULL` .

  - If a vector is `NULL` , `NULL` is returned.

**Limitations**

  - The function can only be used to sort vectors in a table with an `ORDER BY` clause.

  - The function output must be the only ordering key in the `ORDER BY` clause.

  - The `ORDER BY` clause must be followed by a `LIMIT` clause.

  - One of the function arguments must directly reference an embedding column, and the other must be a constant expression, such as a query parameter reference.

  - You can't use the function in the following ways:
    
      - In a `WHERE` , `ON` , or `GROUP BY` clause.
    
      - In a `SELECT` clause unless it's for ordering results in a later `ORDER BY` clause.
    
      - As the input of another expression.

**Return type**

`FLOAT64`

**Examples**

In the following example, vectors are used to compute the approximate cosine distance:

In the following example, up to 1000 leaves in the vector index are searched to produce the approximate nearest two vectors using cosine distance:

    SELECT FirstName, LastName
    FROM Singers@{FORCE_INDEX=Singer_vector_index} AS s
    ORDER BY APPROX_COSINE_DISTANCE(@queryVector, s.embedding, options=>JSON'{"num_leaves_to_search": 1000}')
    LIMIT 2;
    
    /*-----------+------------+
     | FirstName | LastName   |
     +-----------+------------+
     | Marc      | Richards   |
     | Catalina  | Smith      |
     +-----------+------------*/

## `APPROX_DOT_PRODUCT`

    APPROX_DOT_PRODUCT(vector1, vector2, options=>value)

**Description**

Computes the approximate [dot product](https://mathworld.wolfram.com/DotProduct.html) of two vectors.

**Definitions**

  - `vector1` : A vector that's represented by an `ARRAY<T>` value.

  - `vector2` : A vector that's represented by an `ARRAY<T>` value.

  - `options` : A named argument with a value that represents a Spanner-specific optimization. `value` must be the following:
    
      - `JSON'{"num_leaves_to_search": INT}'`
    
    This option specifies the approximate nearest neighbors (ANN) algorithm configuration used in your query. The total number of leaves is specified when you create your vector index. For this argument, we recommend using a number that's 1% the total number of leaves defined in the `CREATE VECTOR INDEX` statement. The number of leaves to search is defined by the `num_leaves_to_search` option for both 2-level and 3-level trees.
    
    If an unsupported option is provided, an error is produced.

**Details**

`APPROX_DOT_PRODUCT` approximates the [`DOT_PRODUCT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#dot_product) between two vectors. Approximation typically occurs when using specific indexing strategies that precompute clustering.

Query results across invocations aren't guaranteed to repeat.

You can add a filter such as `WHERE s.id = 42` to your query. However, that might lead to poor recall problems because the `WHERE` filter happens after internal limits are applied. To mitigate this issue, you can increase the value of the `num_of_leaves_to_search` option.

  - `ARRAY<T>` can be used to represent a vector. Each zero-based index in this array represents a dimension. The value for each element in this array represents a magnitude.
    
    `T` can represent the following and must be the same for both vectors:
    
      - `INT64`
      - `FLOAT32`
      - `FLOAT64`
    
    In the following example vector, there are four dimensions. The magnitude is `10.0` for dimension `0` , `55.0` for dimension `1` , `40.0` for dimension `2` , and `34.0` for dimension `3` :
    
        [10.0, 55.0, 40.0, 34.0]

  - Both vectors in this function must share the same dimensions, and if they don't, an error is produced.

  - A vector can be a zero vector. A vector is a zero vector if it has no dimensions or all dimensions have a magnitude of `0` , such as `[]` or `[0.0, 0.0]` .

  - An error is produced if a magnitude in a vector is `NULL` .

  - If a vector is `NULL` , `NULL` is returned.

**Limitations**

  - The function can only be used to sort vectors in a table with an `ORDER BY` clause.

  - The function output must be the only ordering key in the `ORDER BY` clause.

  - The `ORDER BY` clause must be followed by a `LIMIT` clause.

  - One of the function arguments must directly reference an embedding column, and the other must be a constant expression, such as a query parameter reference.

  - You can't use the function in the following ways:
    
      - In a `WHERE` , `ON` , or `GROUP BY` clause.
    
      - In a `SELECT` clause unless it's for ordering results in a later `ORDER BY` clause.
    
      - As the input of another expression.

**Return type**

`FLOAT64`

**Examples**

In the following example, up to 1000 leaves in the vector index are searched to produce the approximate nearest two vectors using dot product distance:

    SELECT FirstName, LastName
    FROM Singers@{FORCE_INDEX=Singer_vector_index} AS s
    ORDER BY APPROX_DOT_PRODUCT(@queryVector, s.embedding, options=>JSON'{"num_leaves_to_search": 1000}') DESC
    LIMIT 2;
    
    /*-----------+------------+
     | FirstName | LastName   |
     +-----------+------------+
     | Marc      | Richards   |
     | Catalina  | Smith      |
     +-----------+------------*/

## `APPROX_EUCLIDEAN_DISTANCE`

    APPROX_EUCLIDEAN_DISTANCE(vector1, vector2, options=>value)

**Description**

Computes the approximate [Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance) between two vectors.

**Definitions**

  - `vector1` : A vector that's represented by an `ARRAY<T>` value.

  - `vector2` : A vector that's represented by an `ARRAY<T>` value.

  - `options` : A named argument with a value that represents a Spanner-specific optimization. `value` must be the following:
    
      - `JSON'{"num_leaves_to_search": INT}'`
    
    This option specifies the approximate nearest neighbors (ANN) algorithm configuration used in your query. The total number of leaves is specified when you create your vector index. For this argument, we recommend using a number that's 1% the total number of leaves defined in the `CREATE VECTOR INDEX` statement. The number of leaves to search is defined by the `num_leaves_to_search` option for both 2-level and 3-level trees.
    
    If an unsupported option is provided, an error is produced.

**Details**

`APPROX_EUCLIDEAN_DISTANCE` approximates the [`EUCLIDEAN_DISTANCE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#euclidean_distance) between two vectors. Approximation typically occurs when using specific indexing strategies that precompute clustering.

Query results across invocations aren't guaranteed to repeat.

You can add a filter such as `WHERE s.id = 42` to your query. However, that might lead to poor recall problems because the `WHERE` filter happens after internal limits are applied. To mitigate this issue, you can increase the value of the `num_of_leaves_to_search` option.

  - `ARRAY<T>` can be used to represent a vector. Each zero-based index in this array represents a dimension. The value for each element in this array represents a magnitude.
    
    `T` can represent the following and must be the same for both vectors:
    
      - `FLOAT32`
      - `FLOAT64`
    
    In the following example vector, there are four dimensions. The magnitude is `10.0` for dimension `0` , `55.0` for dimension `1` , `40.0` for dimension `2` , and `34.0` for dimension `3` :
    
        [10.0, 55.0, 40.0, 34.0]

  - Both vectors in this function must share the same dimensions, and if they don't, an error is produced.

  - A vector can be a zero vector. A vector is a zero vector if it has no dimensions or all dimensions have a magnitude of `0` , such as `[]` or `[0.0, 0.0]` .

  - An error is produced if a magnitude in a vector is `NULL` .

  - If a vector is `NULL` , `NULL` is returned.

**Limitations**

  - The function can only be used to sort vectors in a table with an `ORDER BY` clause.

  - The function output must be the only ordering key in the `ORDER BY` clause.

  - The `ORDER BY` clause must be followed by a `LIMIT` clause.

  - One of the function arguments must directly reference an embedding column, and the other must be a constant expression, such as a query parameter reference.

  - You can't use the function in the following ways:
    
      - In a `WHERE` , `ON` , or `GROUP BY` clause.
    
      - In a `SELECT` clause unless it's for ordering results in a later `ORDER BY` clause.
    
      - As the input of another expression.

**Return type**

`FLOAT64`

**Examples**

In the following example, vectors are used to compute the approximate Euclidean distance:

In the following example, up to 1000 leaves in the vector index are searched to produce the approximate nearest two vectors using Euclidean distance:

    SELECT FirstName, LastName
    FROM Singers@{FORCE_INDEX=Singer_vector_index} AS s
    ORDER BY APPROX_EUCLIDEAN_DISTANCE(@queryVector, 0.1], s.embedding, options=>JSON'{"num_leaves_to_search": 1000}')
    LIMIT 2;
    
    /*-----------+------------+
     | FirstName | LastName   |
     +-----------+------------+
     | Marc      | Richards   |
     | Catalina  | Smith      |
     +-----------+------------*/

## `ASIN`

    ASIN(X)

**Description**

Computes the principal value of the inverse sine of X. The return value is in the range \[-π/2,π/2\]. Generates an error if X is outside of the range \[-1, 1\].

If X is `NUMERIC` then, the output is `FLOAT64` .

| X       | ASIN(X) |
| ------- | ------- |
| `+inf`  | `NaN`   |
| `-inf`  | `NaN`   |
| `NaN`   | `NaN`   |
| X \< -1 | Error   |
| X \> 1  | Error   |

## `ASINH`

    ASINH(X)

**Description**

Computes the inverse hyperbolic sine of X. Doesn't fail.

If X is `NUMERIC` then, the output is `FLOAT64` .

| X      | ASINH(X) |
| ------ | -------- |
| `+inf` | `+inf`   |
| `-inf` | `-inf`   |
| `NaN`  | `NaN`    |

## `ATAN`

    ATAN(X)

**Description**

Computes the principal value of the inverse tangent of X. The return value is in the range \[-π/2,π/2\]. Doesn't fail.

If X is `NUMERIC` then, the output is `FLOAT64` .

| X      | ATAN(X) |
| ------ | ------- |
| `+inf` | π/2     |
| `-inf` | \-π/2   |
| `NaN`  | `NaN`   |

## `ATAN2`

    ATAN2(X, Y)

**Description**

Calculates the principal value of the inverse tangent of X/Y using the signs of the two arguments to determine the quadrant. The return value is in the range \[-π,π\].

If Y is `NUMERIC` then, the output is `FLOAT64` .

| X                     | Y            | ATAN2(X, Y) |
| --------------------- | ------------ | ----------- |
| `NaN`                 | Any value    | `NaN`       |
| Any value             | `NaN`        | `NaN`       |
| 0.0                   | 0.0          | 0.0         |
| Positive Finite value | `-inf`       | π           |
| Negative Finite value | `-inf`       | \-π         |
| Finite value          | `+inf`       | 0.0         |
| `+inf`                | Finite value | π/2         |
| `-inf`                | Finite value | \-π/2       |
| `+inf`                | `-inf`       | ¾π          |
| `-inf`                | `-inf`       | \-¾π        |
| `+inf`                | `+inf`       | π/4         |
| `-inf`                | `+inf`       | \-π/4       |

## `ATANH`

    ATANH(X)

**Description**

Computes the inverse hyperbolic tangent of X. Generates an error if X is outside of the range (-1, 1).

If X is `NUMERIC` then, the output is `FLOAT64` .

| X       | ATANH(X) |
| ------- | -------- |
| `+inf`  | `NaN`    |
| `-inf`  | `NaN`    |
| `NaN`   | `NaN`    |
| X \< -1 | Error    |
| X \> 1  | Error    |

## `CEIL`

    CEIL(X)

**Description**

Returns the smallest integral value that isn't less than X.

| X      | CEIL(X) |
| ------ | ------- |
| 2.0    | 2.0     |
| 2.3    | 3.0     |
| 2.8    | 3.0     |
| 2.5    | 3.0     |
| \-2.3  | \-2.0   |
| \-2.8  | \-2.0   |
| \-2.5  | \-2.0   |
| 0      | 0       |
| `+inf` | `+inf`  |
| `-inf` | `-inf`  |
| `NaN`  | `NaN`   |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `CEILING`

    CEILING(X)

**Description**

Synonym of CEIL(X)

## `COS`

    COS(X)

**Description**

Computes the cosine of X where X is specified in radians. Never fails.

| X      | COS(X) |
| ------ | ------ |
| `+inf` | `NaN`  |
| `-inf` | `NaN`  |
| `NaN`  | `NaN`  |

## `COSH`

    COSH(X)

**Description**

Computes the hyperbolic cosine of X where X is specified in radians. Generates an error if overflow occurs.

If X is `NUMERIC` then, the output is `FLOAT64` .

| X      | COSH(X) |
| ------ | ------- |
| `+inf` | `+inf`  |
| `-inf` | `+inf`  |
| `NaN`  | `NaN`   |

## `COSINE_DISTANCE`

    COSINE_DISTANCE(vector1, vector2)

**Description**

Computes the [cosine distance](https://en.wikipedia.org/wiki/Cosine_similarity#Cosine_distance) between two vectors.

**Definitions**

  - `vector1` : A vector that's represented by an `ARRAY<T>` value.
  - `vector2` : A vector that's represented by an `ARRAY<T>` value.

**Details**

  - `ARRAY<T>` can be used to represent a vector. Each zero-based index in this array represents a dimension. The value for each element in this array represents a magnitude.
    
    `T` can represent the following and must be the same for both vectors:
    
      - `FLOAT32`
      - `FLOAT64`
    
    In the following example vector, there are four dimensions. The magnitude is `10.0` for dimension `0` , `55.0` for dimension `1` , `40.0` for dimension `2` , and `34.0` for dimension `3` :
    
        [10.0, 55.0, 40.0, 34.0]

  - Both vectors in this function must share the same dimensions, and if they don't, an error is produced.

  - A vector can't be a zero vector. A vector is a zero vector if it has no dimensions or all dimensions have a magnitude of `0` , such as `[]` or `[0.0, 0.0]` . If a zero vector is encountered, an error is produced.

  - An error is produced if a magnitude in a vector is `NULL` .

  - If a vector is `NULL` , `NULL` is returned.

**Return type**

`FLOAT64`

**Examples**

In the following example,vectors are used to compute the cosine distance:

    SELECT COSINE_DISTANCE([1.0, 2.0], [3.0, 4.0]) AS results;
    
    /*----------+
     | results  |
     +----------+
     | 0.016130 |
     +----------*/

The ordering of numeric values in a vector doesn't impact the results produced by this function. For example these queries produce the same results even though the numeric values in each vector is in a different order:

    SELECT COSINE_DISTANCE([1.0, 2.0], [3.0, 4.0]) AS results;

    SELECT COSINE_DISTANCE([2.0, 1.0], [4.0, 3.0]) AS results;

``` 
 /*----------+
  | results  |
  +----------+
  | 0.016130 |
  +----------*/
```

In the following example, the function can't compute cosine distance against the first vector, which is a zero vector:

    -- ERROR
    SELECT COSINE_DISTANCE([0.0, 0.0], [3.0, 4.0]) AS results;

Both vectors must have the same dimensions. If not, an error is produced. In the following example, the first vector has two dimensions and the second vector has three:

    -- ERROR
    SELECT COSINE_DISTANCE([9.0, 7.0], [8.0, 4.0, 5.0]) AS results;

## `DIV`

    DIV(X, Y)

**Description**

Returns the result of integer division of X by Y. Division by zero returns an error. Division by -1 may overflow. If both inputs are `NUMERIC` and the result is overflow, then it returns a `numeric overflow` error.

| X  | Y   | DIV(X, Y) |
| -- | --- | --------- |
| 20 | 4   | 5         |
| 12 | \-7 | \-1       |
| 20 | 3   | 6         |
| 0  | 20  | 0         |
| 20 | 0   | Error     |

**Return Data Type**

The return data type is determined by the argument types with the following table.

| INPUT     | `INT64`   | `NUMERIC` |
| --------- | --------- | --------- |
| `INT64`   | `INT64`   | `NUMERIC` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` |

## `DOT_PRODUCT`

    DOT_PRODUCT(vector1, vector2)

**Description**

Computes the [dot product](https://mathworld.wolfram.com/DotProduct.html) of two vectors. The dot product is computed by summing the product of corresponding vector elements.

**Definitions**

  - `vector1` : A vector that's represented by an `ARRAY<T>` value.
  - `vector2` : A vector that's represented by an `ARRAY<T>` value.

**Details**

  - `ARRAY<T>` can be used to represent a vector. Each zero-based index in this array represents a dimension. The value for each element in this array represents a magnitude.
    
    `T` can represent the following and must be the same for both vectors:
    
      - `INT64`
      - `FLOAT32`
      - `FLOAT64`
    
    In the following example vector, there are four dimensions. The magnitude is `10.0` for dimension `0` , `55.0` for dimension `1` , `40.0` for dimension `2` , and `34.0` for dimension `3` :
    
        [10.0, 55.0, 40.0, 34.0]

  - Both vectors in this function must share the same dimensions, and if they don't, an error is produced.

  - A vector can be a zero vector. A vector is a zero vector if it has no dimensions or all dimensions have a magnitude of `0` , such as `[]` or `[0.0, 0.0]` .

  - An error is produced if a magnitude in a vector is `NULL` .

  - If a vector is `NULL` , `NULL` is returned.

**Return type**

`FLOAT64`

**Examples**

    SELECT DOT_PRODUCT([100], [200]) AS results
    
    /*---------+
     | results |
     +---------+
     | 20000   |
     +---------*/

    SELECT DOT_PRODUCT([100, 10], [200, 6]) AS results
    
    /*---------+
     | results |
     +---------+
     | 20060   |
     +---------*/

    SELECT DOT_PRODUCT([100, 10, 1], [200, 6, 2]) AS results
    
    /*---------+
     | results |
     +---------+
     | 20062   |
     +---------*/

    SELECT DOT_PRODUCT([], []) AS results
    
    /*---------+
     | results |
     +---------+
     | 0       |
     +---------*/

## `EXP`

    EXP(X)

**Description**

Computes *e* to the power of X, also called the natural exponential function. If the result underflows, this function returns a zero. Generates an error if the result overflows.

| X      | EXP(X) |
| ------ | ------ |
| 0.0    | 1.0    |
| `+inf` | `+inf` |
| `-inf` | 0.0    |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `EUCLIDEAN_DISTANCE`

    EUCLIDEAN_DISTANCE(vector1, vector2)

**Description**

Computes the [Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance) between two vectors.

**Definitions**

  - `vector1` : A vector that's represented by an `ARRAY<T>` value.
  - `vector2` : A vector that's represented by an `ARRAY<T>` value.

**Details**

  - `ARRAY<T>` can be used to represent a vector. Each zero-based index in this array represents a dimension. The value for each element in this array represents a magnitude.
    
    `T` can represent the following and must be the same for both vectors:
    
      - `FLOAT32`
      - `FLOAT64`
    
    In the following example vector, there are four dimensions. The magnitude is `10.0` for dimension `0` , `55.0` for dimension `1` , `40.0` for dimension `2` , and `34.0` for dimension `3` :
    
        [10.0, 55.0, 40.0, 34.0]

  - Both vectors in this function must share the same dimensions, and if they don't, an error is produced.

  - A vector can be a zero vector. A vector is a zero vector if it has no dimensions or all dimensions have a magnitude of `0` , such as `[]` or `[0.0, 0.0]` .

  - An error is produced if a magnitude in a vector is `NULL` .

  - If a vector is `NULL` , `NULL` is returned.

**Return type**

`FLOAT64`

**Examples**

In the following example, vectors are used to compute the Euclidean distance:

    SELECT EUCLIDEAN_DISTANCE([1.0, 2.0], [3.0, 4.0]) AS results;
    
    /*----------+
     | results  |
     +----------+
     | 2.828    |
     +----------*/

The ordering of magnitudes in a vector doesn't impact the results produced by this function. For example these queries produce the same results even though the magnitudes in each vector is in a different order:

    SELECT EUCLIDEAN_DISTANCE([1.0, 2.0], [3.0, 4.0]);

    SELECT EUCLIDEAN_DISTANCE([2.0, 1.0], [4.0, 3.0]);

``` 
 /*----------+
  | results  |
  +----------+
  | 2.828    |
  +----------*/
```

Both vectors must have the same dimensions. If not, an error is produced. In the following example, the first vector has two dimensions and the second vector has three:

    -- ERROR
    SELECT EUCLIDEAN_DISTANCE([9.0, 7.0], [8.0, 4.0, 5.0]) AS results;

## `FLOOR`

    FLOOR(X)

**Description**

Returns the largest integral value that isn't greater than X.

| X      | FLOOR(X) |
| ------ | -------- |
| 2.0    | 2.0      |
| 2.3    | 2.0      |
| 2.8    | 2.0      |
| 2.5    | 2.0      |
| \-2.3  | \-3.0    |
| \-2.8  | \-3.0    |
| \-2.5  | \-3.0    |
| 0      | 0        |
| `+inf` | `+inf`   |
| `-inf` | `-inf`   |
| `NaN`  | `NaN`    |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `GREATEST`

    GREATEST(X1,...,XN)

**Description**

Returns the greatest value among `X1,...,XN` . If any argument is `NULL` , returns `NULL` . Otherwise, in the case of floating-point arguments, if any argument is `NaN` , returns `NaN` . In all other cases, returns the value among `X1,...,XN` that has the greatest value according to the ordering used by the `ORDER BY` clause. The arguments `X1, ..., XN` must be coercible to a common supertype, and the supertype must support ordering.

| X1,...,XN | GREATEST(X1,...,XN) |
| --------- | ------------------- |
| 3,5,1     | 5                   |

**Return Data Types**

Data type of the input values.

## `IEEE_DIVIDE`

    IEEE_DIVIDE(X, Y)

**Description**

Divides X by Y; this function never fails. Returns `FLOAT64` unless both X and Y are `FLOAT32` , in which case it returns `FLOAT32` . Unlike the division operator (/), this function doesn't generate errors for division by zero or overflow.

| X      | Y      | IEEE\_DIVIDE(X, Y) |
| ------ | ------ | ------------------ |
| 20.0   | 4.0    | 5.0                |
| 20.0   | 6.0    | 3.3333333333333335 |
| 0.0    | 25.0   | 0.0                |
| 25.0   | 0.0    | `+inf`             |
| \-25.0 | 0.0    | `-inf`             |
| 25.0   | \-0.0  | `-inf`             |
| 0.0    | 0.0    | `NaN`              |
| 0.0    | `NaN`  | `NaN`              |
| `NaN`  | 0.0    | `NaN`              |
| `+inf` | `+inf` | `NaN`              |
| `-inf` | `-inf` | `NaN`              |

## `IS_INF`

    IS_INF(X)

**Description**

Returns `TRUE` if the value is positive or negative infinity.

Returns `FALSE` for `NUMERIC` inputs since `NUMERIC` can't be `INF` .

| X      | IS\_INF(X) |
| ------ | ---------- |
| `+inf` | `TRUE`     |
| `-inf` | `TRUE`     |
| 25     | `FALSE`    |

## `IS_NAN`

    IS_NAN(X)

**Description**

Returns `TRUE` if the value is a `NaN` value.

Returns `FALSE` for `NUMERIC` inputs since `NUMERIC` can't be `NaN` .

| X     | IS\_NAN(X) |
| ----- | ---------- |
| `NaN` | `TRUE`     |
| 25    | `FALSE`    |

## `LEAST`

    LEAST(X1,...,XN)

**Description**

Returns the least value among `X1,...,XN` . If any argument is `NULL` , returns `NULL` . Otherwise, in the case of floating-point arguments, if any argument is `NaN` , returns `NaN` . In all other cases, returns the value among `X1,...,XN` that has the least value according to the ordering used by the `ORDER BY` clause. The arguments `X1, ..., XN` must be coercible to a common supertype, and the supertype must support ordering.

| X1,...,XN | LEAST(X1,...,XN) |
| --------- | ---------------- |
| 3,5,1     | 1                |

**Return Data Types**

Data type of the input values.

## `LN`

    LN(X)

**Description**

Computes the natural logarithm of X. Generates an error if X is less than or equal to zero.

| X        | LN(X)  |
| -------- | ------ |
| 1.0      | 0.0    |
| `+inf`   | `+inf` |
| `X <= 0` | Error  |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `LOG`

    LOG(X [, Y])

**Description**

If only X is present, `LOG` is a synonym of `LN` . If Y is also present, `LOG` computes the logarithm of X to base Y.

| X         | Y               | LOG(X, Y) |
| --------- | --------------- | --------- |
| 100.0     | 10.0            | 2.0       |
| `-inf`    | Any value       | `NaN`     |
| Any value | `+inf`          | `NaN`     |
| `+inf`    | 0.0 \< Y \< 1.0 | `-inf`    |
| `+inf`    | Y \> 1.0        | `+inf`    |
| X \<= 0   | Any value       | Error     |
| Any value | Y \<= 0         | Error     |
| Any value | 1.0             | Error     |

**Return Data Type**

| INPUT     | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| --------- | --------- | --------- | --------- | --------- |
| `INT64`   | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `FLOAT32` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |
| `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |

## `LOG10`

    LOG10(X)

**Description**

Similar to `LOG` , but computes logarithm to base 10.

| X       | LOG10(X) |
| ------- | -------- |
| 100.0   | 2.0      |
| `-inf`  | `NaN`    |
| `+inf`  | `+inf`   |
| X \<= 0 | Error    |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `MOD`

    MOD(X, Y)

**Description**

Modulo function: returns the remainder of the division of X by Y. Returned value has the same sign as X. An error is generated if Y is 0.

| X  | Y  | MOD(X, Y) |
| -- | -- | --------- |
| 25 | 12 | 1         |
| 25 | 0  | Error     |

**Return Data Type**

The return data type is determined by the argument types with the following table.

| INPUT     | `INT64`   | `NUMERIC` |
| --------- | --------- | --------- |
| `INT64`   | `INT64`   | `NUMERIC` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` |

## `POW`

    POW(X, Y)

**Description**

Returns the value of X raised to the power of Y. If the result underflows and isn't representable, then the function returns a value of zero.

| X                         | Y                         | POW(X, Y)                                       |
| ------------------------- | ------------------------- | ----------------------------------------------- |
| 2.0                       | 3.0                       | 8.0                                             |
| 1.0                       | Any value including `NaN` | 1.0                                             |
| Any value including `NaN` | 0                         | 1.0                                             |
| \-1.0                     | `+inf`                    | 1.0                                             |
| \-1.0                     | `-inf`                    | 1.0                                             |
| ABS(X) \< 1               | `-inf`                    | `+inf`                                          |
| ABS(X) \> 1               | `-inf`                    | 0.0                                             |
| ABS(X) \< 1               | `+inf`                    | 0.0                                             |
| ABS(X) \> 1               | `+inf`                    | `+inf`                                          |
| `-inf`                    | Y \< 0                    | 0.0                                             |
| `-inf`                    | Y \> 0                    | `-inf` if Y is an odd integer, `+inf` otherwise |
| `+inf`                    | Y \< 0                    | 0                                               |
| `+inf`                    | Y \> 0                    | `+inf`                                          |
| Finite value \< 0         | Non-integer               | Error                                           |
| 0                         | Finite value \< 0         | Error                                           |

**Return Data Type**

The return data type is determined by the argument types with the following table.

| INPUT     | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| --------- | --------- | --------- | --------- | --------- |
| `INT64`   | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `FLOAT32` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |
| `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |

## `POWER`

    POWER(X, Y)

**Description**

Synonym of [`POW(X, Y)`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#pow) .

## `ROUND`

    ROUND(X [, N])

**Description**

If only X is present, rounds X to the nearest integer. If N is present, rounds X to N decimal places after the decimal point. If N is negative, rounds off digits to the left of the decimal point. Rounds halfway cases away from zero. Generates an error if overflow occurs.

| Expression         | Return Value |
| ------------------ | ------------ |
| `ROUND(2.0)`       | 2.0          |
| `ROUND(2.3)`       | 2.0          |
| `ROUND(2.8)`       | 3.0          |
| `ROUND(2.5)`       | 3.0          |
| `ROUND(-2.3)`      | \-2.0        |
| `ROUND(-2.8)`      | \-3.0        |
| `ROUND(-2.5)`      | \-3.0        |
| `ROUND(0)`         | 0            |
| `ROUND(+inf)`      | `+inf`       |
| `ROUND(-inf)`      | `-inf`       |
| `ROUND(NaN)`       | `NaN`        |
| `ROUND(123.7, -1)` | 120.0        |
| `ROUND(1.235, 2)`  | 1.24         |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `SAFE_ADD`

    SAFE_ADD(X, Y)

**Description**

Equivalent to the addition operator ( `+` ), but returns `NULL` if overflow occurs.

| X | Y | SAFE\_ADD(X, Y) |
| - | - | --------------- |
| 5 | 4 | 9               |

**Return Data Type**

| INPUT     | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| --------- | --------- | --------- | --------- | --------- |
| `INT64`   | `INT64`   | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `FLOAT32` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |
| `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |

## `SAFE_DIVIDE`

    SAFE_DIVIDE(X, Y)

**Description**

Equivalent to the division operator ( `X / Y` ), but returns `NULL` if an error occurs, such as a division by zero error.

| X  | Y  | SAFE\_DIVIDE(X, Y) |
| -- | -- | ------------------ |
| 20 | 4  | 5                  |
| 0  | 20 | `0`                |
| 20 | 0  | `NULL`             |

**Return Data Type**

| INPUT     | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| --------- | --------- | --------- | --------- | --------- |
| `INT64`   | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `FLOAT32` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |
| `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |

## `SAFE_MULTIPLY`

    SAFE_MULTIPLY(X, Y)

**Description**

Equivalent to the multiplication operator ( `*` ), but returns `NULL` if overflow occurs.

| X  | Y | SAFE\_MULTIPLY(X, Y) |
| -- | - | -------------------- |
| 20 | 4 | 80                   |

**Return Data Type**

| INPUT     | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| --------- | --------- | --------- | --------- | --------- |
| `INT64`   | `INT64`   | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `FLOAT32` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |
| `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |

## `SAFE_NEGATE`

    SAFE_NEGATE(X)

**Description**

Equivalent to the unary minus operator ( `-` ), but returns `NULL` if overflow occurs.

| X   | SAFE\_NEGATE(X) |
| --- | --------------- |
| \+1 | \-1             |
| \-1 | \+1             |
| 0   | 0               |

**Return Data Type**

| INPUT  | `INT64` | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | ------- | --------- | --------- | --------- |
| OUTPUT | `INT64` | `NUMERIC` | `FLOAT32` | `FLOAT64` |

## `SAFE_SUBTRACT`

    SAFE_SUBTRACT(X, Y)

**Description**

Returns the result of Y subtracted from X. Equivalent to the subtraction operator ( `-` ), but returns `NULL` if overflow occurs.

| X | Y | SAFE\_SUBTRACT(X, Y) |
| - | - | -------------------- |
| 5 | 4 | 1                    |

**Return Data Type**

| INPUT     | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| --------- | --------- | --------- | --------- | --------- |
| `INT64`   | `INT64`   | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `NUMERIC` | `NUMERIC` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
| `FLOAT32` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |
| `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` | `FLOAT64` |

## `SIGN`

    SIGN(X)

**Description**

Returns `-1` , `0` , or `+1` for negative, zero and positive arguments respectively. For floating point arguments, this function doesn't distinguish between positive and negative zero.

| X    | SIGN(X) |
| ---- | ------- |
| 25   | \+1     |
| 0    | 0       |
| \-25 | \-1     |
| NaN  | NaN     |

**Return Data Type**

| INPUT  | `INT64` | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | ------- | --------- | --------- | --------- |
| OUTPUT | `INT64` | `NUMERIC` | `FLOAT32` | `FLOAT64` |

## `SIN`

    SIN(X)

**Description**

Computes the sine of X where X is specified in radians. Never fails.

| X      | SIN(X) |
| ------ | ------ |
| `+inf` | `NaN`  |
| `-inf` | `NaN`  |
| `NaN`  | `NaN`  |

## `SINH`

    SINH(X)

**Description**

Computes the hyperbolic sine of X where X is specified in radians. Generates an error if overflow occurs.

If X is `NUMERIC` then, the output is `FLOAT64` .

| X      | SINH(X) |
| ------ | ------- |
| `+inf` | `+inf`  |
| `-inf` | `-inf`  |
| `NaN`  | `NaN`   |

## `SQRT`

    SQRT(X)

**Description**

Computes the square root of X. Generates an error if X is less than 0.

| X       | SQRT(X) |
| ------- | ------- |
| `25.0`  | `5.0`   |
| `+inf`  | `+inf`  |
| `X < 0` | Error   |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |

## `TAN`

    TAN(X)

**Description**

Computes the tangent of X where X is specified in radians. Generates an error if overflow occurs.

| X      | TAN(X) |
| ------ | ------ |
| `+inf` | `NaN`  |
| `-inf` | `NaN`  |
| `NaN`  | `NaN`  |

## `TANH`

    TANH(X)

**Description**

Computes the hyperbolic tangent of X where X is specified in radians. Doesn't fail.

If X is `NUMERIC` then, the output is `FLOAT64` .

| X      | TANH(X) |
| ------ | ------- |
| `+inf` | 1.0     |
| `-inf` | \-1.0   |
| `NaN`  | `NaN`   |

## `TRUNC`

    TRUNC(X [, N])

**Description**

If only X is present, `TRUNC` rounds X to the nearest integer whose absolute value isn't greater than the absolute value of X. If N is also present, `TRUNC` behaves like `ROUND(X, N)` , but always rounds towards zero and never overflows.

| X      | TRUNC(X) |
| ------ | -------- |
| 2.0    | 2.0      |
| 2.3    | 2.0      |
| 2.8    | 2.0      |
| 2.5    | 2.0      |
| \-2.3  | \-2.0    |
| \-2.8  | \-2.0    |
| \-2.5  | \-2.0    |
| 0      | 0        |
| `+inf` | `+inf`   |
| `-inf` | `-inf`   |
| `NaN`  | `NaN`    |

**Return Data Type**

| INPUT  | `INT64`   | `NUMERIC` | `FLOAT32` | `FLOAT64` |
| ------ | --------- | --------- | --------- | --------- |
| OUTPUT | `FLOAT64` | `NUMERIC` | `FLOAT64` | `FLOAT64` |
