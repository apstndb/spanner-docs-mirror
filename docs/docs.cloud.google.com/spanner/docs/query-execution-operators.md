This page describes the operators used in Spanner [query execution plans](https://docs.cloud.google.com/spanner/docs/query-execution-plans) . For information about how to retrieve an execution plan for a specific query using the Google Cloud console, see [Understanding how Spanner executes queries](https://docs.cloud.google.com/spanner/docs/sql-best-practices#how-execute-queries) .

Execution plans support GoogleSQL-dialect databases and PostgreSQL-dialect databases.

> **PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases.

## Mapping SQL constructs to query execution operators

The exact mapping between SQL constructs and query execution operators depends on the query optimization. The following table shows some common mappings:

| **SQL**                             | **Query execution operator**                                                                                                                                                            |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Table reference                     | [Table Scan, Index Scan](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#scan)                                                                                          |
| `WHERE`                             | [Filter Scan](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#filter_scan) , [Filter](https://docs.cloud.google.com/spanner/docs/query-operators-unary#filter)          |
| `GROUP BY`                          | [Aggregate](https://docs.cloud.google.com/spanner/docs/query-operators-unary#aggregate)                                                                                                 |
| Scalar function (such as `ISNULL` ) | [Compute](https://docs.cloud.google.com/spanner/docs/query-operators-unary#compute)                                                                                                     |
| Aggregate function (such as `SUM` ) | [Aggregate](https://docs.cloud.google.com/spanner/docs/query-operators-unary#aggregate)                                                                                                 |
| `JOIN`                              | Any join operator; see [Joins](https://docs.cloud.google.com/spanner/docs/query-operators-binary)                                                                                       |
| Subquery                            | [Scalar](https://docs.cloud.google.com/spanner/docs/query-operators-scalar-subqueries) or [Array](https://docs.cloud.google.com/spanner/docs/query-operators-array-subqueries) subquery |
| `LIMIT`                             | [Limit](https://docs.cloud.google.com/spanner/docs/query-operators-unary#limit) , [Sort Limit](https://docs.cloud.google.com/spanner/docs/query-operators-unary#sort)                   |
| `ORDER BY`                          | [Sort, Sort Limit](https://docs.cloud.google.com/spanner/docs/query-operators-unary#sort)                                                                                               |

## Query execution operators

This section lists all query execution operators that can make up a query execution plan in Spanner.

### Leaf operators

Operators that have no children.

| Name                                                                                                   | Summary                                                                  |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| [Array unnest](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#array-unnest)           | Flattens an input array into rows of elements.                           |
| [Generate relation](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#generate-relation) | Returns zero or more rows.                                               |
| [Unit relation](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#unit-relation)         | Returns one row.                                                         |
| [Empty relation](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#empty-relation)       | Returns no rows.                                                         |
| [Scan](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#scan)                           | Scans a source of rows and returns them.                                 |
| [Filter scan](https://docs.cloud.google.com/spanner/docs/query-operators-leaf#filter_scan)             | Works with the scan to reduce the number of rows read from the database. |

### Unary operators

Operators that have a single relational child.

| Name                                                                                                            | Summary                                                                                                                           |
| --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [Aggregate](https://docs.cloud.google.com/spanner/docs/query-operators-unary#aggregate)                         | Implements `GROUP BY` SQL statements and aggregate functions.                                                                     |
| [Apply mutations](https://docs.cloud.google.com/spanner/docs/query-operators-unary#apply-mutations)             | Applies the mutations from a Data Manipulation Language (DML) statement to the table.                                             |
| [Create batch](https://docs.cloud.google.com/spanner/docs/query-operators-unary#create_batch)                   | Batches its input rows into a sequence.                                                                                           |
| [Compute](https://docs.cloud.google.com/spanner/docs/query-operators-unary#compute)                             | Produces output by reading its input rows and adding one or more additional columns that are computed using scalar expressions.   |
| [Compute struct](https://docs.cloud.google.com/spanner/docs/query-operators-unary#compute_struct)               | Creates a variable for a structure that contains fields for each of the input columns.                                            |
| [DataBlockToRowAdapter](https://docs.cloud.google.com/spanner/docs/query-operators-unary#datablocktorowadapter) | Adapts a batch-oriented execution method to a row-oriented execution method.                                                      |
| [Filter](https://docs.cloud.google.com/spanner/docs/query-operators-unary#filter)                               | Reads all rows from its input, applies a scalar predicate on each row, and then returns only the rows that satisfy the predicate. |
| [Limit](https://docs.cloud.google.com/spanner/docs/query-operators-unary#limit)                                 | Constrains the number of rows returned.                                                                                           |
| [Local split union](https://docs.cloud.google.com/spanner/docs/query-operators-unary#local-split-union)         | Finds table splits stored on the local server, runs a subquery on each split, and then creates a union that combines all results. |
| [Random ID assign](https://docs.cloud.google.com/spanner/docs/query-operators-unary#random_id_assign)           | Produces output by reading its input rows and adding a random number to each row.                                                 |
| [RowToDataBlockAdapter](https://docs.cloud.google.com/spanner/docs/query-operators-unary#rowtodatablockadapter) | Adapts a row-oriented execution method to a batch-oriented execution method.                                                      |
| [Serialize result](https://docs.cloud.google.com/spanner/docs/query-operators-unary#serialize_result)           | Serializes each row of the final result of the query for returning to the client.                                                 |
| [Sort](https://docs.cloud.google.com/spanner/docs/query-operators-unary#sort)                                   | Reads its input rows, orders them by column(s), and then returns the sorted results.                                              |
| [Table-valued function (TVF)](https://docs.cloud.google.com/spanner/docs/query-operators-unary#tvf)             | Produces output by reading its input rows and applying the specified function.                                                    |
| [Union input](https://docs.cloud.google.com/spanner/docs/query-operators-unary#union_input)                     | Returns results to a union all operator.                                                                                          |

### Binary operators

Operators that have two relational children.

| Name                                                                                                 | Summary                                                                                                          |
| ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| [Apply join](https://docs.cloud.google.com/spanner/docs/query-operators-binary#apply-join)           | Applies each row on the input side to the map side using an apply method.                                        |
| [Hash join](https://docs.cloud.google.com/spanner/docs/query-operators-binary#hash-join)             | Reads rows from input marked as build and inserts them into a hash table based on a join condition.              |
| [Merge join](https://docs.cloud.google.com/spanner/docs/query-operators-binary#merge-join)           | Consumes both input streams concurrently and outputs rows when the join condition is satisfied.                  |
| [Recursive union](https://docs.cloud.google.com/spanner/docs/query-operators-binary#recursive-union) | Performs a union of two inputs, one that represents a base case, and the other that represents a recursive case. |

### N-ary operators

Operators that have more than two relational children.

| Name                                                                                    | Summary                                                            |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| [Union all](https://docs.cloud.google.com/spanner/docs/query-operators-n-ary#union_all) | Combines all row sets of its children without removing duplicates. |

### Distributed operators

Operators that execute across multiple servers.

| Name                                                                                                                        | Summary                                                                                                                                               |
| --------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Distributed union](https://docs.cloud.google.com/spanner/docs/query-operators-distributed#distributed-union)               | Conceptually divides one or more tables into multiple splits, remotely evaluates a subquery independently on each split, and then unions all results. |
| [Distributed apply](https://docs.cloud.google.com/spanner/docs/query-operators-distributed#distributed-apply)               | Extends the apply join operator by executing across multiple servers.                                                                                 |
| [Distributed merge union](https://docs.cloud.google.com/spanner/docs/query-operators-distributed#distributed-merge-union)   | Distributes a query across multiple remote servers and then combines the query results to produce a sorted result.                                    |
| [Push broadcast hash join](https://docs.cloud.google.com/spanner/docs/query-operators-distributed#push-broadcast-hash-join) | Implements SQL joins using a distributed hash join.                                                                                                   |

### Scalar subqueries

SQL sub-expressions that return a single scalar value.

| Name                                                                                              | Summary                                                |
| ------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| [Scalar subqueries](https://docs.cloud.google.com/spanner/docs/query-operators-scalar-subqueries) | SQL sub-expressions that return a single scalar value. |

### Array subqueries

SQL sub-expressions that return an array.

| Name                                                                                            | Summary                                   |
| ----------------------------------------------------------------------------------------------- | ----------------------------------------- |
| [Array subqueries](https://docs.cloud.google.com/spanner/docs/query-operators-array-subqueries) | SQL sub-expressions that return an array. |

### Struct constructor

An operator that creates a struct (a collection of fields) for rows resulting from a compute operation.

| Name                                                                                                | Summary                                                                                                 |
| --------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| [Struct constructor](https://docs.cloud.google.com/spanner/docs/query-operators-struct-constructor) | An operator that creates a struct (a collection of fields) for rows resulting from a compute operation. |
