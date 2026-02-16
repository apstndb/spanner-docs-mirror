## About subqueries

A subquery is a [query](/spanner/docs/reference/postgresql/query-syntax) that appears inside another query statement. Subqueries are also referred to as sub-selects or nested selects. The full `  SELECT  ` syntax is valid in subqueries.

The `  WITH  ` clause is not supported on a subquery. The following query returns an error:

``` text
SELECT username
FROM (
  WITH result AS (SELECT * FROM Players)
  SELECT *
  FROM result);
```

## Common tables used in examples

The following are tables that are used in the sample queries on this page:

``` text
Players
+-----------------------------+
| username  | level   | team  |
+-----------------------------+
| gorbie    | 29      | red   |
| junelyn   | 2       | blue  |
| corba     | 43      | green |
+-----------------------------+
```

``` text
Mascots
+-------------------+
| mascot   | team   |
+-------------------+
| cardinal | red    |
| parrot   | green  |
| finch    | blue   |
| sparrow  | yellow |
+-------------------+
```

## Expression subqueries

Expression subqueries are used in a query wherever expressions are valid. They return a single value, as opposed to a column or table. Expression subqueries can be [correlated](#correlated_subquery_concepts) .

### Scalar subqueries

``` text
SELECT ( subquery ) FROM table
```

**Description**

A subquery inside an expression is interpreted as a scalar subquery. Scalar subqueries are often used in the `  SELECT  ` list or `  WHERE  ` clause.

A scalar subquery is an ordinary SELECT query in parentheses that returns exactly one row with one column. The SELECT query is executed and the single returned value is used in the surrounding value expression. It is an error to use a query that returns more than one row or more than one column as a scalar subquery. (But if, during a particular execution, the subquery returns no rows, there is no error; the scalar result is taken to be null.) The subquery can refer to variables from the surrounding query, which will act as constants during any one evaluation of the subquery.

**Examples**

In this example, a correlated scalar subquery returns the mascots for a list of players, using the [`  Players  `](#example_tables) and [`  Mascots  `](#example_tables) tables:

``` text
SELECT
  username,
  (SELECT mascot FROM Mascots WHERE Players.team = Mascots.team) AS player_mascot
FROM
  Players;

+---------------------------+
| username  | player_mascot |
+---------------------------+
| gorbie    | cardinal      |
| junelyn   | finch         |
| corba     | parrot        |
+---------------------------+
```

In this example, an aggregate scalar subquery calculates `  avg_level  ` , the average level of a user in the [`  Players  `](#example_tables) table.

``` text
SELECT
  username,
  level,
  (SELECT AVG(level) FROM Players) AS avg_level
FROM
  Players;

+---------------------------------------+
| username  | level      | avg_level    |
+---------------------------------------+
| gorbie    | 29         | 24.66        |
| junelyn   | 2          | 24.66        |
| corba     | 43         | 24.66        |
+---------------------------------------+
```

### IN subqueries

``` text
SELECT value IN ( subquery )
```

**Description**

Returns TRUE if `  value  ` matches the select-list column value in any of the returned rows.

Returns FALSE if no equal row is found or the subquery returns zero rows.

Returns NULL if `  value  ` is NULL or if no equal row is found and the subquery returns at least one NULL row.

The subquery's SELECT list must have a single column of any type and its type must be comparable to the type for `  value  ` . If not, an error is returned.

If you prefer to use ANY/SOME syntax, these are equivalent:

``` text
value IN ( subquery )
value = ANY ( subquery )
value = SOME ( subquery )
```

Operators other than `  =  ` are not supported for ANY/SOME expressions.

**Examples**

In this example, the `  IN  ` operator that checks to see if a username called `  corba  ` exists within the [`  Players  `](#example_tables) table:

``` text
SELECT
  'corba' IN (SELECT username FROM Players) AS result;

+--------+
| result |
+--------+
| TRUE   |
+--------+
```

### NOT IN subqueries

``` text
SELECT value NOT IN ( subquery )
```

**Description**

Returns FALSE if `  value  ` does not match the select-list column value in any of the returned rows.

Returns TRUE if no equal row is found or the subquery returns zero rows.

Returns NULL if `  value  ` is NULL or if no equal row is found and the subquery returns at least one NULL row.

The subquery's SELECT list must have a single column of any type and its type must be comparable to the type for `  value  ` . If not, an error is returned.

If you prefer to use ALL syntax, these are equivalent:

``` text
value NOT IN ( subquery )
value != ALL ( subquery )
```

Operators other than `  !=  ` are not supported for ALL expressions.

### EXISTS subqueries

``` text
SELECT EXISTS( subquery )
```

**Description**

Returns TRUE if the subquery produces one or more rows. Returns FALSE if the subquery produces zero rows. Never returns `  NULL  ` . Unlike all other expression subqueries, there are no rules about the column list. Any number of columns may be selected and it will not affect the query result.

**Examples**

In this example, the `  EXISTS  ` operator checks to see if any rows are produced, using the [`  Players  `](#example_tables) table:

``` text
SELECT
  EXISTS(SELECT username FROM Players WHERE team = 'yellow') AS result;

+--------+
| result |
+--------+
| FALSE  |
+--------+
```

## Table subqueries

``` text
SELECT select-list FROM ( subquery ) [ [ AS ] alias ]
```

**Description**

With table subqueries, the outer query treats the result of the subquery as a table. You can only use these in the `  FROM  ` clause.

**Examples**

In this example, a subquery returns a table of usernames from the [`  Players  `](#example_tables) table:

``` text
SELECT results.username
FROM (SELECT * FROM Players) AS results;

+-----------+
| username  |
+-----------+
| gorbie    |
| junelyn   |
| corba     |
+-----------+
```

## Correlated subqueries

A correlated subquery is a subquery that references a column from outside that subquery. Correlation prevents reusing of the subquery result.

**Examples**

In this example, a list of mascots that don't have any players assigned to them is returned. The [`  Mascots  `](#example_tables) and [`  Players  `](#example_tables) tables are referenced.

``` text
SELECT mascot
FROM Mascots
WHERE
  NOT EXISTS(SELECT username FROM Players WHERE Mascots.team = Players.team);

+----------+
| mascot   |
+----------+
| sparrow  |
+----------+
```

In this example, a correlated scalar subquery returns the mascots for a list of players, using the [`  Players  `](#example_tables) and [`  Mascots  `](#example_tables) tables:

``` text
SELECT
  username,
  (SELECT mascot FROM Mascots WHERE Players.team = Mascots.team) AS player_mascot
FROM Players;

+---------------------------+
| username  | player_mascot |
+---------------------------+
| gorbie    | cardinal      |
| junelyn   | finch         |
| corba     | parrot        |
+---------------------------+
```
