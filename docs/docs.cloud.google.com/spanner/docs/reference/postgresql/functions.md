This page defines the functions supported for PostgreSQL-dialect databases in Spanner.

The content on this page is based on the PostgreSQL documentation, which is available under the [PostgreSQL License](https://www.postgresql.org/about/licence/) . There are differences in behavior between the PostgreSQL capabilities supported in Spanner and their open source PostgreSQL equivalents.

## Mathematical functions

Unless otherwise specified, functions return the same data type as provided in the argument.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example/Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       abs(float4 | float8 | int8 | numeric)      </code></td>
<td><code dir="ltr" translate="no">       abs(-17) → 17      </code></td>
<td>Absolute value.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       acos(float8)      </code></td>
<td><code dir="ltr" translate="no">       acos(1) → 0      </code></td>
<td>Inverse cosine, result in radians.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       asin(float8)      </code></td>
<td><code dir="ltr" translate="no">       asin(1) → 1.5707963267948966      </code></td>
<td>Inverse sine, result in radians.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       atan(float8)      </code></td>
<td><code dir="ltr" translate="no">       atan(1) → 0.7853981633974483      </code></td>
<td>Inverse tangent, result in radians.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       atan2(x float8, y float8)      </code></td>
<td><code dir="ltr" translate="no">       atan2(1,0) → 1.5707963267948966      </code></td>
<td>Inverse tangent of <code dir="ltr" translate="no">       x/y      </code> , result in radians.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ceil(float8 | numeric)      </code></td>
<td><code dir="ltr" translate="no">       ceil(42.2::FLOAT8) → 43              ceil(-42.8::FLOAT8) → -42      </code></td>
<td>Nearest integer greater than or equal to argument.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       cos(float8)      </code></td>
<td><code dir="ltr" translate="no">       cos(0) → 1      </code></td>
<td>Cosine, argument in radians.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       dexp(float8)      </code></td>
<td><code dir="ltr" translate="no">       dexp(3) → 20.085536923187668      </code></td>
<td>Raise e to the specified exponent (e^x).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       div(x numeric, y numeric)      </code></td>
<td><code dir="ltr" translate="no">       div(9, 4) → 2      </code></td>
<td>Integer quotient of x/y (truncates towards zero).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       dlog10(float8)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns the base 10 logarithm of the provided value.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       dlog1(float8)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns the value's natural logarithm.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       dpow(float8, float8)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns the value of the first number raised to the power of the second number.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       dsqrt(float8)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns the argument's square root.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       exp(float8)      </code></td>
<td><code dir="ltr" translate="no">       exp(1.0::FLOAT8) → 2.7182818284590452      </code></td>
<td>Exponential (e raised to the given power).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       floor(float8 | numeric)      </code></td>
<td><code dir="ltr" translate="no">       floor(42.8::FLOAT8) → 42              floor(-42.8) → -43      </code></td>
<td>Nearest integer less than or equal to argument.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ln(float8)      </code></td>
<td><code dir="ltr" translate="no">       ln(2.0::FLOAT8) → 0.6931471805599453      </code></td>
<td>Natural logarithm.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       log(float8)      </code></td>
<td><code dir="ltr" translate="no">       log(100.0::FLOAT8) → 2      </code></td>
<td>Base 10 logarithm.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       mod(x int8, y int8)              mod(x numeric, y numeric)      </code></td>
<td><code dir="ltr" translate="no">       mod(9,4) → 1      </code></td>
<td>Remainder of <code dir="ltr" translate="no">       x/y      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       power(float8, float8)      </code></td>
<td><code dir="ltr" translate="no">       power(9.0::FLOAT8, 3.0::FLOAT8) → 729      </code></td>
<td><code dir="ltr" translate="no">       a      </code> raised to the power of <code dir="ltr" translate="no">       b      </code> .
<p><code dir="ltr" translate="no">        pow       </code> is an alias of <code dir="ltr" translate="no">        power       </code> .</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       round(float8)      </code></td>
<td><code dir="ltr" translate="no">       round(42.4::FLOAT8) → 42      </code></td>
<td>Rounds to nearest integer.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sign(float8)      </code></td>
<td><code dir="ltr" translate="no">       sign(-8.4::FLOAT8) → -1      </code></td>
<td>Sign of the argument (-1, 0, or +1).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sin(float8)      </code></td>
<td><code dir="ltr" translate="no">       sin(1) → 0.8414709848078965      </code></td>
<td>Sine, argument in radians.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.bit_reverse(bigint_value, /*preserve_sign=*/bool)      </code></td>
<td><code dir="ltr" translate="no">       spanner.bit_reverse(1, true);              --&gt; returns 4611686018427387904              spanner.bit_reverse(10, false);              --&gt; returns  5764607523034234880      </code></td>
<td>Returns a bit-reversed value for a <code dir="ltr" translate="no">       bigint      </code> value. When <code dir="ltr" translate="no">       preserve_sign      </code> is <code dir="ltr" translate="no">       true      </code> , this function provides the same bit-reversal algorithm used in bit-reversed sequence. See <a href="/spanner/docs/primary-key-default-value#bit-reversed-sequence">Bit-reversed sequence</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.cosine_distance(float4[], float4[])              spanner.cosine_distance(float8[], float8[])      </code></td>
<td><p>Returns float8.</p>
<p><code dir="ltr" translate="no">        spanner.cosine_distance('{1.0, 2.0}'::float4[], '{3.0, 4.0}'::float4[]) → 0.016130       </code></p>
<p><code dir="ltr" translate="no">        spanner.cosine_distance('{2.0, 1.0}'::float8[], '{4.0, 3.0}'::float8[]) → 0.016130       </code></p></td>
<td><p>Computes the <a href="https://en.wikipedia.org/wiki/Cosine_similarity#Cosine_distance">cosine distance</a> between two vectors.</p>
<p>Each vector represents a quantity that includes magnitude and direction. Vectors are represented as <code dir="ltr" translate="no">        float4[]       </code> or <code dir="ltr" translate="no">        float8[]       </code> .</p>
<p>A vector can have one or more dimensions. Both vectors in this function must share these same dimensions, and if they don't, an error is produced.</p>
<p>The ordering of numeric values in a vector doesn't impact the results produced by this function.</p>
<p>An error is produced if an element or field in a vector is <code dir="ltr" translate="no">        null       </code> .</p>
<p>A vector can't be a zero vector. A vector is a zero vector if all elements in the vector are 0. For example, <code dir="ltr" translate="no">        '{0.0, 0.0}'::float4       </code> . If a zero vector is encountered, an error is produced.</p>
<p>If either of the arguments is <code dir="ltr" translate="no">        null       </code> , <code dir="ltr" translate="no">        null       </code> is returned.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.dot_product(int8[], int8[])              spanner.dot_product(float4[], float4[])              spanner.dot_product(float8[], float8[])      </code></td>
<td><p>Returns float8.</p>
<p><code dir="ltr" translate="no">        spanner.dot_product('{100}'::int8[], '{200}'::int8[]) → 20000       </code></p>
<p><code dir="ltr" translate="no">        spanner.dot_product('{100, 10}'::float4[], '{200, 6}'::float4[]) → 20060       </code></p></td>
<td><p>Computes the <a href="https://mathworld.wolfram.com/DotProduct.html">dot product</a> of two vectors. The dot product is computed by summing the product of corresponding vector elements.</p>
<p>Each vector represents a quantity that includes magnitude and direction. Vectors are represented as <code dir="ltr" translate="no">        int8[]       </code> , <code dir="ltr" translate="no">        float4[]       </code> , or <code dir="ltr" translate="no">        float8[]       </code> .</p>
<p>A vector can have one or more dimensions. Both vectors in this function must share these same dimensions, and if they don't, an error is produced.</p>
<p>The ordering of numeric values in a vector doesn't impact the results produced by this function.</p>
<p>An error is produced if an element or field in a vector is <code dir="ltr" translate="no">        null       </code> .</p>
<p>A vector can be a zero vector. A vector is a zero vector if it has no dimensions or if all elements in the vector are 0. For example, <code dir="ltr" translate="no">        '{0.0, 0.0}'::float4       </code> . If a zero vector is encountered, an error is produced.</p>
<p>If either of the arguments is <code dir="ltr" translate="no">        null       </code> , <code dir="ltr" translate="no">        null       </code> is returned.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.euclidean_distance(float4[], float4[])              spanner.euclidean_distance(float8[], float8[])      </code></td>
<td><p>Returns float8.</p>
<p><code dir="ltr" translate="no">        spanner.euclidean_distance('{1.0, 2.0}'::float4[], '{3.0, 4.0}'::float4[]) → 2.828       </code></p>
<p><code dir="ltr" translate="no">        spanner.euclidean_distance('{2.0, 1.0}'::float8[], '{4.0, 3.0}'::float8[]) → 2.828       </code></p></td>
<td><p>Computes the <a href="https://en.wikipedia.org/wiki/Euclidean_distance">Euclidean distance</a> between two vectors.</p>
<p>Each vector represents a quantity that includes magnitude and direction. Vectors are represented as <code dir="ltr" translate="no">        float4[]       </code> or <code dir="ltr" translate="no">        float8[]       </code> .</p>
<p>A vector can have one or more dimensions. Both vectors in this function must share these same dimensions, and if they don't, an error is produced.</p>
<p>The ordering of numeric values in a vector doesn't impact the results produced by this function.</p>
<p>An error is produced if an element or field in a vector is <code dir="ltr" translate="no">        null       </code> .</p>
<p>A vector can be a zero vector. A vector is a zero vector if all elements in the vector are 0. For example, <code dir="ltr" translate="no">        '{0.0, 0.0}'::float4       </code> .</p>
<p>If either of the arguments is <code dir="ltr" translate="no">        null       </code> , <code dir="ltr" translate="no">        null       </code> is returned.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sqrt(float8)      </code></td>
<td><code dir="ltr" translate="no">       sqrt(2::FLOAT8) → 1.4142135623730951      </code></td>
<td>Square root.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tan(float8)      </code></td>
<td><code dir="ltr" translate="no">       tan(1) → 1.5574077246549023      </code></td>
<td>Tangent, argument in radians.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       trunc(float8)      </code></td>
<td><code dir="ltr" translate="no">       trunc(42.8::FLOAT8) → 42              trunc(-42.8::FLOAT8) → -42      </code></td>
<td>Truncates to integer (towards zero).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       trunc(x numeric, y integer)      </code></td>
<td><code dir="ltr" translate="no">       trunc(42.4382, 2) → 42.43      </code></td>
<td>Truncates x to y decimal places.</td>
</tr>
</tbody>
</table>

## Machine learning functions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.ML_PREDICT_ROW(model_endpoint               text|jsonb              , args               jsonb              )      </code></td>
<td>Returns JSONB.<br />
<code dir="ltr" translate="no"></code></td>
<td><code dir="ltr" translate="no">       spanner.ML_PREDICT_ROW      </code> is a scalar function that allows predictions on a per row basis and can appear anywhere a scalar expression is allowed in SQL statements. You can get online predictions in your SQL code by calling this function. For more information about this function, see <a href="/spanner/docs/ml-tutorial#use_ml_predict_for_ml_serving">Use ML Predict for ML serving</a> .</td>
</tr>
</tbody>
</table>

## Array functions and comparisons

**PostgreSQL interface note:** The PostgreSQL interface does not support multi-dimensional arrays.

### Array functions

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 50%" />
<col style="width: 30%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       array_cat(anyarray, anyarray)      </code></td>
<td><code dir="ltr" translate="no">       array_cat(ARRAY['cat', 'dog'], ARRAY['bird', 'turtle']) → {"cat", "dog", "bird", "turtle"}      </code></td>
<td>Concatenates two arrays.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       array_to_string (               array              text_array,               delimiter              text [,               null_string              text ] )      </code></td>
<td><code dir="ltr" translate="no">       array_to_string(ARRAY['a', NULL, 'c', 'd', 'e'], ',', '*')              → a,*,c,d,e               array_to_string(ARRAY['a', NULL, 'c', 'd', 'e'], ',')              → a,c,d,e               array_to_string(ARRAY['a', NULL, 'c', 'd', 'e'], ',', NULL)              → NULL               array_to_string(ARRAY['a', NULL, 'c', 'd', 'e'], NULL, '*')              → NULL      </code></td>
<td>Converts the values of the elements in a text array to their string representations. The first argument is the <em>array</em> which must be a text array. The second argument is a user-specified <em>delimiter</em> . The third (optional) argument is a user-specified <em>null_string</em> that the function substitutes for NULL values.<br />
<br />
If you don't pass a <em>null_string</em> , and the function encounters a NULL value, the NULL value is not included in the results of the function.<br />
<br />
If you pass NULL for either the <em>delimiter</em> argument or the <em>null_string</em> argument, then the entire array_to_string function returns <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       array_upper(anyarray,               dimension              int8)      </code></td>
<td><code dir="ltr" translate="no">       array_upper(ARRAY[1, 2, 3, 4], 1) → 4      </code></td>
<td>Returns the upper bound of the requested array dimension. Note that Spanner does not support multidimensional arrays. The only dimension supported is <code dir="ltr" translate="no">       1      </code> . For more information, see <a href="/spanner/docs/reference/postgresql/arrays">Working with arrays in PostgreSQL-dialect databases</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       array_length(anyarray,               dimension              int8)      </code></td>
<td><code dir="ltr" translate="no">       array_length(ARRAY[1, 2, 3, 4], 1) → 4      </code></td>
<td>Returns the size of the array. Returns <code dir="ltr" translate="no">       NULL      </code> for an empty or <code dir="ltr" translate="no">       NULL      </code> array, or if the dimension is <code dir="ltr" translate="no">       NULL      </code> . multidimensional arrays are not supported. The only dimension supported is <code dir="ltr" translate="no">       1      </code> . For more information, see <a href="/spanner/docs/reference/postgresql/arrays">Working with arrays in PostgreSQL-dialect databases</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       array(subquery)      </code></td>
<td></td>
<td>Returns an <code dir="ltr" translate="no">       ARRAY      </code> with one element for each row in the subquery. For more information, see <a href="/spanner/docs/reference/postgresql/arrays">Working with arrays in PostgreSQL-dialect databases</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       unnest(anyarray)      </code></td>
<td></td>
<td>Expands an <code dir="ltr" translate="no">       ARRAY      </code> into a set of rows. For more information, see <a href="/spanner/docs/reference/postgresql/arrays">Working with arrays in PostgreSQL-dialect databases</a> .</td>
</tr>
</tbody>
</table>

  - For details about the array aggregate function, see [aggregate functions](/spanner/docs/reference/postgresql/functions#aggregate) .

### Array comparisons

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Comparison syntax</th>
<th>Example</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       expression = ANY (anyarray)      </code><br />
<br />
<code dir="ltr" translate="no">       expression = SOME (anyarray)      </code></td>
<td><code dir="ltr" translate="no">       2 = ANY(array[1, 2]) → true      </code></td>
<td><code dir="ltr" translate="no">       ANY      </code> / <code dir="ltr" translate="no">       SOME      </code> array comparison construct. Returns <code dir="ltr" translate="no">       true      </code> if the evaluated value of the expression on the left is equal to any of the array elements. There are no differences between <code dir="ltr" translate="no">       ANY      </code> and <code dir="ltr" translate="no">       SOME      </code> .<br />
<br />
<code dir="ltr" translate="no">       ANY      </code> / <code dir="ltr" translate="no">       SOME      </code> only supports the <code dir="ltr" translate="no">       =      </code> operator.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       expression operator ALL (anyarray)      </code></td>
<td><code dir="ltr" translate="no">       (1+2) &gt;= ALL(array[1, 2, 3]) → true      </code></td>
<td><code dir="ltr" translate="no">       ALL      </code> array comparison construct. Returns <code dir="ltr" translate="no">       true      </code> if the expression on the left evaluates to <code dir="ltr" translate="no">       true      </code> when compared against all elements of the array with the specified operator.<br />
<br />
<code dir="ltr" translate="no">       =      </code> , <code dir="ltr" translate="no">       &lt;&gt;      </code> , <code dir="ltr" translate="no">       &gt;      </code> , <code dir="ltr" translate="no">       &gt;=      </code> , <code dir="ltr" translate="no">       &lt;      </code> , and <code dir="ltr" translate="no">       &lt;=      </code> operators are supported with <code dir="ltr" translate="no">       ALL      </code> .</td>
</tr>
</tbody>
</table>

## String functions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       btrim(text)      </code></td>
<td><code dir="ltr" translate="no">       btrim('  xyxyyx    ') → xyxyyx      </code></td>
<td>Removes leading and trailing whitespace from the given string.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       btrim(               string              text,               characters              text)      </code></td>
<td><code dir="ltr" translate="no">       btrim('xyxtrimyyx', 'xyz') → trim      </code></td>
<td>Removes the longest string containing only characters in <code dir="ltr" translate="no">         characters       </code> from the start and end of <code dir="ltr" translate="no">         string       </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       concat(text, text[, ...])      </code></td>
<td><p><code dir="ltr" translate="no">        concat('abcde', 2::text, '', 22::text) → abcde222       </code><br />
</p>
<p><code dir="ltr" translate="no">        concat('abcde', 2::text, NULL, 22::text) → NULL       </code></p></td>
<td>Concatenates the provided text arguments. Non-text arguments must first be explicitly cast to <code dir="ltr" translate="no">       text      </code> . Any SQL <code dir="ltr" translate="no">       NULL      </code> argument results in a SQL <code dir="ltr" translate="no">       NULL      </code> result.
<p><code dir="ltr" translate="no">        textcat       </code> also concatenates text.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       length(text)      </code></td>
<td>Returns int8.<br />
<code dir="ltr" translate="no">       length('mike') → 4      </code></td>
<td>Returns the number of characters in the string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       lower(text)      </code></td>
<td><code dir="ltr" translate="no">       lower('PostgreSQL') → postgresql      </code></td>
<td>Converts the string to all lower case.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       lpad(text, int8)      </code></td>
<td><code dir="ltr" translate="no">       lpad('hi', 7) → ␣␣␣␣␣hi      </code></td>
<td>Extends the string to the specified length by prepending spaces. If the string is already longer than length then it is truncated on the right.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       lpad(               string              text,               length              int8,               fill              text)      </code></td>
<td><code dir="ltr" translate="no">       lpad('hi', 7, 'xy') → xyxyxhi      </code></td>
<td>Extends the string to length <code dir="ltr" translate="no">         length       </code> by prepending the characters <code dir="ltr" translate="no">         fill       </code> , repeated. If the string is already longer than <code dir="ltr" translate="no">         length       </code> then it is truncated on the right.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ltrim(text)      </code></td>
<td><code dir="ltr" translate="no">       ltrim('     test') → test      </code></td>
<td>Removes leading spaces from a string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ltrim(               string              text,               characters              text)      </code></td>
<td><code dir="ltr" translate="no">       ltrim('zzzytest', 'xyz') → test      </code></td>
<td>Removes the longest string containing only characters in <code dir="ltr" translate="no">         characters       </code> from the start of <code dir="ltr" translate="no">         string       </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       quote_ident(               string              text)      </code></td>
<td><code dir="ltr" translate="no">       quote_ident('Example') → "Example"      </code></td>
<td>Given a string argument, returns a quoted identifier suitable for inclusion in SQL statements.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       regexp_replace(               string              text,               pattern              text,               replacement              text)      </code></td>
<td><code dir="ltr" translate="no">       regexp_replace('Thomas', '.[mN]a.', 'M') → ThM      </code></td>
<td>Replaces substrings resulting from the first match of a POSIX regular expression. For more information, see the open source PostgreSQL <a href="https://www.postgresql.org/docs/13/functions-matching.html#FUNCTIONS-POSIX-REGEXP">POSIX Regular Expressions documentation</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       repeat(text, int8)      </code></td>
<td>Returns text.<br />
<code dir="ltr" translate="no">       repeat('Pg', 4) → PgPgPgPg      </code></td>
<td>Repeats a string the specified number of times.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       replace(               string              text,               from              text,               to              text)      </code></td>
<td><code dir="ltr" translate="no">       replace('abcdefabcdef', 'cd', 'XX') → abXXefabXXef      </code></td>
<td>Replaces all occurrences in <code dir="ltr" translate="no">         string       </code> of substring <code dir="ltr" translate="no">         from       </code> with substring <code dir="ltr" translate="no">         to       </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       reverse(text)      </code></td>
<td><code dir="ltr" translate="no">       reverse('abcde') → edcba      </code></td>
<td>Reverses the order of the characters in the string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rpad(text, int8)      </code></td>
<td>Returns text. In the following example, the result includes three trailing spaces.<br />
<code dir="ltr" translate="no">       rpad('hi', 5) → hi␣␣␣      </code></td>
<td>Extends the string to the specified length by appending spaces. If the string is already longer than the specified length then it is truncated.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rpad(               string              text,               length              int8,               fill              text)      </code></td>
<td>Returns text.<br />
<code dir="ltr" translate="no">       rpad('hi', 5, 'xy') → hixyx      </code></td>
<td>Extends the <code dir="ltr" translate="no">         string       </code> to length <code dir="ltr" translate="no">         length       </code> by appending the characters <code dir="ltr" translate="no">         fill       </code> , repeated if necessary. If the string is already longer than <code dir="ltr" translate="no">         length       </code> then it is truncated.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rtrim(text)      </code></td>
<td><code dir="ltr" translate="no">       rtrim('test    ') → test      </code></td>
<td>Removes trailing spaces from a string.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rtrim(               string              text,               characters              text)      </code></td>
<td><code dir="ltr" translate="no">       rtrim('testxxzx', 'xyz') → test      </code></td>
<td>Removes the longest string containing only characters in <code dir="ltr" translate="no">         characters       </code> from the end of the <code dir="ltr" translate="no">         string       </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.soundex(text)      </code></td>
<td><code dir="ltr" translate="no">       spanner.soundex('Ashcraft') -&gt; A261      </code></td>
<td>Returns a <code dir="ltr" translate="no">       STRING      </code> that represents the <a href="https://en.wikipedia.org/wiki/Soundex">Soundex</a> code for value.
<p>Soundex produces a phonetic representation of a string. It indexes words by sound, as pronounced in English. It's typically used to help determine whether two strings have similar English-language pronunciations, such as the family names <em>Levine</em> and <em>Lavine</em> , or the words <em>to</em> and <em>too</em> ,</p>
<p>The result of the Soundex consists of a letter followed by 3 digits. Non-latin characters are ignored. If the remaining string is empty after removing non-Latin characters, an empty <code dir="ltr" translate="no">          string        </code> is returned.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.split_substr(               value              text,               delimiter              text,               start_split              int8[,               count              int8])      </code></td>
<td><p><code dir="ltr" translate="no">        spanner.split_substr('www.abc.xyz.com', '.', 1, 0) → ''       </code></p>
<p><code dir="ltr" translate="no">        spanner.split_substr('www.abc.xyz.com', '.', 1, 2) → www.abc       </code></p>
<p><code dir="ltr" translate="no">        spanner.split_substr('www.abc.xyz.com', '.', 1, 1) → www       </code></p>
<p><code dir="ltr" translate="no">        spanner.split_substr('www.abc.xyz.com', '.', -1, 1) → com       </code></p>
<p><code dir="ltr" translate="no">        spanner.split_substr('www.abc.xyz.com', '.', 2, 2) → abc.xyz       </code></p></td>
<td><p>Returns substrings from <code dir="ltr" translate="no">          value        </code> that is determined by the <code dir="ltr" translate="no">          delimiter        </code> , with <code dir="ltr" translate="no">          start_split        </code> indicating the first split of the substring and <code dir="ltr" translate="no">          count        </code> indicating the number of splits to be returned.</p>
<p><code dir="ltr" translate="no">          value        </code> is the supplied <code dir="ltr" translate="no">        text       </code> value from which a substring is returned.</p>
<p><code dir="ltr" translate="no">          delimiter        </code> must be a literal character or a sequence of characters that is matched from left to right against <code dir="ltr" translate="no">          value        </code> . It can't be a regular expression. If the <code dir="ltr" translate="no">          delimiter        </code> is a sequence of characters, then two instances of the delimiter in <code dir="ltr" translate="no">          value        </code> can't overlap.</p>
<p><code dir="ltr" translate="no">          start_split        </code> is an integer that specifies the first split of the substring to return. If <code dir="ltr" translate="no">          start_split        </code> is <code dir="ltr" translate="no">        1       </code> , <code dir="ltr" translate="no">        0       </code> , or less than the negative of the split count, then the function returns a substring that starts with the first split. If the value is greater than the number of splits, the function returns an empty string. If the value is negative, then the splits are counted from the end of the input string.</p>
<p><code dir="ltr" translate="no">          count        </code> is an integer that can be optionally specified to determine the maximum number of splits to include in the returned substring. If <code dir="ltr" translate="no">          count        </code> is unspecified or if the sum of <code dir="ltr" translate="no">          count        </code> and <code dir="ltr" translate="no">          start_split        </code> is greater than the split count, the function returns the substring from the <code dir="ltr" translate="no">          start_split        </code> position to the end of <code dir="ltr" translate="no">          value        </code> . If <code dir="ltr" translate="no">          count        </code> is <code dir="ltr" translate="no">        0       </code> , the function returns an empty string. If <code dir="ltr" translate="no">          count        </code> is negative, the function returns an error.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       starts_with(               string              text,               prefix              text)      </code></td>
<td>Returns Boolean.<br />
<code dir="ltr" translate="no">       starts_with('alphabet', 'alph') → true      </code></td>
<td>Returns <code dir="ltr" translate="no">       true      </code> if <code dir="ltr" translate="no">         string       </code> starts with <code dir="ltr" translate="no">         prefix       </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       strpos(               string              text,               substring              text)      </code></td>
<td>Returns int8.<br />
<code dir="ltr" translate="no">       strpos('high', 'ig') → 2      </code></td>
<td>Returns first starting index of the specified <code dir="ltr" translate="no">         substring       </code> within <code dir="ltr" translate="no">         string       </code> , or zero if it's not present.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       substr(               string              text,               start              int8)      </code></td>
<td><code dir="ltr" translate="no">       substr('alphabet', 3) → phabet      </code></td>
<td>Extracts the substring of the provided text starting at the specified character.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       substr(               string              text,               start              int8,               count              int8)      </code></td>
<td><code dir="ltr" translate="no">       substr('alphabet', 3, 2) → ph      </code></td>
<td>Extracts the substring of <code dir="ltr" translate="no">         string       </code> starting at the <code dir="ltr" translate="no">         start       </code> character, and extending for <code dir="ltr" translate="no">         count       </code> characters.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       substring(               string              text,               pattern              text)      </code></td>
<td><code dir="ltr" translate="no">       substring('exampletext', 'tex.') → 'text'      </code></td>
<td>Extracts the substring that matches a POSIX regular expression. For more information, see the open source PostgreSQL <a href="https://www.postgresql.org/docs/13/functions-matching.html#FUNCTIONS-POSIX-REGEXP">POSIX Regular Expressions documentation</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       textcat(               string              text,               string              text)      </code></td>
<td><p><code dir="ltr" translate="no">        textcat('abcde', '222') → abcde222       </code><br />
</p>
<p><code dir="ltr" translate="no">        textcat('abcde', NULL) → NULL       </code><br />
</p></td>
<td>Concatenates the text representations of the two arguments. Any SQL <code dir="ltr" translate="no">       NULL      </code> argument results in a SQL <code dir="ltr" translate="no">       NULL      </code> result.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       upper(               string              text)      </code></td>
<td><code dir="ltr" translate="no">       upper('hello') → HELLO      </code></td>
<td>Converts the string to all upper case.</td>
</tr>
</tbody>
</table>

## Binary string functions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       btrim(               bytes              bytea,               bytesremoved              bytea)      </code></td>
<td><code dir="ltr" translate="no">       btrim('\x1234567890'::bytea, '\x9012'::bytea) → \x345678      </code></td>
<td>Removes the longest string containing only bytes appearing in <em>bytesremoved</em> from the start and end of <em>bytes</em> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       length(bytea)      </code></td>
<td>Returns int8.<br />
<code dir="ltr" translate="no">       length('\x1234567890'::bytea) → 5      </code></td>
<td>Returns the number of bytes in the binary string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sha256(bytea)      </code></td>
<td><code dir="ltr" translate="no">       sha256('abc'::bytea) → ungWv48Bz+pBQUDeXa4iI7ADYaOWF3qctBD/YfIAFa0=      </code></td>
<td>Computes the SHA-256 hash of the binary string.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sha512(bytea)      </code></td>
<td><code dir="ltr" translate="no">       sha512('abc'::bytea) → 3a81oZNherrMQXNJriBBMRLm+k6JqX6iCp7u5ktV05ohkpkqJ0/BqDa6PCOj/uu9RU1EI2Q86A4qmslPpUyknw==      </code></td>
<td>Computes the SHA-512 hash of the binary string.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       substr(               bytes              bytea,               start              int8)      </code></td>
<td><code dir="ltr" translate="no">       substr('\x1234567890'::bytea, 3) → \x567890      </code></td>
<td>Extracts the substring of <em>bytes</em> starting at the <em>start</em> byte.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       substr(               bytes              bytea,               start              int8,               count              int8)      </code></td>
<td><code dir="ltr" translate="no">       substr('\x1234567890'::bytea, 3, 2) → \x5678      </code></td>
<td>Extracts the substring of <em>bytes</em> starting at the <em>start</em> byte, and extending for <em>count</em> bytes.</td>
</tr>
</tbody>
</table>

## Hash functions

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.farm_fingerprint(               value              bytea | text)      </code></td>
<td><code dir="ltr" translate="no">       spanner.farm_fingerprint('abc') → 2640714258260161385      </code></td>
<td>Computes the fingerprint of <em>value</em> using the FarmHash Fingerprint64 algorithm.</td>
</tr>
</tbody>
</table>

## Date and time functions

This section describes the date and time functions that are available in Spanner.

### Date and time functions

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       current_date      </code></td>
<td><code dir="ltr" translate="no">       SELECT CURRENT_DATE;              Result: 2022-05-13      </code></td>
<td>Returns current <code dir="ltr" translate="no">       date      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       current_timestamp      </code></td>
<td><code dir="ltr" translate="no">       SELECT CURRENT_TIMESTAMP;              Result: 2022-05-13T16:30:29.880850967Z      </code></td>
<td>Returns current date and time in <code dir="ltr" translate="no">       timestamptz      </code> format.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       date_trunc(text, timestamptz)      </code></td>
<td><code dir="ltr" translate="no">       date_trunc('day', timestamptz '2020-01-02 13:14:15+0') -&gt; 2020-01-02 00:00:00-08      </code></td>
<td>Truncates a timestamp to the precision of the provided field. The truncation is done with respect to the default time zone (America/Los_Angeles)</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       date_trunc(text, timestamptz, text)      </code></td>
<td><code dir="ltr" translate="no">       date_trunc('day', timestamptz '2001-02-16 20:38:40+00', 'Australia/Sydney') -&gt; 2001-02-16 08:00:00-05      </code></td>
<td>Truncates a timestamp to the precision of the provided field. The trunctation is done with respect to the provided time zone</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       extract(field FROM source)      </code></td>
<td><code dir="ltr" translate="no">       extract(decade from timestamptz '2001-01-01 01:00:00+00') -&gt; 200      </code></td>
<td>Retrieves subfields from date and time values and returns values of type numeric. Source can use the date or timestamptz data type.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       make_date(int8, int8, int8)      </code></td>
<td><code dir="ltr" translate="no">       make_date(2013, 7, 15) → 2013-07-15      </code></td>
<td>Creates date from year, month, and day fields (negative years signify BCE).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       now()      </code></td>
<td><code dir="ltr" translate="no">       now() → 2022-05-02T19:17:45.145511221Z      </code></td>
<td>Returns current date and time in <code dir="ltr" translate="no">       timestamptz      </code> format.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_timestamp(int8)      </code></td>
<td><code dir="ltr" translate="no">       to_timestamp(1284352323) → 2010-09-13T04:32:03Z      </code></td>
<td>Converts Unix epoch (seconds since 1970-01-01 00:00:00+00) to <code dir="ltr" translate="no">       timestamptz      </code> format.</td>
</tr>
</tbody>
</table>

### Spanner specific date and time functions

Spanner has several functions that perform date or time math that accept `  INTERVAL  ` values in `  TEXT  ` form. You must use the `  spanner  ` namespace to call these functions.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.date(timestamptz, text)      </code></td>
<td><code dir="ltr" translate="no">       spanner.date('2025-04-14 03:38:40+00'::timestamptz, 'America/New_York') -&gt; 2025-04-13      </code></td>
<td><p>Extracts date from a timestamptz in a specified time zone.</p>
<p>If a time zone value is not provided in the first parameter, the time zone value defaults to <a href="/spanner/docs/reference/standard-sql/data-types#time_zones">America/Los_Angeles</a> . For example, <code dir="ltr" translate="no">        spanner.date('2025-04-14 23:38:40'::timestamptz, 'America/New_York') -&gt; 2025-04-15       </code></p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.date_bin(text, timestamptz, timestamptz)      </code></td>
<td><code dir="ltr" translate="no">       spanner.date_bin('15 minutes', timestamptz '2001-02-16 20:38:40Z', timestamptz '2001-02-16 20:05:00Z') -&gt; 2001-02-16 20:35:00Z      </code></td>
<td>Bins input into a specified interval aligned with a specified origin.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.timestamptz_add(timestamptz, text)      </code></td>
<td><code dir="ltr" translate="no">       spanner.timestamptz_add(timestamptz '2001-02-16 20:38:40Z', '1 day 3min') -&gt; 2001-02-17 20:41:40Z      </code></td>
<td>Adds an interval to a <code dir="ltr" translate="no">       timestamptz      </code> . To be more consistent with the PostgreSQL language, we recommend using the <a href="/spanner/docs/reference/postgresql/data-types#interval-type"><code dir="ltr" translate="no">        INTERVAL       </code></a> type with the addition operator ( <code dir="ltr" translate="no">       +      </code> ) instead.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.timestamptz_subtract(timestamptz, text)      </code></td>
<td><code dir="ltr" translate="no">       spanner.timestamptz_subtract(timestamptz '2001-02-16 20:38:40Z', '1 month 2 hours') -&gt; 2001-01-16 18:38:40Z      </code></td>
<td>Subtracts an interval from a <code dir="ltr" translate="no">       timestamptz      </code> . To be more consistent with the PostgreSQL language, we recommend using the <a href="/spanner/docs/reference/postgresql/data-types#interval-type"><code dir="ltr" translate="no">        INTERVAL       </code></a> type with the subtraction operator ( <code dir="ltr" translate="no">       -      </code> ) instead.</td>
</tr>
</tbody>
</table>

## Search functions

Spanner has several functions that perform full-text search operations. For more information, see [Full-text search](/spanner/docs/full-text-search) . For more information on search functions, see the GoogleSQL [Search functions](/spanner/docs/reference/standard-sql/search_functions) section.

### Indexing

Functions that you can use to create search indexes.

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 20%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.token(value text/bytea)      </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code></td>
<td>Constructs an exact match <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing a text value verbatim to accelerate exact match expressions.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.tokenize_bool(              value bool)       </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Constructs a boolean <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing a <code dir="ltr" translate="no">       BOOL      </code> value to accelerate boolean match expressions.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.tokenize_fulltext(              value text/text[]              [, language_tag text]              [, content_type text]              [, token_category text])       </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Constructs a full-text <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing text for full-text matching.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.tokenize_jsonb(              value jsonb)       </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Constructs a JSON <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing a <code dir="ltr" translate="no">       JSONB      </code> value to accelerate JSON predicates.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.tokenize_ngrams(              value text              [, ngram_size_min int8]              [, ngram_size_max int8]              [, remove_diacritics bool])       </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Constructs an n-gram <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing a text value for matching n-grams.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.tokenize_number(              value int8              [, comparison_type text]              [, algorithm text]              [, min int8]              [, max int8]              [, granularity int8]              [, tree_base int8]              [, ieee_precision int8])       </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Constructs a numeric <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing numeric values to accelerate numeric comparison expressions.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.tokenize_substring(              value text/text[]              [, language_tag text]              [, ngram_size_min int8]              [, ngram_size_max int8]              [, relative_search_types text[]]              [, content_type text]              [, short_tokens_only_for_anchors bool]              [, remove_diacritics bool])       </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Constructs a substring <code dir="ltr" translate="no">       tokenlist      </code> value by tokenizing text for substring matching.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.tokenlist_concat(tokens spanner.tokenlist[])      </code></td>
<td>Returns a<br />
<code dir="ltr" translate="no">       spanner.tokenlist      </code> .</td>
<td>Displays a human-readable representation of tokens present in a <code dir="ltr" translate="no">       tokenlist      </code> value for debugging purposes.</td>
</tr>
</tbody>
</table>

### Retrieval and presentation

Functions that you can use to search for data, score the search result, or format the search result.

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 20%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.score(              tokens spanner.tokenlist,              query text              [, dialect text]              [, language_tag text]              [, enhance_query bool]              [, options jsonb])       </code></td>
<td>Returns a <code dir="ltr" translate="no">       float8      </code> .</td>
<td>Calculates a relevance score of a <code dir="ltr" translate="no">       tokenlist      </code> for a full-text search query. The higher the score, the stronger the match.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.score_ngrams(              tokens spanner.tokenlist,              ngrams_query text              [, language_tag text]              [, algorithm text])       </code></td>
<td></td>
<td>Calculates the relevance score of a <code dir="ltr" translate="no">       tokenlist      </code> for a fuzzy search. The higher the score, the stronger the match.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.search(              tokens spanner.tokenlist,              query text              [, dialect text]              [, language_tag text]              [, enhance_query bool])       </code></td>
<td>Returns a <code dir="ltr" translate="no">       bool      </code> .</td>
<td>Returns <code dir="ltr" translate="no">       TRUE      </code> if a full-text search query matches tokens.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.search_ngrams(              tokens spanner.tokenlist,              ngrams_query text              [, language_tag text]              [, min_ngrams int8]              [, min_ngrams_percent float8])       </code></td>
<td>Returns a <code dir="ltr" translate="no">       bool      </code> .</td>
<td>Checks whether enough n-grams match the tokens in a fuzzy search.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.search_substring(              tokens spanner.tokenlist              [, query text]              [, language_tag text]              [, relative_search_type text])       </code></td>
<td>Returns a <code dir="ltr" translate="no">       bool      </code> .</td>
<td>Returns <code dir="ltr" translate="no">       TRUE      </code> if a substring query matches tokens.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.snippet(              value text,              query text              [, language_tag text]              [, enhance_query bool]              [, max_snippet_width int8]              [, max_snippets int8]              [, content_type text])       </code></td>
<td>Returns <code dir="ltr" translate="no">       jsonb      </code> .</td>
<td>Gets a list of snippets that match a full-text search query.</td>
</tr>
</tbody>
</table>

### Debugging

Functions that you can use for debugging.

<table>
<colgroup>
<col style="width: 40%" />
<col style="width: 20%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.debug_tokenlist(              spanner.tokenlist)       </code></td>
<td>Returns <code dir="ltr" translate="no">       text      </code> .</td>
<td>Displays a human-readable representation of tokens present in the <code dir="ltr" translate="no">       tokenlist      </code> value for debugging purposes.</td>
</tr>
</tbody>
</table>

## JSONB functions

Spanner supports several `  JSONB  ` functions.

For more information, see the [PostgreSQL `  JSONB  ` documentation](https://www.postgresql.org/docs/current/functions-json.html) .

### JSONB functions

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 50%" />
<col style="width: 30%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb_array_elements(JSONB)      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_array_elements('[1, "abc", {"k": "v"}]'::jsonb)       </code></p>
<pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>      /*---------------------*
      | jsonb_array_elements |
      +----------------------+
      | &#39;1&#39;                  |
      | &#39;&quot;abc&quot;&#39;              |
      | &#39;{&quot;k&quot;: &quot;v&quot;}&#39;         |
      *---------------------*/
      </code></pre></td>
<td><p>Expands a <code dir="ltr" translate="no">        jsonb       </code> array to a set of <code dir="ltr" translate="no">        jsonb       </code> values. Returns multiple rows, with one element per row. Unlike open source PostgreSQL, this can only be called as a table valued function in the <code dir="ltr" translate="no">        FROM       </code> clause.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb_build_array(ANY[, ...])      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_build_array(1, 'abc') → [1, "abc"]       </code></p></td>
<td><p>Builds a <code dir="ltr" translate="no">        jsonb       </code> array out of a variadic argument list.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb_build_object(TEXT, ANY[, ...])      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_build_object('key1', 1, 'key2', 'abc') → {"key": 1, "key2": "abc"}       </code></p></td>
<td><p>Builds a <code dir="ltr" translate="no">        jsonb       </code> object out of a variadic argument list. The argument list consists of alternating keys and values. The keys are of type <code dir="ltr" translate="no">        text       </code> .</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb_insert(target jsonb, path text[], new_value jsonb [, insert_after bool]) → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_insert('{"a": [0,1,2]}', '{a, 1}', '"new_value"') → {"a": [0, "new_value", 1, 2]}       </code></p>
<p><code dir="ltr" translate="no">        jsonb_insert('{"a": [0,1,2]}', '{a, 1}', '"new_value"', true) → {"a": [0, 1, "new_value", 2]}       </code></p></td>
<td>Returns <code dir="ltr" translate="no">       target      </code> with <code dir="ltr" translate="no">       new_value      </code> inserted as specified by <code dir="ltr" translate="no">       path      </code> . If the item designated by the path is an array element, <code dir="ltr" translate="no">       new_value      </code> is inserted before that item if <code dir="ltr" translate="no">       insert_after      </code> is <code dir="ltr" translate="no">       false      </code> (which is the default behavior), or after it if <code dir="ltr" translate="no">       insert_after      </code> is <code dir="ltr" translate="no">       true      </code> . If the item designated by the path is an object field, <code dir="ltr" translate="no">       new_value      </code> is inserted only if the object does not already contain that key. All earlier steps in the path must exist, or the <code dir="ltr" translate="no">       target      </code> is returned unchanged. As with the path-oriented operators, negative integers that appear in the path count from the end of JSON arrays. If the last path step is an array index that is out of range, the new value is added at the beginning of the array if the index is negative or at the end of the array if it is positive.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb_set(target jsonb, path text[], new_value jsonb [, create_if_missing bool]) → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_set('[{"f1":1,"f2":null},2,null,3]', '{0,f1}', '[2,3,4]', false) → [{"f1": [2, 3, 4], "f2": null}, 2, null, 3]       </code></p>
<p><code dir="ltr" translate="no">        jsonb_set('[{"f1":1,"f2":null},2]', '{0,f3}', '[2,3,4]') → [{"f1": 1, "f2": null, "f3": [2, 3, 4]}, 2]       </code></p></td>
<td>Returns <code dir="ltr" translate="no">       target      </code> with the item designated by <code dir="ltr" translate="no">       path      </code> replaced by <code dir="ltr" translate="no">       new_value      </code> , or with <code dir="ltr" translate="no">       new_value      </code> added if <code dir="ltr" translate="no">       create_if_missing      </code> is <code dir="ltr" translate="no">       true      </code> (which is the default behavior) and the item designated by the path does not exist. All earlier steps in the path must exist, or the <code dir="ltr" translate="no">       target      </code> is returns unchanged. As with the path-oriented operators, negative integers that appear in the path count from the end of JSON arrays. If the last path step is an array index that is out of range. If <code dir="ltr" translate="no">       create_if_missing      </code> is <code dir="ltr" translate="no">       true      </code> and the last path step is an out-of-range array index, the new value is added to the beginning of the array (if the index is negative) or the end of the array (if the index is positive).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb_set_lax(target jsonb, path text[], new_value jsonb [, create_if_missing bool [, null_value_treatment text]]) → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_set_lax('[{"f1":1,"f2":null},2,null,3]', '{0,f1}', null) → [{"f1": null, "f2": null}, 2, null, 3]       </code></p>
<p><code dir="ltr" translate="no">        jsonb_set_lax('[{"f1":99,"f2":null},2]', '{0,f3}', null, true, 'return_target') → [{"f1": 99, "f2": null}, 2]       </code></p></td>
<td><p>If <code dir="ltr" translate="no">        new_value       </code> is not <code dir="ltr" translate="no">        NULL       </code> , this function behaves identically to <code dir="ltr" translate="no">        jsonb_set       </code> . Otherwise, this function behaves according to the value of <code dir="ltr" translate="no">        null_value_treatment       </code> which must be one of <code dir="ltr" translate="no">        raise_exception       </code> , <code dir="ltr" translate="no">        use_json_null       </code> , <code dir="ltr" translate="no">        delete_key       </code> , or <code dir="ltr" translate="no">        return_target       </code> . The default is <code dir="ltr" translate="no">        use_json_null       </code> .</p>
<p><code dir="ltr" translate="no">        jsonb_set_lax       </code> has the same behavior as <code dir="ltr" translate="no">        jsonb_set       </code> unless the <code dir="ltr" translate="no">        null_value_treatment       </code> parameter is a <code dir="ltr" translate="no">        NULL       </code> null, then this function returns an error. If the <code dir="ltr" translate="no">        new_value       </code> parameter is a SQL <code dir="ltr" translate="no">        NULL       </code> then the <code dir="ltr" translate="no">        jsonb_set_lax       </code> function returns behavior based on the <code dir="ltr" translate="no">        null_value_treatment       </code> parameter.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb_strip_nulls(jsonb) → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_strip_nulls('[{"f1":1, "f2":null}, 2, null, 3]') → [{"f1": 1}, 2, null, 3]       </code></p></td>
<td>Deletes all object fields that have null values from a defined JSON array, recursively. Does not affect null values outside of object fields.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_jsonb(ANY)      </code></td>
<td><p><code dir="ltr" translate="no">        to_jsonb(1.2334000) → 1.2334000       </code></p>
<p><code dir="ltr" translate="no">        to_jsonb(true) → true       </code></p>
<p><code dir="ltr" translate="no">        to_jsonb('abc'::varchar) → abc       </code></p></td>
<td><p>Converts the given value to <code dir="ltr" translate="no">        jsonb       </code> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb_object_keys(jsonb)      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_object_keys('{"a":1, "b":{"c":1, "d":4}}'::jsonb) → {"a", "b"}       </code></p></td>
<td><p>Returns a set of keys in the top-level <code dir="ltr" translate="no">        jsonb       </code> object as a set of <code dir="ltr" translate="no">        string       </code> values. Unlike in open source PostgreSQL, this function can only be called in the <code dir="ltr" translate="no">        FROM       </code> clause.</p>
<p>The keys are de-duplicated and returned in length-first lexicographic order.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb_typeof(jsonb)      </code></td>
<td><p><code dir="ltr" translate="no">        jsonb_typeof('-123.4') → number       </code><br />
</p>
<p><code dir="ltr" translate="no">        jsonb_typeof('{"a":1, "b":2}') → object       </code></p>
<p><code dir="ltr" translate="no">        jsonb_typeof('["a", "b", "c"]') → array       </code></p>
<p><code dir="ltr" translate="no">        jsonb_typeof('null'::jsonb) → null       </code></p>
<p><code dir="ltr" translate="no">        jsonb_typeof(NULL) IS NULL → true       </code></p></td>
<td><p>Returns the type of the top-level <code dir="ltr" translate="no">        jsonb       </code> value as a text string. The possible types are <code dir="ltr" translate="no">        object       </code> , <code dir="ltr" translate="no">        array       </code> , <code dir="ltr" translate="no">        string       </code> , <code dir="ltr" translate="no">        number       </code> , <code dir="ltr" translate="no">        boolean       </code> , and <code dir="ltr" translate="no">        NULL       </code> .</p>
<p>The <code dir="ltr" translate="no">        null       </code> result shouldn't be confused with a SQL <code dir="ltr" translate="no">        NULL       </code> , as the examples illustrate.</p></td>
</tr>
</tbody>
</table>

### Spanner specific JSONB functions

Spanner has several JSONB functions that are not available in open source PostgreSQL. You must use the `  spanner  ` namespace to call these functions.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.bool_array(jsonb)      </code></td>
<td><code dir="ltr" translate="no">       spanner.bool_array('[true, false]'::jsonb) → [true, false]      </code><br />
<br />
<code dir="ltr" translate="no">       spanner.bool_array('["true"]'::jsonb) → ERROR      </code></td>
<td>Returns an array of <code dir="ltr" translate="no">       boolean      </code> values from a <code dir="ltr" translate="no">       jsonb      </code> array. Raises an error if the argument is not an array of boolean values.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.float32_array(jsonb)      </code></td>
<td><code dir="ltr" translate="no">       spanner.float32_array('[1, -2, 3.0]'::jsonb) → [1.0, -2.0, 3.0]      </code><br />
<br />
<code dir="ltr" translate="no">       spanner.float32_array('[1e100]'::jsonb) → ERROR      </code></td>
<td>Returns an array of <code dir="ltr" translate="no">       real      </code> values from a <code dir="ltr" translate="no">       jsonb      </code> array. Raises an error if the argument is not an array of number values in <code dir="ltr" translate="no">       real      </code> domain.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.float64_array(jsonb)      </code></td>
<td><code dir="ltr" translate="no">       spanner.float64_array('[1, -2, 3.0]'::jsonb) → [1.0, -2.0, 3.0]      </code><br />
<br />
<code dir="ltr" translate="no">       spanner.float64_array('[1e100]'::jsonb) → ERROR      </code></td>
<td>Returns an array of <code dir="ltr" translate="no">       real      </code> values from a <code dir="ltr" translate="no">       jsonb      </code> array. Raises an error if the argument is not an array of number values in <code dir="ltr" translate="no">       double precision      </code> domain.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.int64_array(jsonb)      </code></td>
<td><code dir="ltr" translate="no">       spanner.int64_array('[1, -2, 3.0]'::jsonb) → [1, -2, 3]      </code><br />
<br />
<code dir="ltr" translate="no">       spanner.int64_array('[1.1]'::jsonb) → ERROR      </code></td>
<td>Returns an array of <code dir="ltr" translate="no">       int8      </code> values from a <code dir="ltr" translate="no">       jsonb      </code> array. Raises an error if the argument is not an array of number values in <code dir="ltr" translate="no">       int8      </code> domain.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.jsonb_query_array(jsonb)      </code></td>
<td><code dir="ltr" translate="no">       spanner.jsonb_query_array('[1, "abc", {"k": "v"}]'::jsonb) → [1, "abc", {"k": "v"}]      </code></td>
<td>Returns an array of <code dir="ltr" translate="no">       jsonb      </code> values from a <code dir="ltr" translate="no">       jsonb      </code> array. Similar to <a href="https://www.postgresql.org/docs/14/functions-json.html">jsonb_array_elements</a> in PostgreSQL, except that it returns an array of values rather than a set of values.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.string_array(jsonb)      </code></td>
<td><code dir="ltr" translate="no">       spanner.string_array('["a", "b", "c"]'::jsonb) → ['a', 'b', 'c']      </code><br />
<br />
<code dir="ltr" translate="no">       spanner.string_array('[null]'::jsonb) → ERROR      </code></td>
<td>Returns an array of <code dir="ltr" translate="no">       text      </code> values from a <code dir="ltr" translate="no">       jsonb      </code> array. Raises an error if the argument is not an array of string values.</td>
</tr>
</tbody>
</table>

## Interval functions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       make_interval ( [ years int8 [, months int8 [, weeks int8 [, days int8 [, hours int8 [, mins int8 [, secs double precision ]]]]]]] ) → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT make_interval(years =&gt; 1, months =&gt; 2, weeks =&gt; 3, days =&gt; 15, hours =&gt; 10, mins =&gt; 30, secs =&gt; 15.1) -&gt; P1Y2M36DT10H30M15.1S      </code></td>
<td>Creates an interval from years, months, weeks, days, hours, minutes, and seconds fields. The default value for each of the fields is 0.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       -interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT -INTERVAL '1 year 2 months 15 days 45 seconds 500 microseconds';      </code> <code dir="ltr" translate="no">       Result: P-1Y-2M-15DT-45.0005S      </code></td>
<td>Negate an interval.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       interval + interval → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT INTERVAL '1 year 2 months 15 days' + INTERVAL '1 hour 15 minutes 45 seconds 500 milliseconds';      </code> <code dir="ltr" translate="no">       Result: P1Y2M15DT1H15M45.5S      </code></td>
<td>Add intervals.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       interval - interval → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT INTERVAL '1 year 2 months 10 hours 30 minutes' - INTERVAL '15 days 45 seconds 500 microseconds';      </code> <code dir="ltr" translate="no">       Result: P1Y2M-15DT10H29M14.9995S      </code></td>
<td>Subtract intervals.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       interval * double precision → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT INTERVAL '4 months 12 days 20 seconds' * 4.0;      </code> <code dir="ltr" translate="no">       Result: P1Y4M48DT1M20S      </code></td>
<td>Multiply an interval by a scalar.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       interval / double precision → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT INTERVAL '1 hour' / 2.5;      </code> <code dir="ltr" translate="no">       Result: PT24M      </code></td>
<td>Divide an interval by a scalar.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       timestamptz + interval → timestamptz      </code></td>
<td><code dir="ltr" translate="no">       SELECT TIMESTAMP WITH TIME ZONE '2021-12-18T10:00:00+00' + INTERVAL '2 months 15 days 40 minutes';      </code> <code dir="ltr" translate="no">       Result: 2022-03-05T10:40:00Z      </code></td>
<td>Add an interval to a timestamp with a time zone.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       timestamptz - interval → timestamptz      </code></td>
<td><code dir="ltr" translate="no">       SELECT TIMESTAMP WITH TIME ZONE '2024-12-18T10:00:00+00' - INTERVAL '2 months 15 days 40 minutes';      </code> <code dir="ltr" translate="no">       Result: 2024-10-03T08:20:00Z      </code></td>
<td>Subtract an interval from a timestamp with a time zone.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       timestamptz - timestamptz → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT TIMESTAMPTZ '2024-12-18T10:00:00+00' - TIMESTAMPTZ '2024-10-03T08:20:00Z'      </code> <code dir="ltr" translate="no">       Result: PT1825H40M      </code></td>
<td>Subtract timestamps with a time zone. Unlike open source PostgreSQL, Spanner doesn't convert 24 hour time periods into days.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       justify_hours(interval) → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT justify_hours(interval '50 hours 10 minutes')      </code> <code dir="ltr" translate="no">       Result: P2DT2H10M      </code> <code dir="ltr" translate="no">       SELECT justify_hours(interval '-12 day 50 hours 10 minutes')      </code> <code dir="ltr" translate="no">       Result: P-9DT-21H-50M      </code></td>
<td>Normalizes 24-hour time periods into full days. Adjusts time and days to have the same sign.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       justify_days(interval) → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT justify_days(interval '45 days 50 hours 10 minutes')      </code>
<p><code dir="ltr" translate="no">        Result: P1M15DT50H10M       </code></p>
<p><code dir="ltr" translate="no">        SELECT justify_days(interval '-1 year 45 days')       </code></p>
<code dir="ltr" translate="no">       Result: P-10M-15D      </code></td>
<td>Normalizes 30-day time periods into full months. Adjusts days and months to have the same sign.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       justify_interval(interval) → interval      </code></td>
<td><code dir="ltr" translate="no">       SELECT justify_interval(INTERVAL '29 days 60 hours')      </code> <code dir="ltr" translate="no">       Result: P1M1DT12H      </code> <code dir="ltr" translate="no">       SELECT justify_interval(INTERVAL '-34 days 60 hours')      </code> <code dir="ltr" translate="no">       Result: P-1M-1DT-12H      </code></td>
<td>Normalizes 24-hour time periods into full days, then 30-day time periods into full months. Adjusts all parts to have the same sign.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       extract(field FROM source) → numeric      </code></td>
<td><code dir="ltr" translate="no">       SELECT extract(SECOND FROM INTERVAL '1 year 2 months 15 days 10 hours 30 minutes 15 seconds 100 milliseconds')      </code> <code dir="ltr" translate="no">       Result: 15.100000      </code></td>
<td>Retrieves subfield from an interval value and returns a value of type numeric.</td>
</tr>
</tbody>
</table>

## Aggregate functions

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       array_agg(anynonarray [ORDER BY input_sort_columns])      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Inserts the given values, including nulls, into an array. <code dir="ltr" translate="no">       input_sort_columns      </code> , if specified, must have the same syntax as a query-level ORDER BY clause and is used to sort the inputs.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       avg(float4 | float8 | interval | int8 | numeric)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Computes the average (arithmetic mean) of all the non-null input values.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       bit_and(int8)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Computes the bitwise AND of all non-null input values.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       bit_or(int8)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Computes the bitwise OR of all non-null input values.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       bool_and(bool)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns <code dir="ltr" translate="no">       true      </code> if all non-null input values are <code dir="ltr" translate="no">       true      </code> , otherwise false.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       bool_or(bool)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns <code dir="ltr" translate="no">       true      </code> if any non-null input value is <code dir="ltr" translate="no">       true      </code> , otherwise false.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       count(*)      </code></td>
<td>Returns int8. <code dir="ltr" translate="no"></code></td>
<td>Computes the number of input rows.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       count(bool | bytea | float4 | float8 | interval | int8 | text | timestamptz)      </code></td>
<td>Returns int8. <code dir="ltr" translate="no"></code></td>
<td>Computes the number of input rows in which the input value is not null.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       every(bool)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Equivalent to <code dir="ltr" translate="no">       bool_and()      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       max(float4 | float8 | interval | int8 | numeric | text | timestamptz)      </code></td>
<td>Returns same type as input type. <code dir="ltr" translate="no"></code></td>
<td>Computes the maximum of the non-null input values.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       min(float4 | float8 | interval | int8 | numeric | text | timestamptz)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Computes the minimum of the non-null input values.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       string_agg(               value              bytea,               delimiter              bytea)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Concatenates the non-null input values into a string. Each value after the first is preceded by the corresponding <em>delimiter</em> (if it's not null).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       string_agg(               value              text,               delimiter              text [ORDER BY input_sort_columns])      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Concatenates the non-null input values into a string. Each value after the first is preceded by the corresponding <em>delimiter</em> (if it's not null). <code dir="ltr" translate="no">       input_sort_columns      </code> , if specified, must have the same syntax as a query-level ORDER BY clause and is used to sort the inputs.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sum(float4 | float8 | interval | int8 | numeric)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Computes the sum of the non-null input values.</td>
</tr>
</tbody>
</table>

## Conditional functions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       coalesce(ANY REPEATED)      </code></td>
<td><code dir="ltr" translate="no">       coalesce(NULL, 'abc', 'def') →  'abc'      </code></td>
<td>Returns the first of its arguments that is not null. Null is returned only if all arguments are null. It is often used to substitute a default value for null values when data is retrieved for display.
<p>The arguments must all use the same data type. The result has the same data type.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       greatest(ANY REPEATED)      </code></td>
<td><code dir="ltr" translate="no">       greatest(6, 10, 3, 14, 2) → 14      </code></td>
<td>Returns the largest value from a list of any number of expressions. The expressions must all use the same data type. The result has the same data type. NULL values in the list are ignored. The result is NULL only if all the expressions evaluate to NULL.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       least(ANY REPEATED)      </code></td>
<td><code dir="ltr" translate="no">       least(6, 10, 3, 14, 2) → 2      </code></td>
<td>Returns the smallest value from a list of any number of expressions. The expressions must all use the same data type. The result will have the same data type. NULL values in the list are ignored. The result is NULL only if all the expressions evaluate to NULL.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       nullif(               value1              ANY,               value2              ANY)      </code></td>
<td><code dir="ltr" translate="no"></code></td>
<td>Returns a null value if <code dir="ltr" translate="no">       value1      </code> equals <code dir="ltr" translate="no">       value2      </code> ; otherwise it returns <code dir="ltr" translate="no">       value1      </code> . The two arguments must use comparable types. Specifically, they are compared exactly as if you had written <code dir="ltr" translate="no">       value1 = value2      </code> , so there must be a suitable = operator available.
<p>The result has the same type as the first argument, but there is a subtle difference. What is actually returned is the first argument of the implied = operator, and in some cases that is promoted to match the second argument's type. For example, <code dir="ltr" translate="no">        NULLIF(1, 2.2)       </code> yields a numeric, because there is no integer = numeric operator, only numeric = numeric.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ISNULL      </code></td>
<td><code dir="ltr" translate="no">         datatype              ISNULL → boolean      </code></td>
<td>Tests whether value is null (non-standard syntax).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       NOTNULL      </code></td>
<td><code dir="ltr" translate="no">         datatype              NOTNULL → boolean      </code></td>
<td>Tests whether value is not null (non-standard syntax).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       IS TRUE      </code></td>
<td><code dir="ltr" translate="no">       boolean IS TRUE → boolean               true IS TRUE → true              NULL::boolean IS TRUE → false (rather than NULL)      </code></td>
<td>Tests whether boolean expression yields <code dir="ltr" translate="no">       true      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       IS NOT TRUE      </code></td>
<td><code dir="ltr" translate="no">       boolean IS NOT TRUE → boolean               true IS NOT TRUE → false              NULL::boolean IS NOT TRUE → true (rather than NULL)      </code></td>
<td>Tests whether boolean expression yields false or unknown.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       IS FALSE      </code></td>
<td><code dir="ltr" translate="no">       boolean IS FALSE → boolean               true IS FALSE → false              NULL::boolean IS FALSE → false (rather than NULL)      </code></td>
<td>Tests whether boolean expression yields false.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       IS NOT FALSE      </code></td>
<td><code dir="ltr" translate="no">       boolean IS NOT FALSE → boolean               true IS NOT FALSE → true              NULL::boolean IS NOT FALSE → true (rather than NULL)      </code></td>
<td>Tests whether boolean expression yields <code dir="ltr" translate="no">       true      </code> or unknown.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       IS UNKNOWN      </code></td>
<td><code dir="ltr" translate="no">       boolean IS UNKNOWN → boolean               true IS UNKNOWN → false              NULL::boolean IS UNKNOWN → true (rather than NULL)      </code></td>
<td>Tests whether boolean expression yields unknown.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       IS NOT UNKNOWN      </code></td>
<td><code dir="ltr" translate="no">       boolean IS NOT UNKNOWN → boolean               true IS NOT UNKNOWN → true              NULL::boolean IS NOT UNKNOWN → false (rather than NULL)      </code></td>
<td>Tests whether boolean expression yields <code dir="ltr" translate="no">       true      </code> or false.</td>
</tr>
</tbody>
</table>

## Pattern matching functions

This section describes the pattern matching functions that are available in Spanner.

### Pattern matching functions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       like(               string              bytea,               pattern              bytea)      </code></td>
<td>Returns Boolean.<br />
<code dir="ltr" translate="no"></code></td>
<td>Returns <code dir="ltr" translate="no">       true      </code> if the string matches the supplied pattern. For more information about the <code dir="ltr" translate="no">       LIKE      </code> expression, see the <a href="https://www.postgresql.org/docs/13/functions-matching.html#FUNCTIONS-LIKE">postgresql.org documentation</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       like(               string              text,               pattern              text)      </code></td>
<td>Returns Boolean.<br />
<code dir="ltr" translate="no"></code></td>
<td>Returns <code dir="ltr" translate="no">       true      </code> if the string matches the supplied pattern. For more information about the <code dir="ltr" translate="no">       LIKE      </code> expression, see the <a href="https://www.postgresql.org/docs/13/functions-matching.html#FUNCTIONS-LIKE">postgresql.org documentation</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       regexp_match(               string              text,               pattern              text)      </code></td>
<td><br />
<code dir="ltr" translate="no">       regexp_match('exambarbequeple','(bar)(beque)') → {'bar', 'beque'}      </code></td>
<td>Returns an array of matching substrings within the first match of a POSIX regular expression pattern to a string. If there is no match, then the result is <code dir="ltr" translate="no">       NULL      </code> . If there is a match, and the pattern contains parenthesized subexpressions, then the result is a text array whose <em>n</em> th element is the substring matching the <em>n</em> th parenthesized subexpression of the pattern (not counting non-capturing parentheses).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       regexp_match(               string              text,               pattern              text,               flags              text)      </code></td>
<td><br />
<code dir="ltr" translate="no">       regexp_match('examBaRBeQUEple','(bar)(beque)', 'i') → {'BaR', 'BeQUE'}      </code></td>
<td>Returns an array of matching substrings within the first match of a POSIX regular expression pattern to a string. If there is no match, the result is <code dir="ltr" translate="no">       NULL      </code> . If a match is found and the pattern contains parenthesized subexpressions, then the result is a text array whose <em>n</em> th element is the substring matching the <em>n</em> th parenthesized subexpression of the pattern (not counting non-capturing parentheses). The flags parameter contains zero or more single-letter flags that change the function's behavior. For more information about using flags, see the open source PostgreSQL <a href="https://www.postgresql.org/docs/current/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE">Embedded-option Table documentation</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       regexp_split_to_array(               string              text,               pattern              text)      </code></td>
<td><br />
<code dir="ltr" translate="no">       regexp_split_to_array('the quick brown fox jumps over the lazy dog','\s+')       → {'the','quick',''brown','fox','jumps','over','the','lazy','dog'}      </code></td>
<td>Splits a string using a POSIX regular expression pattern as a delimiter. If there is no match to the pattern, the function returns the string. If there is at least one match, then for each match, the function returns the text from the end of the last match (or the beginning of the string) to the beginning of the match. When there are no more matches, the function returns the text from the end of the last match to the end of the string.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       regexp_split_to_array(               string              text,               pattern              text,               flags              text)      </code></td>
<td><br />
<code dir="ltr" translate="no">       regexp_split_to_array('thE QUick bROWn FOx jUMPs ovEr The lazy dOG','e', 'i')       → {'th',' QUick bROWn FOx jUMPs ov','r Th',' lazy dOG'}      </code></td>
<td>Splits a string using a POSIX regular expression pattern as a delimiter. If there is no match to the pattern, then the function returns the string. If there is at least one match, then for each match, the function returns the text from the end of the last match (or the beginning of the string) to the beginning of the match. When there are no more matches, the function returns the text from the end of the last match to the end of the string. The flags parameter contains zero or more single-letter flags that change the function's behavior. For more information about using flags, see the open source PostgreSQL <a href="https://www.postgresql.org/docs/current/functions-matching.html#POSIX-EMBEDDED-OPTIONS-TABLE">Embedded-option Table documentation</a> .</td>
</tr>
</tbody>
</table>

## Formatting functions

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_char(               interval_value              interval,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       SELECT to_char(INTERVAL '1 year 2 months 15 days 10 hours 30 minutes 15 seconds 100 milliseconds', 'YYYY-MM-DD HH24:MI:SS.MS');      </code></td>
<td>Converts interval to string according to the given date format. <a href="#footnote1"><sup>[1]</sup></a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_char(               number              int8,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_char(125, '999') → 125      </code></td>
<td>Converts int8 to string according to the given format. <a href="#footnote2"><sup>[2]</sup></a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_char(               number              numeric,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_char(-125.8, '999D99S') → 125.8-      </code></td>
<td>Converts numeric to string according to the given format. <a href="#footnote2"><sup>[2]</sup></a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_char(               number              float4,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_char(125.8::float4, '999D9') → 125.8      </code></td>
<td>Converts float4 to string according to the given format. <a href="#footnote2"><sup>[2]</sup></a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_char(               number              float8,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_char(125.8::float8, '999D9') → 125.8      </code></td>
<td>Converts float8 to string according to the given format. <a href="#footnote2"><sup>[2]</sup></a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_char(               timestamp              timestamptz,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_char(timestamp '2002-04-20 17:31:12.66', 'HH12:MI:SS') → 05:31:12      </code></td>
<td>Converts timestamptz to string according to the given date format. <a href="#footnote3"><sup>[3]</sup></a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_number(               number              text,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_number('12,454.8-', '99G999D9S') → -12454.8      </code></td>
<td>Converts string to numeric according to the given format. <a href="#footnote3"><sup>[2]</sup></a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_date(               date              text,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_date('05 Dec 2000', 'DD Mon YYYY') → 2000-12-05      </code></td>
<td>Converts string to date according to the given date format. <a href="#footnote3"><sup>[3]</sup></a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_timestamp(               timestamp              timestamptz,               format              text)      </code></td>
<td><code dir="ltr" translate="no">       to_timestamp('05 Dec 2000', 'DD Mon YYYY') → 2000-12-05 00:00:00-05      </code></td>
<td>Converts string to timestamptz format according to the given date time format. <a href="#footnote3"><sup>[3]</sup></a></td>
</tr>
</tbody>
</table>

<sup>\[1\]</sup> For a list of supported numeric formatting, see [Template Patterns for Numeric Formatting](https://www.postgresql.org/docs/current/functions-formatting.html#FUNCTIONS-FORMATTING-NUMERIC-TABLE) .

<sup>\[2\]</sup> For a list of supported numeric formatting, see [Template Patterns for Numeric Formatting](https://www.postgresql.org/docs/current/functions-formatting.html#FUNCTIONS-FORMATTING-NUMERIC-TABLE) .

<sup>\[3\]</sup> For a list of supported date/time formatting, see [Supported formats for `  date  ` data type](/spanner/docs/reference/postgresql/data-types#supported-date) and [Supported formats for `  timestamptz  ` data type](/spanner/docs/reference/postgresql/data-types#supported-timestamptz) .

## Sequence functions

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       nextval (varchar) → bigint      </code></td>
<td><code dir="ltr" translate="no">       nextval ('MySequence')      </code></td>
<td>Takes a sequence name string and returns the next sequence value in the <code dir="ltr" translate="no">       bigint      </code> data type. This function is only allowed in read-write transactions</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner.get_internal_sequence_state(varchar)      </code></td>
<td><code dir="ltr" translate="no">       spanner.get_internal_sequence_state('MySequence')      </code></td>
<td>Gets the current sequence internal counter before bit reversal. As the sequence generates values, its internal counter changes. This function is useful when using import or export, and for migrations. If <code dir="ltr" translate="no">       nextval ('MySequence')      </code> is never called on the sequence, then this function returns NULL.</td>
</tr>
</tbody>
</table>

## Set returning functions

This section describes functions that possibly return more than one row.

Unlike open source PostgreSQL, Spanner only supports these functions in the \`FROM\` clause.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       generate_series(               start              int8 | numeric,               end              int8 | numeric [,               step              int8 | numeric])      </code></td>
<td><p><code dir="ltr" translate="no">        SELECT * FROM generate_series(2, 4)       </code></p>
<pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>      /*---------------------*
      | generate_series      |
      +----------------------+
      | 2                    |
      | 3                    |
      | 4                    |
      *---------------------*/
      </code></pre>
<p><code dir="ltr" translate="no">        SELECT * FROM generate_series(3, 0, -2)       </code></p>
<pre class="text" dir="ltr" data-is-upgraded="" translate="no"><code>      /*---------------------*
      | generate_series      |
      +----------------------+
      | 3                    |
      | 1                    |
      *---------------------*/
      </code></pre></td>
<td><p>Generates a series of values. The function accepts two arguments: the <em>start</em> value and the <em>end</em> value. An optional third <em>step</em> argument specifies the increment between the first two arguments (default is 1).</p>
<p>This function generates a set of rows, with each row representing a value from the series. The series begins at the <em>start</em> value and includes values up to and including the <em>end</em> value, or until the final increment is reached.</p></td>
</tr>
</tbody>
</table>

## Utility functions

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner.generate_uuid()      </code></td>
<td><code dir="ltr" translate="no">       SELECT spanner.generate_uuid() AS uuid  →     4192bff0-e1e0-43ce-a4db-912808c32493      </code></td>
<td>Returns a random universally unique identifier (UUID) (Version 4) as a string. that Spanner can use for primary key columns. The returned string consists of 32 hexadecimal digits in five groups separated by hyphens in the form 8-4-4-4-12. The hexadecimal digits represent 122 random bits and 6 fixed bits, in compliance with RFC 4122 section 4.4. The returned string is lowercase.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       gen_random_uuid()      </code></td>
<td><code dir="ltr" translate="no">       gen_random_uuid() -&gt; uuid      </code></td>
<td>Returns a random universally unique identifier (UUID) (Version 4).</td>
</tr>
</tbody>
</table>
