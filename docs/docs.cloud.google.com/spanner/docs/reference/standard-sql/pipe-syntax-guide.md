---
name: documents/docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax-guide
uri: https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax-guide
title: Work with pipe query syntax in GoogleSQL
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

Pipe query syntax is an extension to GoogleSQL that supports a linear query structure designed to make your queries easier to read, write, and maintain. You can use pipe syntax anywhere you write GoogleSQL.

Pipe syntax supports the same operations as existing [GoogleSQL query syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax) , or *standard syntax* —for instance, selection, aggregation and grouping, joining, and filtering—but the operations can be applied in any order, any number of times. The linear structure of pipe syntax lets you write queries so that the order of the query syntax matches the order of logical steps taken to build the result table.

Standard syntax suffers from issues that can make it difficult to read, write, and maintain. The following table shows how pipe syntax addresses these issues:

| Standard syntax                                                                                                | Pipe syntax                                                                                  |
| -------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Clauses must appear in a particular order.                                                                     | Pipe operators can be applied in any order.                                                  |
| More complex queries, such as queries with multi-level aggregation, usually require CTEs or nested subqueries. | More complex queries are usually expressed by adding pipe operators to the end of the query. |
| During aggregation, columns are repeated in the `SELECT` , `GROUP BY` , and `ORDER BY` clauses.                | Columns can be listed only once per aggregation.                                             |

For full syntax details, see the [Pipe query syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax) reference documentation.

## Basic syntax

In pipe syntax, queries start with a standard SQL query or a `FROM` clause. For example, a standalone `FROM` clause, such as `FROM MyTable` , is valid pipe syntax. The result of the standard SQL query or the table from the `FROM` clause can then be passed as input to a pipe symbol, `|>` , followed by a pipe operator name and any arguments to that operator. The pipe operator transforms the table in some way, and the result of that transformation can be passed to another pipe operator.

You can use any number of pipe operators in your query to do things such as select, order, filter, join, or aggregate columns. The names of pipe operators match their standard syntax counterparts and generally have the same behavior. The main difference between standard syntax and pipe syntax is the way you structure your query. As the logic expressed by your query becomes more complex, the query can still be expressed as a linear sequence of pipe operators, without using deeply nested subqueries, making it easier to read and understand.

Pipe syntax has the following key characteristics:

  - Each pipe operator in pipe syntax consists of the pipe symbol, `|>` , an operator name, and any arguments:  
    `|> operator_name argument_list`
  - Pipe operators can be added to the end of any valid query.
  - Pipe operators can be applied in any order, any number of times.
  - Pipe syntax works anywhere standard syntax is supported: in queries, views, table-valued functions, and other contexts.
  - Pipe syntax can be mixed with standard syntax in the same query. For example, subqueries can use different syntax from the parent query.
  - A pipe operator can see every alias that exists in the table preceding the pipe.
  - A query can [start with a `FROM` clause](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#from_queries) , and pipe operators can optionally be added after the `FROM` clause.

Consider the following table:

    CREATE TABLE Produce(
      item STRING(MAX),
      sales INT64,
      category STRING(MAX),
    ) PRIMARY KEY (item, sales);
    
    INSERT INTO Produce (item, sales, category) VALUES
      ('apples', 2, 'fruit'),
      ('apples', 7, 'fruit'),
      ('carrots', 0, 'vegetable'),
      ('bananas', 15, 'fruit');

The following queries each contain valid pipe syntax that shows how you can build a query sequentially.

Queries can [start with a `FROM` clause](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#from_queries) and don't need to contain a pipe symbol:

    -- View the table.
    FROM Produce;
    
    /*---------+-------+-----------+
     | item    | sales | category  |
     +---------+-------+-----------+
     | apples  | 2     | fruit     |
     | apples  | 7     | fruit     |
     | bananas | 15    | fruit     |
     | carrots | 0     | vegetable |
     +---------+-------+-----------*/

You can filter with a [`WHERE` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#where_pipe_operator) :

    -- Filter items with no sales.
    FROM Produce
    |> WHERE sales > 0;
    
    /*---------+-------+-----------+
     | item    | sales | category  |
     +---------+-------+-----------+
     | apples  | 2     | fruit     |
     | apples  | 7     | fruit     |
     | bananas | 15    | fruit     |
     +---------+-------+-----------*/

To perform aggregation, use the [`AGGREGATE` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#aggregate_pipe_operator) , followed by any number of aggregate functions, followed by a `GROUP BY` clause. The `GROUP BY` clause is part of the `AGGREGATE` pipe operator and isn't separated by a pipe symbol ( `|>` ).

    -- Compute total sales by item.
    FROM Produce
    |> WHERE sales > 0
    |> AGGREGATE SUM(sales) AS total_sales, COUNT(*) AS num_sales
       GROUP BY item;
    
    /*---------+-------------+-----------+
     | item    | total_sales | num_sales |
     +---------+-------------+-----------+
     | apples  | 9           | 2         |
     | bananas | 15          | 1         |
     +---------+-------------+-----------*/

Now suppose you have the following table that contains an ID for each item:

    CREATE TABLE ItemData(
      item STRING(MAX),
      id INT64,
    ) PRIMARY KEY (id);
    
    INSERT INTO ItemData (item, id) VALUES
      ('apples', 123),
      ('bananas', 456),
      ('carrots', 789);

You can use the [`JOIN` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#join_pipe_operator) to join the results of the previous query with this table to include each item's ID:

    FROM Produce
    |> WHERE sales > 0
    |> AGGREGATE SUM(sales) AS total_sales, COUNT(*) AS num_sales
       GROUP BY item
    |> JOIN ItemData USING(item);
    
    /*---------+-------------+-----------+-----+
     | item    | total_sales | num_sales | id  |
     +---------+-------------+-----------+-----+
     | apples  | 9           | 2         | 123 |
     | bananas | 15          | 1         | 456 |
     +---------+-------------+-----------+-----*/

## Key differences from standard syntax

Pipe syntax differs from standard syntax in the following ways:

  - Queries can [start with a `FROM` clause](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#from_queries) .
  - The `SELECT` pipe operator doesn't perform aggregation. You must use the [`AGGREGATE` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#aggregate_pipe_operator) instead.
  - Filtering is always done with the [`WHERE` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#where_pipe_operator) , which can be applied anywhere. The `WHERE` pipe operator, which replaces `HAVING` , can filter the results of aggregation functions.

For more details, see the complete list of [pipe operators](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#pipe_operators) .

## Additional features in pipe syntax

With few exceptions, pipe syntax supports all operators that standard syntax does with the same syntax. In addition, pipe syntax introduces additional pipe operators and uses a modified syntax for aggregations and joins. The following sections explain some of these operators. For all supported operators, see the complete list of [pipe operators](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#pipe_operators) .

### `EXTEND` pipe operator

The [`EXTEND` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#extend_pipe_operator) lets you append computed columns to the current table. The `EXTEND` pipe operator is similar to the `SELECT *, new_column` statement, but it gives you more flexibility in referencing column aliases.

Consider the following table that contains two test scores for each person:

    CREATE TABLE Scores(
      student STRING(MAX),
      score1 INT64,
      score2 INT64,
      points_possible INT64,
    ) PRIMARY KEY (student);
    
    INSERT INTO Scores (student, score1, score2, points_possible) VALUES
      ('Alex', 9, 10, 10),
      ('Dana', 5, 7, 10);
    
    /*---------+--------+--------+-----------------+
     | student | score1 | score2 | points_possible |
     +---------+--------+--------+-----------------+
     | Alex    | 9      | 10     | 10              |
     | Dana    | 5      | 7      | 10              |
     +---------+--------+--------+-----------------*/

Suppose you want to compute the average raw score and average percentage score that each student received on the test. In standard syntax, later columns in a `SELECT` statement don't have visibility to earlier aliases. To avoid a subquery, you have to repeat the expression for the average:

    SELECT student,
      (score1 + score2) / 2 AS average_score,
      (score1 + score2) / 2 / points_possible AS average_percent
    FROM Scores;

The `EXTEND` pipe operator can reference previously used aliases, making the query easier to read and less error prone:

    FROM Scores
    |> EXTEND (score1 + score2) / 2 AS average_score
    |> EXTEND average_score / points_possible AS average_percent
    |> SELECT student, average_score, average_percent;
    
    /*---------+---------------+-----------------+
     | student | average_score | average_percent |
     +---------+---------------+-----------------+
     | Alex    | 9.5           | .95             |
     | Dana    | 6.0           | 0.6             |
     +---------+---------------+-----------------*/

### `SET` pipe operator

The [`SET` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#set_pipe_operator) lets you replace the value of columns in the current table. The `SET` pipe operator is similar to the `SELECT * REPLACE (expression AS column)` statement. You can reference the original value by qualifying the column name with a table alias.

    FROM (SELECT 3 AS x, 5 AS y)
    |> SET x = 2 * x;
    
    /*---+---+
     | x | y |
     +---+---+
     | 6 | 5 |
     +---+---*/

### `DROP` pipe operator

The [`DROP` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#drop_pipe_operator) lets you remove columns from the current table. The `DROP` pipe operator is similar to the `SELECT * EXCEPT(column)` statement. After a column is dropped you can still reference the original value by qualifying the column name with a table alias.

    FROM (SELECT 1 AS x, 2 AS y) AS t
    |> DROP x;
    
    /*---+
     | y |
     +---+
     | 2 |
     +---*/

### `RENAME` pipe operator

The [`RENAME` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#rename_pipe_operator) lets you rename columns from the current table. The `RENAME` pipe operator is similar to the `SELECT * EXCEPT(old_column), old_column AS new_column` statement.

    FROM (SELECT 1 AS x, 2 AS y, 3 AS z) AS t
    |> RENAME y AS w;
    
    /*---+---+---+
     | x | w | z |
     +---+---+---+
     | 1 | 2 | 3 |
     +---+---+---*/

### `AGGREGATE` pipe operator

To perform aggregation in pipe syntax, use the [`AGGREGATE` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#aggregate_pipe_operator) , followed by any number of aggregate functions, followed by a `GROUP BY` clause. You don't need to repeat columns in a `SELECT` clause.

The examples in this section use the `Produce` table:

    CREATE TABLE Produce(
      item STRING(MAX),
      sales INT64,
      category STRING(MAX),
    ) PRIMARY KEY (item, sales);
    
    INSERT INTO Produce (item, sales, category) VALUES
      ('apples', 2, 'fruit'),
      ('apples', 7, 'fruit'),
      ('carrots', 0, 'vegetable'),
      ('bananas', 15, 'fruit');
    
    /*---------+-------+-----------+
     | item    | sales | category  |
     +---------+-------+-----------+
     | apples  |     7 | fruit     |
     | apples  |     7 | fruit     |
     | bananas |    15 | fruit     |
     | carrots |     0 | vegetable |
     +---------+-------+-----------*/

    FROM Produce
    |> AGGREGATE SUM(sales) AS total, COUNT(*) AS num_records
       GROUP BY item, category;
    
    /+---------+-----------+-------+-------------+
     | item    | category  | total | num_records |
     +---------+-----------+-------+-------------+
     | bananas | fruit     |    15 |           1 |
     | apples  | fruit     |     9 |           2 |
     | carrots | vegetable |     0 |           1 |
     +---------+-----------+-------+-------------+/

If you are ready to order your results immediately following aggregation, you can mark the columns in the `GROUP BY` clause that you want to order with `ASC` or `DESC` . Unmarked columns aren't ordered.

If you want to order all columns, then you can replace the `GROUP BY` clause with a [`GROUP AND ORDER BY` clause](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#shorthand_order_pipe_syntax) , which orders every column in ascending order by default. You can specify `DESC` following the columns that you want to order in descending order. For example, the following three queries are equivalent:

    -- Use a separate ORDER BY clause.
    FROM Produce
    |> AGGREGATE SUM(sales) AS total, COUNT(*) AS num_records
       GROUP BY category, item
    |> ORDER BY category DESC, item;

    -- Explicitly mark how to order columns in the GROUP BY clause.
    FROM Produce
    |> AGGREGATE SUM(sales) AS total, COUNT(*) AS num_records
       GROUP BY category DESC, item ASC;

    -- Only mark descending columns in the GROUP AND ORDER BY clause.
    FROM Produce
    |> AGGREGATE SUM(sales) AS total, COUNT(*) AS num_records
       GROUP AND ORDER BY category DESC, item;

The advantage of using a `GROUP AND ORDER BY` clause is that you don't have to repeat column names in two places.

To perform full table aggregation, use `GROUP BY()` or omit the `GROUP BY` clause entirely:

    FROM Produce
    |> AGGREGATE SUM(sales) AS total, COUNT(*) AS num_records;
    
    /*-------+-------------+
     | total | num_records |
     +-------+-------------+
     | 24    | 4           |
     +-------+-------------*/

### `JOIN` pipe operator

The [`JOIN` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#join_pipe_operator) lets you join the current table with another table and supports the standard [join operations](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax#join_types) , including `CROSS` , `INNER` , `LEFT` , `RIGHT` , and `FULL` .

The following examples reference the `Produce` and `ItemData` tables:

    CREATE TABLE Produce(
      item STRING(MAX),
      sales INT64,
      category STRING(MAX),
    ) PRIMARY KEY (item, sales);
    
    INSERT INTO Produce (item, sales, category) VALUES
      ('apples', 2, 'fruit'),
      ('apples', 7, 'fruit'),
      ('carrots', 0, 'vegetable'),
      ('bananas', 15, 'fruit');

    CREATE TABLE ItemData(
      item STRING(MAX),
      id INT64,
    ) PRIMARY KEY (id);
    
    INSERT INTO ItemData (item, id) VALUES
      ('apples', 123),
      ('bananas', 456),
      ('carrots', 789);

The following example uses a `USING` clause and avoids column ambiguity:

    FROM Produce
    |> JOIN ItemData USING(item)
    |> WHERE item = 'apples';
    
    /*--------+-------+----------+-----+
     | item   | sales | category | id  |
     +--------+-------+----------+-----+
     | apples | 2     | fruit    | 123 |
     | apples | 7     | fruit    | 123 |
     +--------+-------+----------+-----*/

To reference columns in the current table, such as to disambiguate columns in an `ON` clause, you need to alias the current table by using the [`AS` pipe operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax#as_pipe_operator) . You can optionally alias the joined table. You can reference both aliases following subsequent pipe operators:

    FROM Produce
    |> AS produce_table
    |> JOIN ItemData AS item_table
       ON produce_table.item = item_table.item
    |> WHERE produce_table.item = 'bananas'
    |> SELECT item_table.item, sales, id;
    
    /*---------+-------+-----+
     | item    | sales | id  |
     +---------+-------+-----+
     | bananas | 15    | 456 |
     +---------+-------+-----*/

The right-hand side of the join doesn't have visibility to the left-hand side of the join, which means you can't join the current table with itself. For example, the following query fails:

    -- This query doesn't work.
    FROM Produce
    |> AS produce_table
    |> JOIN produce_table AS produce_table_2 USING(item);

To perform a self-join with a modified table, you can use a common table expression (CTE) inside of a `WITH` clause.

    WITH cte_table AS (
      FROM Produce
      |> WHERE item = 'carrots'
    )
    FROM cte_table
    |> JOIN cte_table AS cte_table_2 USING(item);

## Example

Consider the following table with information about customer orders:

    CREATE TABLE CustomerOrders (
      customer_id INT64,
      order_id INT64,
      state STRING(2),
      cost INT64,
      item_type STRING(MAX),
    ) PRIMARY KEY (customer_id, order_id, state);
    
    INSERT INTO CustomerOrders (customer_id, order_id, state, cost, item_type) VALUES
      (1, 100, 'WA', 5,  'clothing'),
      (1, 101, 'WA', 20, 'clothing'),
      (1, 102, 'WA', 3,  'food'),
      (2, 103, 'NY', 16, 'clothing'),
      (2, 104, 'NY', 22, 'housewares'),
      (2, 104, 'WA', 45, 'clothing'),
      (3, 105, 'MI', 29, 'clothing');

Suppose you want to know, for each state and item type, the average amount spent by repeat customers. You could write the query in the following way:

    SELECT state, item_type, AVG(total_cost) AS average
    FROM (
      SELECT
        SUM(t1.cost) as total_cost,
        t1.customer_id,
        t1.state,
        t1.item_type
        FROM
          CustomerOrders as t1,
          ( SELECT c.customer_id, count(*) as num_orders
            FROM CustomerOrders c GROUP BY c.customer_id
          ) AS t2
        WHERE t1.customer_id = t2.customer_id
          AND t2.num_orders > 1
        GROUP BY t1.customer_id, t1.state, t1.item_type
      )
    GROUP BY state, item_type
    ORDER BY state DESC, item_type ASC;

If you read the query from top to bottom, you encounter the column `total_cost` before it has been defined. Even within the subquery, you read the names of columns before you see which table they come from.

To make sense of this query, it needs to be read from the inside out. The columns `state` and `item_type` are repeated numerous times in the `SELECT` and `GROUP BY` clauses, then again in the `ORDER BY` clause.

The following equivalent query is written using pipe syntax:

    FROM CustomerOrders
    |> AGGREGATE SUM(cost) as total_cost, GROUP BY customer_id, state, item_type
    |> JOIN (
      FROM CustomerOrders
      |> AGGREGATE COUNT(*) AS num_orders, GROUP BY customer_id
    ) USING (customer_id)
    |> WHERE num_orders > 1
    |> AGGREGATE AVG(total_cost) AS average GROUP BY state DESC, item_type ASC;
    
    /*-------+------------+---------+
     | state | item_type  | average |
     +-------+------------+---------+
     | WA    | clothing   | 35.0    |
     | WA    | food       | 3.0     |
     | NY    | clothing   | 16.0    |
     | NY    | housewares | 22.0    |
     +-------+------------+---------*/

With pipe syntax, you can write the query to follow the logical steps you might think through to solve the original problem. The lines of syntax in the query correspond to the following logical steps:

  - Start with the table of customer orders.
  - Find out how much each customer spent on each type of item by state.
  - Count the number of orders for each customer.
  - Restrict the results to repeat customers.
  - Find the average amount that repeat customers spend for each state and item type.

## What's next

  - [Pipe query syntax reference](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/pipe-syntax)
  - [Standard query syntax reference](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/query-syntax)
  - [VLDB 2024](https://research.google/pubs/sql-has-problems-we-can-fix-them-pipe-syntax-in-sql/) conference paper on pipe syntax
