This page describes the dialect differences between GoogleSQL and PostgreSQL and offers recommendations for using PostgreSQL approaches for specific GoogleSQL features.

## GoogleSQL dialect feature differences

GoogleSQL feature

PostgreSQL dialect recommendation

[Sample datasets](/spanner/docs/create-manage-databases#use-datasets)

No recommendation available.

[BigQuery external datasets](/bigquery/docs/spanner-external-datasets)

Use [Spanner federated queries](/bigquery/docs/spanner-federated-queries) .

[`  ENUM  `](/spanner/docs/reference/standard-sql/data-types#enum_type)

Use `  TEXT  ` columns with checked constraints instead. Unlike `  ENUMS  ` , the sort order of a `  TEXT  ` column can't be user-defined. The following example restricts the column to only support the `  'C'  ` , `  'B'  ` , and `  'A'  ` values.

``` sql
CREATE TABLE singers (
 singer_id BIGINT PRIMARY KEY,
 type TEXT NOT NULL CHECK (type IN ('C', 'B', 'A'))
);
       
```

[`  GROUP_METHOD  ` hint](/spanner/docs/reference/standard-sql/query-syntax#group_hints)

No recommendation available.

[Graph](/spanner/docs/graph/overview)

No recommendation available.

[`  HAVING MAX  ` or `  HAVING MIN  `](/spanner/docs/reference/standard-sql/aggregate-function-calls#aggregate_function_call_syntax)

Use a `  JOIN  ` or a subquery to filter for the `  MAX  ` or `  MIN  ` value for the aggregation. The following example requires filtering `  MAX  ` or `  MIN  ` in a subquery.

``` sql
WITH amount_per_year AS (
 SELECT 1000 AS amount, 2025 AS year
 UNION ALL
 SELECT 10000, 2024
 UNION ALL
 SELECT 500, 2023
 UNION ALL
 SELECT 1500, 2025
 UNION ALL
 SELECT 20000, 2024
)

SELECT SUM(amount) AS max_year_amount_sum
FROM amount_per_year
WHERE year = (SELECT MAX(year) FROM amount_per_year);
```

[Informational foreign keys](/spanner/docs/foreign-keys/overview#use-informational-foreign-keys)

No recommendation available.

[`  JSON  ` data type](/spanner/docs/reference/standard-sql/data-types#json_type)

Use the [`  JSONB  ` data type.](/spanner/docs/reference/postgresql/data-types)

`  SELECT to_json(table) FROM table  `

We recommend explicitly mapping each column with the `  jsonb_build_object  ` function:

``` sql
WITH singers AS (
  SELECT 1::int8 AS id, 'Singer First Name'::text AS first_name
)

SELECT jsonb_build_object('id', id, 'first_name', first_name)
FROM singers;
```

[`  ORDER BY … COLLATE …  `](/spanner/docs/reference/standard-sql/collation-concepts#collate_about)

No recommendation available.

`  NUMERIC  ` column as a primary key, secondary index, or foreign key

We recommend using an index over a `  TEXT  ` generated column, as shown in the following example:

``` sql
CREATE TABLE singers(
 id numeric NOT NULL,
 pk text GENERATED ALWAYS AS (id::text) STORED,
 PRIMARY KEY(pk)
);
```

[Protocol buffer](/spanner/docs/reference/standard-sql/data-definition-language#protocol-buffers) data type

You can store serialized protocol buffers as the PostgreSQL `  BYTEA  ` data type .

[`  PRIMARY KEY DESC  `](/spanner/docs/schema-design#ordering_timestamp-based_keys)

No recommendation available.

[`  SELECT AS VALUE  `](/spanner/docs/reference/standard-sql/query-syntax#select_as_value)

[`  SELECT * EXCEPT  `](/spanner/docs/reference/standard-sql/query-syntax#select_except)

We recommend that you spell out all columns in the `  SELECT  ` statement.

[`  SELECT * REPLACE  `](/spanner/docs/reference/standard-sql/query-syntax#select_replace)

We recommend that you spell out all columns in the `  SELECT  ` statement.

The following columns in the `  SPANNER_SYS  ` statistics tables:

  - [Transaction statistics](/spanner/docs/introspection/transaction-statistics) : `  TOTAL_LATENCY_DISTRIBUTION  ` and `  OPERATIONS_BY_TABLE  `
  - [Query statistics](/spanner/docs/introspection/query-statistics) : `  LATENCY_DISTRIBUTION  `
  - [Lock Statistics](/spanner/docs/introspection/lock-statistics) : `  SAMPLE_LOCK_REQUESTS  `

No recommendation available.

[`  TABLESAMPLE  `](/spanner/docs/reference/standard-sql/query-syntax#tablesample_operator)

We recommend that you apply a custom function `  F  ` , which converts a row to `  TEXT  ` or `  BYTEA  ` . You can then use `  spanner.farm_fingerprint  ` to sample your data.  
  
In the following example, we use `  CONCAT  ` as our function `  F  ` :

``` sql
-- Given the following schema

CREATE TABLE singers (
 singer_id BIGINT PRIMARY KEY,
 first_name VARCHAR(1024),
 last_name VARCHAR(1024),
 singer_info BYTEA
);

-- Create a hash for each row (using all columns)
WITH hashed_rows AS (
  SELECT
    *,
    ABS(MOD(spanner.farm_fingerprint(
      CONCAT(
        singer_id::text,
        first_name,
        last_name,
        singer_info::text
      )
    ), 100)) AS hash_value
  FROM singers
)

-- Sample data

SELECT *
FROM hashed_rows
WHERE hash_value < 10 -- sample roughly 10%
LIMIT 10; /* Optional: LIMIT to a max of 10 rows
             to be returned */
```

[`  VALUE IN UNNEST(ARRAY(...))  `](/spanner/docs/reference/standard-sql/subqueries#in_subquery_concepts)

Use the equality operator with the `  ANY  ` function, as shown in the following example:

``` sql
SELECT value = any(array[...])
```

## GoogleSQL dialect function differences

GoogleSQL function

PostgreSQL dialect recommendation

[`  ACOSH  `](/spanner/docs/reference/standard-sql/mathematical_functions#acosh)

Use the formula of the function explicitly, as shown in the following example:  

``` sql
SELECT LN(x + SQRT(x*x - 1));
```

[`  APPROX_COSINE_DISTANCE  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance)

No recommendation available.

[`  APPROX_DOT_PRODUCT  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product)

[`  APPROX_EUCLIDEAN_DISTANCE  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance)

[`  ANY_VALUE  `](/spanner/docs/reference/standard-sql/aggregate_functions#any_value)

Workaround available outside of aggregation and `  GROUP BY  ` . Use a subquery with the `  ORDER BY  ` or `  LIMIT  ` clauses, as shown in the following example:

``` sql
SELECT * FROM
(
  (expression)
  UNION ALL SELECT NULL, … -- as many columns as you have
) AS rows
ORDER BY 1 NULLS LAST
LIMIT 1;
```

[`  ARRAY_CONCAT_AGG  `](/spanner/docs/reference/standard-sql/aggregate_functions#array_concat_agg)

You can use `  ARRAY_AGG  ` and `  UNNEST  ` as shown in the following example:

``` sql
WITH albums AS
(
  SELECT ARRAY['Song A', NULL, 'Song B'] AS songs
  UNION ALL
  SELECT NULL
  UNION ALL
  SELECT ARRAY[]::TEXT[]
)
SELECT ARRAY_AGG(song) FROM albums, UNNEST(songs) song;
      
```

[`  ARRAY_FIRST  `](/spanner/docs/reference/standard-sql/array_functions#array_first)

Use the array subscript operator, as shown in the following example:

``` sql
SELECT array_expression[1];
```

Note that this will return `  NULL  ` for empty arrays.

[`  ARRAY_INCLUDES  `](/spanner/docs/reference/standard-sql/array_functions#array_includes)

Use the equality operator with the `  ANY  ` function, as shown in the following example:

``` sql
SELECT search_value = ANY(array_to_search);
```

[`  ARRAY_INCLUDES_ALL  `](/spanner/docs/reference/standard-sql/array_functions#array_includes_all)

Use the array contains operator, as shown in the following example:  

``` sql
SELECT array_to_search @> search_values;
```

[`  ARRAY_INCLUDES_ANY  `](/spanner/docs/reference/standard-sql/array_functions#array_includes_any)

Use the array overlap operator, as shown in the following example:  

``` sql
SELECT array_to_search && search_values;
```

[`  ARRAY_IS_DISTINCT  `](/spanner/docs/reference/standard-sql/array_functions#array_is_distinct)

Use a subquery to count distinct values and compare them to the original array length, as shown in the following example:  

``` sql
SELECT ARRAY_LENGTH(value, 1) = (
SELECT COUNT(DISTINCT e)
FROM UNNEST(value) AS e);
```

[`  ARRAY_LAST  `](/spanner/docs/reference/standard-sql/array_functions#array_last)

Use the array subscript operator, as shown in the following example

``` sql
SELECT (value)[ARRAY_LENGTH(value, 1)];
      
```

This returns `  NULL  ` for empty arrays.

[`  ARRAY_MAX  `](/spanner/docs/reference/standard-sql/array_functions#array_max)

Use a subquery with `  UNNEST  ` and the `  MAX  ` function, as shown in the following example:

``` sql
SELECT MAX(e) FROM UNNEST(value) AS e;
      
```

[`  ARRAY_MIN  `](/spanner/docs/reference/standard-sql/array_functions#array_min)

Use a subquery with `  UNNEST  ` and the `  MIN  ` function, as shown in the following example:

``` sql
SELECT MIN(e) FROM UNNEST(value) AS e;
      
```

[`  ARRAY_REVERSE  `](/spanner/docs/reference/standard-sql/array_functions#array_reverse)

No recommendation available.

[`  ASINH  `](/spanner/docs/reference/standard-sql/mathematical_functions#asinh)

Use the formula of the function explicitly, as shown in the following example:  

``` sql
SELECT LN(x + SQRT(x*x - 1));
```

[`  ATANH  `](/spanner/docs/reference/standard-sql/mathematical_functions#atanh)

Use the formula of the function explicitly, as shown in the following example:  

``` sql
SELECT 0.5 * LN((1 + x) / (1 - x));
```

[`  BIT_COUNT  `](/spanner/docs/reference/standard-sql/bit_functions#bit_count)

No recommendation available.

[`  BIT_XOR  `](/spanner/docs/reference/standard-sql/aggregate_functions#bit_xor)

[`  BYTE_LENGTH  `](/spanner/docs/reference/standard-sql/string_functions#byte_length)

[`  CODE_POINTS_TO_BYTES  `](/spanner/docs/reference/standard-sql/string_functions#code_points_to_bytes)

[`  CODE_POINTS_TO_STRING  `](/spanner/docs/reference/standard-sql/string_functions#code_points_to_string)

[`  COSH  `](/spanner/docs/reference/standard-sql/mathematical_functions#cosh)

Use the formula of the function explicitly, as shown in the following example:  

``` sql
SELECT (EXP(x) + EXP(-x)) / 2;
      
```

[`  ERROR  `](/spanner/docs/reference/standard-sql/debugging_functions#error)

No recommendation available.

[`  FROM_BASE32  `](/spanner/docs/reference/standard-sql/string_functions#from_base32)

[`  FROM_BASE64  `](/spanner/docs/reference/standard-sql/string_functions#from_base64)

[`  FROM_HEX  `](/spanner/docs/reference/standard-sql/string_functions#from_hex)

[`  GENERATE_ARRAY  `](/spanner/docs/reference/standard-sql/array_functions#generate_array)

[`  GENERATE_DATE_ARRAY  `](/spanner/docs/reference/standard-sql/array_functions#generate_date_array)

[`  NET.HOST  `](/spanner/docs/reference/standard-sql/net_functions#nethost)

Use a regular expression and the `  substring  ` function, as shown in the following example:

``` sql
/* Use modified regular expression from
  https://tools.ietf.org/html/rfc3986#appendix-A. */

SELECT Substring('http://www.google.com/test' FROM
  '^(?:[^:/?#]+:)?(?://)?([^/?#]*)?[^?#]*(?:\\?[^#]*)?(?:#.*)?')
```

[`  NET.IP_FROM_STRING  `](/spanner/docs/reference/standard-sql/net_functions#netip_from_string)

No recommendation available.

[`  NET.IP_NET_MASK  `](/spanner/docs/reference/standard-sql/net_functions#netip_net_mask)

[`  NET.IP_TO_STRING  `](/spanner/docs/reference/standard-sql/net_functions#netip_to_string)

[`  NET.IP_TRUNC  `](/spanner/docs/reference/standard-sql/net_functions#netip_trunc)

[`  NET.IPV4_FROM_INT64  `](/spanner/docs/reference/standard-sql/net_functions#netipv4_from_int64)

[`  NET.IPV4_TO_INT64  `](/spanner/docs/reference/standard-sql/net_functions#netipv4_to_int64)

[`  NET.PUBLIC_SUFFIX  `](/spanner/docs/reference/standard-sql/net_functions#netpublic_suffix)

[`  NET.REG_DOMAIN  `](/spanner/docs/reference/standard-sql/net_functions#netreg_domain)

[`  NET.SAFE_IP_FROM_STRING  `](/spanner/docs/reference/standard-sql/net_functions#netsafe_ip_from_string)

[`  NORMALIZE  `](/spanner/docs/reference/standard-sql/string_functions#normalize)

[`  NORMALIZE_AND_CASEFOLD  `](/spanner/docs/reference/standard-sql/string_functions#normalize_and_casefold)

[`  REGEXP_EXTRACT_ALL  `](/spanner/docs/reference/standard-sql/string_functions#regexp_extract_all)

[`  SAFE.ADD  `](/spanner/docs/reference/standard-sql/mathematical_functions#safe_add)

We recommend that you protect against an overflow explicitly leveraging the `  NUMERIC  ` data type.

``` sql
WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 1, 2
)

SELECT
 CASE
   WHEN a::numeric + b::numeric > 9223372036854775807 THEN NULL
   WHEN a + b < -9223372036854775808 THEN NULL
   ELSE a + b
 END AS result
FROM numbers;
```

[`  SAFE.CAST  `](/spanner/docs/reference/standard-sql/conversion_functions#safe_casting)

No recommendation available.

[`  SAFE.CONVERT_BYTES_TO_STRING  `](/spanner/docs/reference/standard-sql/string_functions#safe_convert_bytes_to_string)

[`  SAFE.DIVIDE  `](/spanner/docs/reference/standard-sql/mathematical_functions#safe_divide)

We recommend that you protect against an overflow explicitly leveraging the `  NUMERIC  ` data type during a division operation.

``` sql
WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 10, 2
)

SELECT
 CASE
   WHEN b = 0 THEN NULL
   WHEN a::numeric / b::numeric > 9223372036854775807 THEN NULL
   WHEN a::numeric / b::numeric < -9223372036854775808 THEN NULL
   ELSE a / b
 END AS result
FROM numbers;
```

[`  SAFE.MULTIPLY  `](/spanner/docs/reference/standard-sql/mathematical_functions#safe_multiply)

We recommend that you protect against an overflow explicitly leveraging the `  NUMERIC  ` data type during a multiplication operation.

``` sql
WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 1, 2
)

SELECT
 CASE
   WHEN a::numeric * b::numeric > 9223372036854775807 THEN NULL
   WHEN a::numeric * b::numeric < -9223372036854775808 THEN NULL
   ELSE a * b
 END AS result
FROM numbers;
```

[`  SAFE.NEGATE  `](/spanner/docs/reference/standard-sql/mathematical_functions#safe_negate)

We recommend that you protect against an overflow explicitly leveraging the `  NUMERIC  ` data type during a negation operation.

``` sql
WITH numbers AS
(
  SELECT 9223372036854775807 AS a
  UNION ALL
  SELECT -9223372036854775808
)

SELECT
 CASE
   WHEN a <= -9223372036854775808 THEN NULL
   WHEN a >= 9223372036854775809 THEN NULL
   ELSE -a
 END AS result
FROM numbers;
```

[`  SAFE.SUBTRACT  `](/spanner/docs/reference/standard-sql/mathematical_functions#safe_subtract)

We recommend that you protect against an overflow explicitly leveraging the `  NUMERIC  ` data type during a subtraction operation.

``` sql
WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 1, 2
)

SELECT
 CASE
   WHEN a::numeric - b::numeric > 9223372036854775807 THEN NULL
   WHEN a::numeric - b::numeric < -9223372036854775808 THEN NULL
   ELSE a - b
 END AS result
FROM numbers;
```

[`  SAFE.TO_JSON  `](/spanner/docs/reference/standard-sql/json_functions#safe_to_json)

No recommendation available.

[`  SINH  `](/spanner/docs/reference/standard-sql/mathematical_functions#sinh)

Use the formula of the function explicitly, as shown in the following example:  

``` sql
SELECT (EXP(x) - EXP(-x)) / 2;
```

[`  SPLIT  `](/spanner/docs/reference/standard-sql/string_functions#split)

Use the `  regexp_split_to_array  ` function, as shown in the following example:

``` sql
WITH letters AS
(
  SELECT '' as letter_group
  UNION ALL
  SELECT 'a' as letter_group
  UNION ALL
  SELECT 'b c d' as letter_group
)

SELECT regexp_split_to_array(letter_group, ' ') as example
FROM letters;
```

[`  STDDEV  `](/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev)

Use the formula of the function explicitly (unbiased standard deviation), as shown in the following example:  

``` sql
WITH numbers AS
(
  SELECT 1 AS x
  UNION ALL
  SELECT 2
  UNION ALL
  SELECT 3
),

mean AS
(
  SELECT AVG(x)::float8 AS mean
  FROM numbers
)

SELECT SQRT(SUM(POWER(numbers.x - mean.mean, 2)) / (COUNT(x) - 1))
  AS stddev
FROM numbers
CROSS JOIN mean
```

[`  STDDEV_SAMP  `](/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev_samp)

Use the formula of the function explicitly (unbiased standard deviation), as shown in the following example:  

``` sql
WITH numbers AS
(
  SELECT 1 AS x
  UNION ALL
  SELECT 2
  UNION ALL
  SELECT 3
),

mean AS (
  SELECT AVG(x)::float8 AS mean
  FROM numbers
)

SELECT SQRT(SUM(POWER(numbers.x - mean.mean, 2)) / (COUNT(x) - 1))
  AS stddev
FROM numbers
CROSS JOIN mean
      
```

[`  TANH  `](/spanner/docs/reference/standard-sql/mathematical_functions#tanh)

Use the formula of the function explicitly.  

``` sql
SELECT (EXP(x) - EXP(-x)) / (EXP(x) + EXP(-x));
```

[`  TIMESTAMP_MICROS  `](/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_micros)

Use the `  to_timestamp  ` function and truncate the microseconds part of the input (precision loss), as shown in the following example:

``` sql
SELECT to_timestamp(1230219000123456 / 1000000);
```

[`  TIMESTAMP_MILLIS  `](/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_millis)

Use the `  to_timestamp  ` function and truncate the milliseconds part of the input (precision loss), as shown in the following example:

``` sql
SELECT to_timestamp(1230219000123 / 1000);
```

[`  TO_BASE32  `](/spanner/docs/reference/standard-sql/string_functions#to_base32)

No recommendation available.

[`  TO_BASE64  `](/spanner/docs/reference/standard-sql/string_functions#to_base64)

[`  TO_CODE_POINTS  `](/spanner/docs/reference/standard-sql/string_functions#to_code_points)

[`  TO_HEX  `](/spanner/docs/reference/standard-sql/string_functions#to_hex)

[`  VAR_SAMP  `](/spanner/docs/reference/standard-sql/statistical_aggregate_functions#var_samp)

Use the formula of the function explicitly (unbiased variance), as shown in the following:  

``` sql
-- Use formula directly (unbiased)

WITH numbers AS
(
  SELECT 1 AS x
  UNION ALL
  SELECT 2
  UNION ALL
  SELECT 3 ), mean AS
(
  SELECT Avg(x)::float8 AS mean
  FROM   numbers )
SELECT Sum(Power(numbers.x - mean.mean, 2)) / (Count(x) - 1)
  AS variance
FROM numbers
CROSS JOIN mean
```

[`  VARIANCE  `](/spanner/docs/reference/standard-sql/statistical_aggregate_functions#variance)

Use the formula of the function explicitly (unbiased variance), as shown in the following example:  

``` sql
-- Use formula directly (unbiased VARIANCE like VAR_SAMP)

WITH numbers AS
(
  SELECT 1 AS x
  UNION ALL
  SELECT 2
  UNION ALL
  SELECT 3
),

mean AS (
  SELECT AVG(x)::float8 AS mean
  FROM numbers
)

SELECT SUM(POWER(numbers.x - mean.mean, 2)) / (COUNT(x) - 1)
  AS variance
FROM numbers
CROSS JOIN mean
```

## What's next

  - Learn more about [Spanner's PostgreSQL language support](/spanner/docs/reference/postgresql/overview) .
