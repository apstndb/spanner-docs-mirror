This page describes syntax and behavior for performing essential array management tasks for the PostgreSQL interface for Spanner. Arrays for the PostgreSQL interface share the same syntax as arrays in [open source PostgreSQL](https://www.postgresql.org/docs/current/arrays.html) , except as described in [Array type limitations](/spanner/docs/reference/postgresql/data-types#array_type_limitations) . One important limitation is no support for multidimensional arrays.

## Declaration of arrays

The following is an example of how to create a table that declares arrays:

``` text
CREATE TABLE lawn_care_business (
    client_name           text PRIMARY KEY,
    quarterly_fee         integer[],
    services_rendered     text[]
);
```

You name an array by adding square brackets ( `  []  ` ) to the data type name of the array elements. The previous statement creates a table named `  lawn_care_business  ` with two one-dimensional arrays. The first array, `  quarterly_fee  ` , is an `  integer  ` array. The second array, `  services_rendered  ` , is a `  text  ` array.

You can also specify the size of arrays when creating them:

``` text
CREATE TABLE lawn_care_business (
    client_name           text PRIMARY KEY,
    quarterly_fee         integer[4],
    services_rendered     text[3]
);
```

Note, however, that array size is not enforced. You can create an array with a specified size, but the size can be changed after the initial declaration.

#### Array keyword constructor syntax

The PostgreSQL interface also supports the ARRAY keyword constructor syntax, which lets you include expressions, add columns, and more.

The [native PostgreSQL array constructor syntax documentation](https://www.postgresql.org/docs/current/sql-expressions.html#SQL-SYNTAX-ARRAY-CONSTRUCTORS) provides details on using the syntax. The PostgreSQL interface supports this functionality, with the exception of multi-dimensional arrays.

The following command creates a simple table using the ARRAY syntax:

``` text
CREATE TABLE student_id_numbers (
    id                       integer PRIMARY KEY,
    student_phone_numbers    integer ARRAY[]
);
```

## Input values into array columns

A PostgreSQL interface array can only store values of one PostgreSQL type. For a list of supported PostgreSQL interface data types, see [PostgreSQL data types](/spanner/docs/reference/postgresql/data-types) . Nested arrays are not supported.

The standard array format for inputting values into arrays for PostgreSQL looks like this:

Data type

Format

PostgreSQL example

`  integer  `

`  '{value1, value2, value3, value4}'  `

`  INSERT INTO lawn_care_business VALUES ('Bagdan', '{1000, 1000, 1000, 1000}', '{"mowing", "fertilizing"}'); INSERT INTO lawn_care_business VALUES ('Esmae', '{2000, 2500, 2500, 2500}', '{"mowing", "fertilizing", "weeding"}');  `

`  string  `

`  '{"text1", "text2"}'  `

When inputting values using this format you should be aware of the following caveats:

  - You can put double quotes around any value, even integers.
  - You must put double quotes around a string if it contains a comma or curly brace.
  - To enter a `  NULL  ` value, enter either `  null  ` or `  NULL  ` . If you want a string that merely says NULL, enter "NULL".

You can also use the `  ARRAY  ` constructor syntax to input values into an array:

Data type

Format

PostgreSQL example

`  integer  `

`  ARRAY[value1, value2, value3, value4]  `

`  INSERT INTO lawn_care_business VALUES ('Bagdan', ARRAY[1000, 1000, 1000, 1000], ARRAY['mowing', 'fertilizing']); INSERT INTO lawn_care_business VALUES ('Esmae', ARRAY[2000, 2500, 2500, 2500], ARRAY['mowing', 'fertilizing', 'weeding']);  `

`  string  `

`  ARRAY['text1', 'text2']  `

## Access array values

You can run queries on arrays in a table. Continuing the previous example, the following query returns the names of clients who were charged a different fee between the first and second quarters of the year:

``` text
SELECT client_name FROM lawn_care_business WHERE quarterly_fee[1] <> quarterly_fee[2];
```

Result:

``` text
 client_name
-------------
 Esmae
```

PostgreSQL arrays are 1-based, meaning that for an array of size **n** , the first element is `  array[1]  ` and the last element is at `  array[n]  ` .

The following query gets the third quarter fee for all clients:

``` text
SELECT quarterly_fee[3] FROM lawn_care_business;
```

Result:

``` text
 quarterly_fee
---------------
 1000
 2500
```

## Modify array values

To modify the values of an array, you must provide the values for each element in the array. For example:

``` text
UPDATE lawn_care_business SET quarterly_fee = '{2500,2500,2800,2800}'
    WHERE client_name = 'Esmae';
```

The following example updates the same information using `  ARRAY  ` expression syntax:

``` text
UPDATE lawn_care_business SET quarterly_fee = ARRAY[2500,2500,2800,2800]
    WHERE client_name = 'Esmae';
```

You cannot currently update specific values of an array. This includes appending elements to an array at an unused index. For example, the following command is not supported:

``` text
UPDATE lawn_care_business SET services_rendered[4] = 'reseeding'
    WHERE client_name = 'Bagdan';
```

Instead, if you wish to add, remove, or the modify contents of an array, include the entire array in the query:

``` text
UPDATE lawn_care_business SET services_rendered = '{"mowing", "fertilizing", "weeding", "reseeding"}'
    WHERE client_name = 'Bagdan';
```

## Search for values in arrays

Each value must be checked when searching for a value in an array. If you know the size of the array, you can do this manually:

``` text
SELECT * FROM lawn_care_business WHERE quarterly_fee[1] = 1000 OR
                                       quarterly_fee[2] = 1000 OR
                                       quarterly_fee[3] = 1000 OR
                                       quarterly_fee[4] = 1000;
```

**Note:** we advise against searching an array for specific array elements. Doing so may indicate suboptimal database design. Instead, consider using a separate table with a row for each array element. Doing so makes searching easier, and helps when you have many elements to search through.

## Finding lengths

The `  array_length  ` function returns the length of an array.

``` text
SELECT some_numbers,
array_length(some_numbers, 1) AS len
FROM
  (SELECT ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers
   UNION ALL SELECT ARRAY[2, 4, 8, 16, 32] AS some_numbers
   UNION ALL SELECT ARRAY[5, 10] AS some_numbers) AS sequences;

/*--------------------+--------*
 | some_numbers       | len    |
 +--------------------+--------+
 | [0, 1, 1, 2, 3, 5] | 6      |
 | [2, 4, 8, 16, 32]  | 5      |
 | [5, 10]            | 2      |
 *--------------------+--------*/
```

## Converting elements in an array to rows in a table

To convert an `  ARRAY  ` into a set of rows, also known as flattening, use the [`  UNNEST  `](/spanner/docs/reference/postgresql/query-syntax#unnest_operator) operator. `  UNNEST  ` takes an `  ARRAY  ` and returns a table with a single row for each element in the `  ARRAY  ` .

Because `  UNNEST  ` rearranges the order of the `  ARRAY  ` elements, you might want to restore order to the table. To do so, use the optional `  WITH ORDINALITY  ` clause to return an additional column with the index for each array element, then use the `  ORDER BY  ` clause to order the rows by their offset.

#### Example

``` text
SELECT *
FROM UNNEST(ARRAY['foo', 'bar', 'baz', 'qux', 'corge', 'garply', 'waldo', 'fred'])
WITH ORDINALITY AS my_table(element, ordinality)
ORDER BY ordinality;

 /-----------------------*
 | element  | ordinality |
 +----------+------------+
 | foo      | 1          |
 | bar      | 2          |
 | baz      | 3          |
 | qux      | 4          |
 | corge    | 5          |
 | garply   | 6          |
 | waldo    | 7          |
 | fred     | 8          |
 ----------/----------- */
```

## Creating arrays from subqueries

You can convert a subquery result into an array using the [`  ARRAY()  `](/spanner/docs/reference/postgresql/functions-and-operators#array-functions) function.

#### Example

``` text
  SELECT some_numbers,
    ARRAY(SELECT x * 2
      FROM UNNEST(some_numbers) AS X) AS doubled
    FROM (
      SELECT ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers
      UNION ALL SELECT ARRAY[2, 4, 8, 16, 32] AS some_numbers
      UNION ALL SELECT ARRAY[5, 10] AS some_numbers) AS sequences;

/*--------------------+---------------------*
 | some_numbers       | doubled             |
 +--------------------+---------------------+
 | [0, 1, 1, 2, 3, 5] | [0, 2, 2, 4, 6, 10] |
 | [2, 4, 8, 16, 32]  | [4, 8, 16, 32, 64]  |
 | [5, 10]            | [10, 20]            |
 *--------------------+---------------------*/
```

The suquery called sequences in the example contains a column, `  some_numbers  ` , of type `  bigint[]  ` . The query contains another subquery that selects each row in the `  some_numbers  ` column and uses [`  UNNEST  `](/spanner/docs/reference/postgresql/query-syntax#unnest_operator) to return the array as a set of rows. Then, it multiplies each value by two, and re-combines the rows into an array using the `  ARRAY()  ` operator.

## Filtering arrays

The following examples use subqueries and the `  WHERE  ` clause to filter an array in the query.

``` text
SELECT
  ARRAY(SELECT x * 2
        FROM UNNEST(some_numbers) AS x
        WHERE x < 5) AS doubled_less_than_five
FROM (SELECT ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers
   UNION ALL SELECT ARRAY[2, 4, 8, 16, 32] AS some_numbers
   UNION ALL SELECT ARRAY[5, 10] AS some_numbers) sequences;

/*------------------------*
 | doubled_less_than_five |
 +------------------------+
 | [0, 2, 2, 4, 6]        |
 | [4, 8]                 |
 | []                     |
 *------------------------*/
```

Notice that the third row contains an empty array, because the elements in the corresponding original row ( `  [5, 10]  ` ) did not meet the filter requirement of `  x < 5  ` .

You can also filter arrays by using `  SELECT DISTINCT  ` to return only unique elements within an array.

``` text
SELECT ARRAY(
    SELECT DISTINCT x
      FROM UNNEST(some_numbers) AS x) AS unique_numbers
FROM (
  SELECT ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers) AS sequences;

/*-----------------*
 | unique_numbers  |
 +-----------------+
 | [0, 1, 2, 3, 5] |
 *-----------------*/
```

## Scanning arrays

To check if an array contains a specific value, use the [`  ANY  ` / `  SOME  `](/spanner/docs/reference/postgresql/functions-and-operators#array_comparisons) clause. To check if an array contains a value matching a condition, use either the `  ALL  ` clause or `  EXISTS  ` operator with [`  UNNEST  `](/spanner/docs/reference/postgresql/query-syntax#unnest_operator) .

### Scanning for specific values

To scan an array for a specific value, use the `  ANY  ` / `  SOME  ` clause.

The following example returns `  true  ` if the array contains the number `  2  ` .

``` text
SELECT 2 = ANY(ARRAY[0, 1, 1, 2, 3, 5]) AS contains_value;

/*----------------*
 | contains_value |
 +----------------+
 | true           |
 *----------------*/
```

To return the rows of a table where the array column contains a specific value, filter the results of `  ANY  ` / `  SOME  ` using the `  WHERE  ` clause.

#### Example

The following example returns the `  id  ` value for the rows where the array column contains the value `  2  ` .

``` text
SELECT id AS matching_rows
FROM  (
    SELECT 1 AS id, ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers
    UNION ALL SELECT 2 AS id, ARRAY[2, 4, 8, 16, 32] AS some_numbers
    UNION ALL SELECT 3 AS id, ARRAY[5, 10] AS some_numbers
) AS sequences
WHERE 2 = ANY(sequences.some_numbers)
ORDER BY matching_rows;

/*---------------*
 | matching_rows |
 +---------------+
 | 1             |
 | 2             |
 *---------------*/
```

### Scanning for values that satisfy a condition

To scan an array for values that match a condition, use `  UNNEST  ` with subqueries to return a table of the elements in the array, use `  WHERE  ` to filter the resulting table in the subquery, and use `  EXISTS  ` to check if the filtered table contains any rows.

#### Example

The following example returns the `  id  ` value for rows where the array contains values greater than `  5  ` .

``` text
SELECT id AS matching_rows
FROM (
    SELECT 1 AS id, ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers
    UNION ALL
    SELECT 2 AS id, ARRAY[2, 4, 8, 16, 32] AS some_numbers
    UNION ALL
    SELECT 3 AS id, ARRAY[5, 10] AS some_numbers
  ) AS sequences
WHERE EXISTS(SELECT * FROM UNNEST(some_numbers) AS x WHERE x > 5);

/*---------------*
 | matching_rows |
 +---------------+
 | 2             |
 | 3             |
 *---------------*/
```

## Arrays and aggregation

You can aggregate values into an array using `  ARRAY_AGG()  ` .

``` text
SELECT ARRAY_AGG(fruit) AS fruit_basket
FROM (
  SELECT 'apple' AS fruit
   UNION ALL SELECT 'pear' AS fruit
   UNION ALL SELECT 'banana' AS fruit) AS fruits;

/*-----------------------*
 | fruit_basket          |
 +-----------------------+
 | [apple, pear, banana] |
 *-----------------------*/
```

The array returned by `  ARRAY_AGG()  ` is in an arbitrary order, since the order in which the function concatenates values is not guaranteed.

You can also apply aggregate functions such as `  SUM()  ` to the elements in an array. For example, the following query returns the sum of elements for each row of the subquery result.

``` text
SELECT some_numbers,
  (SELECT SUM(x)
   FROM UNNEST(s.some_numbers) AS x) AS sums
FROM (
  SELECT ARRAY[0, 1, 1, 2, 3, 5] AS some_numbers
   UNION ALL SELECT ARRAY[2, 4, 8, 16, 32] AS some_numbers
   UNION ALL SELECT ARRAY[5, 10] AS some_numbers) AS s;

/*--------------------+------*
 | some_numbers       | sums |
 +--------------------+------+
 | [0, 1, 1, 2, 3, 5] | 12   |
 | [2, 4, 8, 16, 32]  | 62   |
 | [5, 10]            | 15   |
 *--------------------+------*/
```

## Converting arrays to strings

The `  array_to_string()  ` function lets you convert a text array to a single text value where the resulting value is the ordered concatenation of the array elements.

The second argument is the separator that the function inserts between inputs to produce the output; this second argument must use the same type as the elements of the first argument.

#### Example

``` text
SELECT ARRAY_TO_STRING(greeting, ' ') AS greetings
FROM (
  SELECT ARRAY['Hello', 'World'] AS greeting
) AS words;

/*-------------*
 | greetings   |
 +-------------+
 | Hello World |
 *-------------*/
```

The optional third argument takes the place of `  NULL  ` values in the input array.

  - If you omit this argument, then the function ignores `  NULL  ` array elements.
  - If you provide an empty string, the function inserts the separator specified in the second argument for `  NULL  ` array elements.

#### Example

``` text
SELECT
  ARRAY_TO_STRING(arr, '.', 'N') AS non_empty_string,
  ARRAY_TO_STRING(arr, '.', '') AS empty_string,
  ARRAY_TO_STRING(arr, '.') AS omitted
FROM (SELECT ARRAY['a', NULL, 'b', NULL, 'c', NULL] AS arr) AS subquery;

/*------------------+--------------+---------*
 | non_empty_string | empty_string | omitted |
 +------------------+--------------+---------+
 | a.N.b.N.c.N      | a..b..c.     | a.b.c   |
 *------------------+--------------+---------*/
```

## Combining arrays

In some cases, you might want to combine multiple arrays into a single array. You can accomplish this using the `  ||  ` operator.

``` text
SELECT ARRAY[1, 2] || ARRAY[3, 4] || ARRAY[5, 6] AS count_to_six;

/*--------------------------------------------------*
 | count_to_six                                     |
 +--------------------------------------------------+
 | [1, 2, 3, 4, 5, 6]                               |
 *--------------------------------------------------*/
```
