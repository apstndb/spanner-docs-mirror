Spanner provides the `  NUMERIC  ` type that can store decimal precision numbers exactly. The semantics of the `  NUMERIC  ` type in Spanner varies between its two SQL dialects (GoogleSQL and PostgreSQL), especially around the limits on [scale and precision](#precision) :

  - [`  NUMERIC  `](/spanner/docs/reference/postgresql/data-types) in the PostgreSQL dialect is an [arbitrary decimal precision](https://en.wikipedia.org/wiki/Arbitrary-precision_arithmetic) numeric type (scale or precision can be any number within the supported range) and thus is an ideal choice for storing arbitrary precision numeric data.

  - [`  NUMERIC  `](/spanner/docs/working-with-numerics#numeric) in GoogleSQL is a [fixed precision](https://en.wikipedia.org/wiki/Fixed-point_arithmetic) numeric type (precision=38 and scale=9) and cannot be used to store arbitrary precision numeric data. When you need to store arbitrary precision numbers in GoogleSQL dialect databases, we recommend that you [store them as strings](#recommendation_store_arbitrary_precision_numbers_as_strings) .

## Precision of Spanner numeric types

Precision is the number of digits in a number. Scale is the number of digits to the right of the decimal point in a number. For example, the number 123.456 has a precision of 6 and a scale of 3. Spanner has three numeric types:

  - 64-bit signed integer type called `  INT64  ` in the GoogleSQL dialect and `  INT8  ` in the PostgreSQL dialect.
  - IEEE 64-bit (double) binary precision floating-point type called `  FLOAT64  ` in the GoogleSQL dialect and `  FLOAT8  ` in the PostgreSQL dialect.
  - Decimal precision `  NUMERIC  ` type.

Let's look at each in terms of precision and scale.

[`  INT64  `](/spanner/docs/reference/standard-sql/data-types#integer_type) / [`  INT8  `](/spanner/docs/reference/postgresql/data-types) represents numeric values that don't have a fractional component. This data type provides 18 digits of precision, with a scale of zero.

[`  FLOAT64  `](/spanner/docs/reference/standard-sql/data-types#floating_point_types) / [`  FLOAT8  `](/spanner/docs/reference/postgresql/data-types) can only represent approximate decimal numeric values with fractional components and provides 15 to 17 significant digits (count of digits in a number with all trailing zeros removed) of decimal precision. We say that this type represents *approximate* decimal numeric values because [IEEE 64-bit floating point](https://ieeexplore.ieee.org/document/4610935) binary representation that Spanner uses cannot precisely represent decimal (base-10) fractions (it can represent only base-2 fractions exactly). This loss of precision introduces rounding errors for some decimal fractions.

For example, when you store the decimal value 0.2 using the `  FLOAT64  ` / `  FLOAT8  ` data type, the binary representation converts back to a decimal value of 0.20000000000000001 (to 18 digits of precision). Similarly (1.4 \* 165) converts back to 230.999999999999971 and (0.1 + 0.2) converts back to 0.30000000000000004. This is why 64-bit floats are described as only having 15-17 significant digits of precision (only some numbers with more than 15 decimal digits can be represented as 64-bit float without rounding). For more details on how floating point precision is calculated, see [Double-precision floating-point format](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) .

Neither `  INT64  ` / `  INT8  ` nor `  FLOAT64  ` / `  FLOAT8  ` has the ideal precision for financial, scientific, or engineering calculations, where a precision of 30 digits or more is commonly required.

The `  NUMERIC  ` data type is suitable for those applications, since it is capable of representing exact decimal precision numeric values having precision of more than 30 decimal digits.

The GoogleSQL [`  NUMERIC  `](/spanner/docs/reference/standard-sql/data-types#numeric_type) data type can represent numbers with a fixed decimal precision of 38 and fixed scale of 9. The range of GoogleSQL `  NUMERIC  ` is -99999999999999999999999999999.999999999 to 99999999999999999999999999999.999999999.

The PostgreSQL dialect `  NUMERIC  ` type can represent numbers with a maximum decimal precision of 147,455 and a maximum scale of 16,383.

If you need to store numbers that are larger than the precision and scale offered by `  NUMERIC  ` , the following sections describe some recommended solutions.

## Recommendation: store arbitrary precision numbers as strings

When you need to store an arbitrary precision number in a Spanner database, and you need more precision than `  NUMERIC  ` provides, we recommend that you store the value as its decimal representation in a `  STRING  ` / `  VARCHAR  ` column. For example, the number `  123.4  ` is stored as the string `  "123.4"  ` .

With this approach, your application must perform a lossless conversion between the application-internal representation of the number and the `  STRING  ` / `  VARCHAR  ` column value for database reads and writes.

Most arbitrary precision libraries have built-in methods to perform this lossless conversion. In Java, for example, you can use the [`  BigDecimal.toPlainString()  `](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html#toPlainString\(\)) method and the [`  BigDecimal(String)  `](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html#%3Cinit%3E\(java.lang.String\)) constructor.

Storing the number as a string has the advantage that the value is stored with exact precision (up to the `  STRING  ` / `  VARCHAR  ` column length limit), and the value remains human-readable.

### Perform exact aggregations and calculations

To perform *exact* aggregations and calculations on string representations of arbitrary precision numbers, your application must perform these calculations. You cannot use SQL aggregate functions.

For example, to perform the equivalent of a SQL `  SUM(value)  ` over a range of rows, the application must query the string values for the rows, then convert and sum them internally in the app.

### Perform approximate aggregations, sorting, and calculations

You can use SQL queries to perform *approximate* aggregate calculations by casting the values to `  FLOAT64  ` / `  FLOAT8  ` .

### GoogleSQL

``` text
SELECT SUM(CAST(value AS FLOAT64)) FROM my_table
```

### PostgreSQL

``` text
SELECT SUM(value::FLOAT8) FROM my_table
```

Similarly, you can sort by numeric value or limit values by range with casting:

### GoogleSQL

``` text
SELECT value FROM my_table ORDER BY CAST(value AS FLOAT64);
SELECT value FROM my_table WHERE CAST(value AS FLOAT64) > 100.0;
```

### PostgreSQL

``` text
SELECT value FROM my_table ORDER BY value::FLOAT8;
SELECT value FROM my_table WHERE value::FLOAT8 > 100.0;
```

These calculations are approximate to the limits of the `  FLOAT64  ` / `  FLOAT8  ` data type.

## Alternatives

There are other ways to store arbitrary precision numbers in Spanner. If storing arbitrary precision numbers as strings does not work for your application, consider the following alternatives:

### Store application-scaled integer values

To store arbitrary precision numbers, you can pre-scale the values before writing, so that numbers are always stored as integers, and re-scale the values after reading. Your application stores a fixed scale factor, and the precision is limited to the 18 digits provided by the `  INT64  ` / `  INT8  ` data type.

Take, for example, a number that needs to be be stored with an accuracy of 5 decimal places. The application converts the value to an integer by multiplying it by 100,000 (shifting the decimal point 5 places to the right), so the value 12.54321 is stored as `  1254321  ` .

In monetary terms, this approach is like storing dollar values as multiples of milli-cents, similar to storing time units as milliseconds.

The application determines the fixed scaling factor. If you change the scaling factor, you must convert all of the previously scaled values in your database.

This approach stores values that are human-readable (assuming you know the scaling factor). Also, you can use SQL queries to perform calculations directly on values stored in the database, as long as the result is scaled correctly and does not overflow.

### Store the unscaled integer value and the scale in separate columns

You can also store arbitrary precision numbers in Spanner using two elements:

  - The unscaled integer value stored in a byte array.
  - An integer that specifies the scaling factor.

First your application converts the arbitrary precision decimal into an unscaled integer value. For example, the application converts `  12.54321  ` to `  1254321  ` . The scale for this example is `  5  ` .

Then the application converts the unscaled integer value into a byte array using a standard portable binary representation (for example, big-endian [two's complement](https://en.wikipedia.org/wiki/Two%27s_complement) ).

The database then stores the byte array ( `  BYTES  ` / `  BYTEA  ` ) and integer scale ( `  INT64  ` / `  INT8  ` ) in two separate columns, and converts them back on read.

In Java, you can use [`  BigDecimal  `](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigDecimal.html) and [`  BigInteger  `](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigInteger.html) to perform these calculations:

``` text
byte[] storedUnscaledBytes = bigDecimal.unscaledValue().toByteArray();
int storedScale = bigDecimal.scale();
```

You can read back to a Java `  BigDecimal  ` using the following code:

``` text
BigDecimal bigDecimal = new BigDecimal(
    new BigInteger(storedUnscaledBytes),
    storedScale);
```

This approach stores values with arbitrary precision and a portable representation, but the values are not human-readable in the database, and all calculations must be performed by the application.

### Store application internal representation as bytes

Another option is to serialize the arbitrary precision decimal values to byte arrays using the application's internal representation, then store them directly in the database.

The stored database values are not human-readable, and the application needs to perform all calculations.

This approach has portability issues. If you try to read the values with a programming language or library different from the one that originally wrote it, it might not work. Reading the values back might not work because different arbitrary precision libraries can have different serialized representations for byte arrays.

## What's next

  - Read about other [data types](/spanner/docs/reference/standard-sql/data-types#numeric_types) available for Spanner.
  - Learn how to correctly set up a Spanner [schema design and data model](/spanner/docs/schema-and-data-model) .
  - Learn about [optimizing your schema design for Spanner](/spanner/docs/whitepapers/optimizing-schema-design) .
