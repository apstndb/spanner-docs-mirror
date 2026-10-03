---
name: documents/docs.cloud.google.com/spanner/docs/reference/dialect-differences
uri: https://docs.cloud.google.com/spanner/docs/reference/dialect-differences
title: Dialect parity between GoogleSQL and PostgreSQL
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

This page describes the dialect differences between GoogleSQL and PostgreSQL and offers recommendations for using PostgreSQL approaches for specific GoogleSQL features.

## GoogleSQL dialect feature differences

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>GoogleSQL feature</th>
<th>PostgreSQL dialect recommendation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax">Pipe syntax</a></td>
<td>No recommendation available, Pipe syntax is GoogleSQL-only.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/bigquery/docs/spanner-external-datasets">BigQuery external datasets</a></td>
<td>Use <a href="https://docs.cloud.google.com/bigquery/docs/spanner-federated-queries">Spanner federated queries</a> . When you run a federated query against a PostgreSQL-dialect database using the <a href="https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/federated_query_functions#external_query"><code>EXTERNAL_QUERY</code></a> function, you must write the query in PostgreSQL syntax.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#enum_type"><code>ENUM</code></a></td>
<td>Use <code>TEXT</code> columns with checked constraints instead. Unlike <code>ENUMS</code> , the sort order of a <code>TEXT</code> column can't be user-defined. The following example restricts the column to only support the <code>'C'</code> , <code>'B'</code> , and <code>'A'</code> values.
<pre class="sql"><code>CREATE TABLE singers (
 singer_id BIGINT PRIMARY KEY,
 type TEXT NOT NULL CHECK (type IN (&#39;C&#39;, &#39;B&#39;, &#39;A&#39;))
);
       </code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#group_hints"><code>GROUP_METHOD</code> hint</a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/graph/overview">Graph</a></td>
<td>No recommendation available.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate-function-calls#aggregate_function_call_syntax"><code>HAVING MAX</code> or <code>HAVING MIN</code></a></td>
<td>Use a <code>JOIN</code> or a subquery to filter for the <code>MAX</code> or <code>MIN</code> value for the aggregation. The following example requires filtering <code>MAX</code> or <code>MIN</code> in a subquery.
<pre class="sql"><code>WITH amount_per_year AS (
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
WHERE year = (SELECT MAX(year) FROM amount_per_year);</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/foreign-keys/overview#use-informational-foreign-keys">Informational foreign keys</a></td>
<td>No recommendation available.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#json_type"><code>JSON</code> data type</a></td>
<td>Use the <a href="https://docs.cloud.google.com/spanner/docs/reference/postgresql/data-types"><code>JSONB</code> data type.</a></td>
</tr>
<tr class="odd">
<td><code>SELECT </code><a href="https://cloud.google.com/spanner/docs/reference/standard-sql/json_functions#to_json"><code>to_json(table)</code></a><code> FROM table</code></td>
<td>We recommend explicitly mapping each column with the <code>jsonb_build_object</code> function:
<pre class="sql"><code>WITH singers AS (
  SELECT 1::int8 AS id, &#39;Singer First Name&#39;::text AS first_name
)

SELECT jsonb_build_object(&#39;id&#39;, id, &#39;first_name&#39;, first_name)
FROM singers;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/collation-concepts#collate_about"><code>ORDER BY … COLLATE …</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><code>NUMERIC</code> column as a primary key, secondary index, or foreign key</td>
<td>We recommend using an index over a <code>TEXT</code> generated column, as shown in the following example:
<pre class="sql"><code>CREATE TABLE singers(
 id numeric NOT NULL,
 pk text GENERATED ALWAYS AS (id::text) STORED,
 PRIMARY KEY(pk)
);</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#protocol-buffers">Protocol buffer</a> data type</td>
<td>You can store serialized protocol buffers as the PostgreSQL <a href="https://docs.cloud.google.com/spanner/docs/reference/postgresql/data-types#supported"><code>BYTEA</code><code> data type</code></a> .</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/schema-design#ordering_timestamp-based_keys"><code>PRIMARY KEY DESC</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#select_as_value"><code>SELECT AS VALUE</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#select_except"><code>SELECT * EXCEPT</code></a></td>
<td>We recommend that you spell out all columns in the <code>SELECT</code> statement.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#select_replace"><code>SELECT * REPLACE</code></a></td>
<td>We recommend that you spell out all columns in the <code>SELECT</code> statement.</td>
</tr>
<tr class="odd">
<td>The following columns in the <code>SPANNER_SYS</code> statistics tables:
<ul>
<li><a href="https://docs.cloud.google.com/spanner/docs/introspection/transaction-statistics">Transaction statistics</a> : <code>TOTAL_LATENCY_DISTRIBUTION</code> and <code>OPERATIONS_BY_TABLE</code></li>
<li><a href="https://docs.cloud.google.com/spanner/docs/introspection/query-statistics">Query statistics</a> : <code>LATENCY_DISTRIBUTION</code></li>
<li><a href="https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics">Lock Statistics</a> : <code>SAMPLE_LOCK_REQUESTS</code></li>
</ul></td>
<td>We recommend using the following JSON-compatible string representation columns instead:
<ul>
<li><a href="https://docs.cloud.google.com/spanner/docs/introspection/transaction-statistics">Transaction statistics</a> : <code>TOTAL_LATENCY_DISTRIBUTION_JSON_STRING</code> and <code>OPERATIONS_BY_TABLE_JSON_STRING</code></li>
<li><a href="https://docs.cloud.google.com/spanner/docs/introspection/query-statistics">Query statistics</a> : <code>LATENCY_DISTRIBUTION_JSON_STRING</code></li>
<li><a href="https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics">Lock Statistics</a> : <code>SAMPLE_LOCK_REQUESTS_JSON_STRING</code></li>
</ul></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/subqueries#in_subquery_concepts"><code>VALUE IN UNNEST(ARRAY(...))</code></a></td>
<td>Use the equality operator with the <code>ANY</code> function, as shown in the following example:
<pre class="sql"><code>SELECT value = any(array[...])</code></pre></td>
</tr>
</tbody>
</table>

## GoogleSQL dialect function differences

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>GoogleSQL function</th>
<th>PostgreSQL dialect recommendation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#acosh"><code>ACOSH</code></a></td>
<td>Use the formula of the function explicitly, as shown in the following example:<br />

<pre class="sql"><code>SELECT LN(x + SQRT(x*x - 1));</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance"><code>APPROX_COSINE_DISTANCE</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product"><code>APPROX_DOT_PRODUCT</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance"><code>APPROX_EUCLIDEAN_DISTANCE</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions#any_value"><code>ANY_VALUE</code></a></td>
<td>Workaround available outside of aggregation and <code>GROUP BY</code> . Use a subquery with the <code>ORDER BY</code> or <code>LIMIT</code> clauses, as shown in the following example:
<pre class="sql"><code>SELECT * FROM
(
  (expression)
  UNION ALL SELECT NULL, … -- as many columns as you have
) AS rows
ORDER BY 1 NULLS LAST
LIMIT 1;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions#array_concat_agg"><code>ARRAY_CONCAT_AGG</code></a></td>
<td>You can use <code>ARRAY_AGG</code> and <code>UNNEST</code> as shown in the following example:
<pre class="sql"><code>WITH albums AS
(
  SELECT ARRAY[&#39;Song A&#39;, NULL, &#39;Song B&#39;] AS songs
  UNION ALL
  SELECT NULL
  UNION ALL
  SELECT ARRAY[]::TEXT[]
)
SELECT ARRAY_AGG(song) FROM albums, UNNEST(songs) song;
      </code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_first"><code>ARRAY_FIRST</code></a></td>
<td>Use the array subscript operator, as shown in the following example:
<pre class="sql"><code>SELECT array_expression[1];</code></pre>
Note that this will return <code>NULL</code> for empty arrays.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes"><code>ARRAY_INCLUDES</code></a></td>
<td>Use the equality operator with the <code>ANY</code> function, as shown in the following example:
<pre class="sql"><code>SELECT search_value = ANY(array_to_search);</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes_all"><code>ARRAY_INCLUDES_ALL</code></a></td>
<td>Use the array contains operator, as shown in the following example:<br />

<pre class="sql"><code>SELECT array_to_search @&gt; search_values;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes_any"><code>ARRAY_INCLUDES_ANY</code></a></td>
<td>Use the array overlap operator, as shown in the following example:<br />

<pre class="sql"><code>SELECT array_to_search &amp;&amp; search_values;</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_is_distinct"><code>ARRAY_IS_DISTINCT</code></a></td>
<td>Use a subquery to count distinct values and compare them to the original array length, as shown in the following example:<br />

<pre class="sql"><code>SELECT ARRAY_LENGTH(value, 1) = (
SELECT COUNT(DISTINCT e)
FROM UNNEST(value) AS e);</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_last"><code>ARRAY_LAST</code></a></td>
<td>Use the array subscript operator, as shown in the following example
<pre class="sql"><code>SELECT (value)[ARRAY_LENGTH(value, 1)];
      </code></pre>
This returns <code>NULL</code> for empty arrays.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_max"><code>ARRAY_MAX</code></a></td>
<td>Use a subquery with <code>UNNEST</code> and the <code>MAX</code> function, as shown in the following example:
<pre class="sql"><code>SELECT MAX(e) FROM UNNEST(value) AS e;
      </code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_min"><code>ARRAY_MIN</code></a></td>
<td>Use a subquery with <code>UNNEST</code> and the <code>MIN</code> function, as shown in the following example:
<pre class="sql"><code>SELECT MIN(e) FROM UNNEST(value) AS e;
      </code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_reverse"><code>ARRAY_REVERSE</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#asinh"><code>ASINH</code></a></td>
<td>Use the formula of the function explicitly, as shown in the following example:<br />

<pre class="sql"><code>SELECT LN(x + SQRT(x*x - 1));</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#atanh"><code>ATANH</code></a></td>
<td>Use the formula of the function explicitly, as shown in the following example:<br />

<pre class="sql"><code>SELECT 0.5 * LN((1 + x) / (1 - x));</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/bit_functions#bit_count"><code>BIT_COUNT</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate_functions#bit_xor"><code>BIT_XOR</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#byte_length"><code>BYTE_LENGTH</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#code_points_to_bytes"><code>CODE_POINTS_TO_BYTES</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#code_points_to_string"><code>CODE_POINTS_TO_STRING</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#cosh"><code>COSH</code></a></td>
<td>Use the formula of the function explicitly, as shown in the following example:<br />

<pre class="sql"><code>SELECT (EXP(x) + EXP(-x)) / 2;
      </code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/debugging_functions#error"><code>ERROR</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#from_base32"><code>FROM_BASE32</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#from_base64"><code>FROM_BASE64</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#from_hex"><code>FROM_HEX</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#generate_array"><code>GENERATE_ARRAY</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#generate_date_array"><code>GENERATE_DATE_ARRAY</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#nethost"><code>NET.HOST</code></a></td>
<td>Use a regular expression and the <code>substring</code> function, as shown in the following example:
<pre class="sql"><code>/* Use modified regular expression from
  https://tools.ietf.org/html/rfc3986#appendix-A. */

SELECT Substring(&#39;http://www.google.com/test&#39; FROM
  &#39;^(?:[^:/?#]+:)?(?://)?([^/?#]*)?[^?#]*(?:\\?[^#]*)?(?:#.*)?&#39;)</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netip_from_string"><code>NET.IP_FROM_STRING</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netip_net_mask"><code>NET.IP_NET_MASK</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netip_to_string"><code>NET.IP_TO_STRING</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netip_trunc"><code>NET.IP_TRUNC</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netipv4_from_int64"><code>NET.IPV4_FROM_INT64</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netipv4_to_int64"><code>NET.IPV4_TO_INT64</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netpublic_suffix"><code>NET.PUBLIC_SUFFIX</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netreg_domain"><code>NET.REG_DOMAIN</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/net_functions#netsafe_ip_from_string"><code>NET.SAFE_IP_FROM_STRING</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#normalize"><code>NORMALIZE</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#normalize_and_casefold"><code>NORMALIZE_AND_CASEFOLD</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#regexp_extract_all"><code>REGEXP_EXTRACT_ALL</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_add"><code>SAFE.ADD</code></a></td>
<td>We recommend that you protect against an overflow explicitly leveraging the <code>NUMERIC</code> data type.
<pre class="sql"><code>WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 1, 2
)

SELECT
 CASE
   WHEN a::numeric + b::numeric &gt; 9223372036854775807 THEN NULL
   WHEN a + b &lt; -9223372036854775808 THEN NULL
   ELSE a + b
 END AS result
FROM numbers;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_functions#safe_casting"><code>SAFE.CAST</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#safe_convert_bytes_to_string"><code>SAFE.CONVERT_BYTES_TO_STRING</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_divide"><code>SAFE.DIVIDE</code></a></td>
<td>We recommend that you protect against an overflow explicitly leveraging the <code>NUMERIC</code> data type during a division operation.
<pre class="sql"><code>WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 10, 2
)

SELECT
 CASE
   WHEN b = 0 THEN NULL
   WHEN a::numeric / b::numeric &gt; 9223372036854775807 THEN NULL
   WHEN a::numeric / b::numeric &lt; -9223372036854775808 THEN NULL
   ELSE a / b
 END AS result
FROM numbers;</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_multiply"><code>SAFE.MULTIPLY</code></a></td>
<td>We recommend that you protect against an overflow explicitly leveraging the <code>NUMERIC</code> data type during a multiplication operation.
<pre class="sql"><code>WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 1, 2
)

SELECT
 CASE
   WHEN a::numeric * b::numeric &gt; 9223372036854775807 THEN NULL
   WHEN a::numeric * b::numeric &lt; -9223372036854775808 THEN NULL
   ELSE a * b
 END AS result
FROM numbers;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_negate"><code>SAFE.NEGATE</code></a></td>
<td>We recommend that you protect against an overflow explicitly leveraging the <code>NUMERIC</code> data type during a negation operation.
<pre class="sql"><code>WITH numbers AS
(
  SELECT 9223372036854775807 AS a
  UNION ALL
  SELECT -9223372036854775808
)

SELECT
 CASE
   WHEN a &lt;= -9223372036854775808 THEN NULL
   WHEN a &gt;= 9223372036854775809 THEN NULL
   ELSE -a
 END AS result
FROM numbers;</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#safe_subtract"><code>SAFE.SUBTRACT</code></a></td>
<td>We recommend that you protect against an overflow explicitly leveraging the <code>NUMERIC</code> data type during a subtraction operation.
<pre class="sql"><code>WITH numbers AS
(
  SELECT 1::int8 AS a, 9223372036854775807::int8 AS b
  UNION ALL
  SELECT 1, 2
)

SELECT
 CASE
   WHEN a::numeric - b::numeric &gt; 9223372036854775807 THEN NULL
   WHEN a::numeric - b::numeric &lt; -9223372036854775808 THEN NULL
   ELSE a - b
 END AS result
FROM numbers;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#safe_to_json"><code>SAFE.TO_JSON</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#sinh"><code>SINH</code></a></td>
<td>Use the formula of the function explicitly, as shown in the following example:<br />

<pre class="sql"><code>SELECT (EXP(x) - EXP(-x)) / 2;</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#split"><code>SPLIT</code></a></td>
<td>Use the <code>regexp_split_to_array</code> function, as shown in the following example:
<pre class="sql"><code>WITH letters AS
(
  SELECT &#39;&#39; as letter_group
  UNION ALL
  SELECT &#39;a&#39; as letter_group
  UNION ALL
  SELECT &#39;b c d&#39; as letter_group
)

SELECT regexp_split_to_array(letter_group, &#39; &#39;) as example
FROM letters;</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev"><code>STDDEV</code></a></td>
<td>Use the formula of the function explicitly (unbiased standard deviation), as shown in the following example:<br />

<pre class="sql"><code>WITH numbers AS
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
CROSS JOIN mean</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev_samp"><code>STDDEV_SAMP</code></a></td>
<td>Use the formula of the function explicitly (unbiased standard deviation), as shown in the following example:<br />

<pre class="sql"><code>WITH numbers AS
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
      </code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#tanh"><code>TANH</code></a></td>
<td>Use the formula of the function explicitly.<br />

<pre class="sql"><code>SELECT (EXP(x) - EXP(-x)) / (EXP(x) + EXP(-x));</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_micros"><code>TIMESTAMP_MICROS</code></a></td>
<td>Use the <code>to_timestamp</code> function and truncate the microseconds part of the input (precision loss), as shown in the following example:
<pre class="sql"><code>SELECT to_timestamp(1230219000123456 / 1000000);</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#timestamp_millis"><code>TIMESTAMP_MILLIS</code></a></td>
<td>Use the <code>to_timestamp</code> function and truncate the milliseconds part of the input (precision loss), as shown in the following example:
<pre class="sql"><code>SELECT to_timestamp(1230219000123 / 1000);</code></pre></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_base32"><code>TO_BASE32</code></a></td>
<td>No recommendation available.</td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_base64"><code>TO_BASE64</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_code_points"><code>TO_CODE_POINTS</code></a></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#to_hex"><code>TO_HEX</code></a></td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#var_samp"><code>VAR_SAMP</code></a></td>
<td>Use the formula of the function explicitly (unbiased variance), as shown in the following:<br />

<pre class="sql"><code>-- Use formula directly (unbiased)

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
CROSS JOIN mean</code></pre></td>
</tr>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#variance"><code>VARIANCE</code></a></td>
<td>Use the formula of the function explicitly (unbiased variance), as shown in the following example:<br />

<pre class="sql"><code>-- Use formula directly (unbiased VARIANCE like VAR_SAMP)

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
CROSS JOIN mean</code></pre></td>
</tr>
</tbody>
</table>

## What's next

- Learn more about [Spanner's PostgreSQL language support](https://docs.cloud.google.com/spanner/docs/reference/postgresql/overview) .
