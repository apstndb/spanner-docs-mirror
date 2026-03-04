This page describes the operators used in Spanner [query execution plans](/spanner/docs/query-execution-plans) . For information about how to retrieve an execution plan for a specific query using the Google Cloud console, see [Understanding how Spanner executes queries](/spanner/docs/sql-best-practices#how-execute-queries) .

Execution plans support GoogleSQL-dialect databases and PostgreSQL-dialect databases.

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases.

## Mapping SQL constructs to query execution operators

The exact mapping between SQL constructs and query execution operators depends on the query optimization. The following table shows some common mappings:

<table>
<thead>
<tr class="header">
<th><strong>SQL</strong></th>
<th><strong>Query execution operator</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Table reference</td>
<td><a href="/spanner/docs/query-operators-leaf#scan">Table Scan, Index Scan</a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       WHERE      </code></td>
<td><a href="/spanner/docs/query-operators-leaf#filter_scan">Filter Scan</a> , <a href="/spanner/docs/query-operators-unary#filter">Filter</a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       GROUP BY      </code></td>
<td><a href="/spanner/docs/query-operators-unary#aggregate">Aggregate</a></td>
</tr>
<tr class="even">
<td>Scalar function (such as <code dir="ltr" translate="no">       ISNULL      </code> )</td>
<td><a href="/spanner/docs/query-operators-unary#compute">Compute</a></td>
</tr>
<tr class="odd">
<td>Aggregate function (such as <code dir="ltr" translate="no">       SUM      </code> )</td>
<td><a href="/spanner/docs/query-operators-unary#aggregate">Aggregate</a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       JOIN      </code></td>
<td>Any join operator; see <a href="/spanner/docs/query-operators-binary">Joins</a></td>
</tr>
<tr class="odd">
<td>Subquery</td>
<td><a href="/spanner/docs/query-operators-scalar-subqueries">Scalar</a> or <a href="/spanner/docs/query-operators-array-subqueries">Array</a> subquery</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       LIMIT      </code></td>
<td><a href="/spanner/docs/query-operators-unary#limit">Limit</a> , <a href="/spanner/docs/query-operators-unary#sort">Sort Limit</a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ORDER BY      </code></td>
<td><a href="/spanner/docs/query-operators-unary#sort">Sort, Sort Limit</a></td>
</tr>
</tbody>
</table>

## Query execution operators

This section lists all query execution operators that can make up a query execution plan in Spanner.

### Leaf operators

Operators that have no children.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-leaf#array-unnest">Array unnest</a></td>
<td>Flattens an input array into rows of elements.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-leaf#generate-relation">Generate relation</a></td>
<td>Returns zero or more rows.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-leaf#unit-relation">Unit relation</a></td>
<td>Returns one row.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-leaf#empty-relation">Empty relation</a></td>
<td>Returns no rows.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-leaf#scan">Scan</a></td>
<td>Scans a source of rows and returns them.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-leaf#filter_scan">Filter scan</a></td>
<td>Works with the scan to reduce the number of rows read from the database.</td>
</tr>
</tbody>
</table>

### Unary operators

Operators that have a single relational child.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#aggregate">Aggregate</a></td>
<td>Implements <code dir="ltr" translate="no">       GROUP BY      </code> SQL statements and aggregate functions.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#apply-mutations">Apply mutations</a></td>
<td>Applies the mutations from a Data Manipulation Language (DML) statement to the table.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#create_batch">Create batch</a></td>
<td>Batches its input rows into a sequence.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#compute">Compute</a></td>
<td>Produces output by reading its input rows and adding one or more additional columns that are computed using scalar expressions.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#compute_struct">Compute struct</a></td>
<td>Creates a variable for a structure that contains fields for each of the input columns.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#datablocktorowadapter">DataBlockToRowAdapter</a></td>
<td>Adapts a batch-oriented execution method to a row-oriented execution method.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#filter">Filter</a></td>
<td>Reads all rows from its input, applies a scalar predicate on each row, and then returns only the rows that satisfy the predicate.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#limit">Limit</a></td>
<td>Constrains the number of rows returned.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#local-split-union">Local split union</a></td>
<td>Finds table splits stored on the local server, runs a subquery on each split, and then creates a union that combines all results.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#random_id_assign">Random ID assign</a></td>
<td>Produces output by reading its input rows and adding a random number to each row.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#rowtodatablockadapter">RowToDataBlockAdapter</a></td>
<td>Adapts a row-oriented execution method to a batch-oriented execution method.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#serialize_result">Serialize result</a></td>
<td>Serializes each row of the final result of the query for returning to the client.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#sort">Sort</a></td>
<td>Reads its input rows, orders them by column(s), and then returns the sorted results.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-unary#tvf">Table-valued function (TVF)</a></td>
<td>Produces output by reading its input rows and applying the specified function.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-unary#union_input">Union input</a></td>
<td>Returns results to a union all operator.</td>
</tr>
</tbody>
</table>

### Binary operators

Operators that have two relational children.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-binary#apply-join">Apply join</a></td>
<td>Applies each row on the input side to the map side using an apply method.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-binary#hash-join">Hash join</a></td>
<td>Reads rows from input marked as build and inserts them into a hash table based on a join condition.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-binary#merge-join">Merge join</a></td>
<td>Consumes both input streams concurrently and outputs rows when the join condition is satisfied.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-binary#recursive-union">Recursive union</a></td>
<td>Performs a union of two inputs, one that represents a base case, and the other that represents a recursive case.</td>
</tr>
</tbody>
</table>

### N-ary operators

Operators that have more than two relational children.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-n-ary#union_all">Union all</a></td>
<td>Combines all row sets of its children without removing duplicates.</td>
</tr>
</tbody>
</table>

### Distributed operators

Operators that execute across multiple servers.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-distributed#distributed-union">Distributed union</a></td>
<td>Conceptually divides one or more tables into multiple splits, remotely evaluates a subquery independently on each split, and then unions all results.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-distributed#distributed-apply">Distributed apply</a></td>
<td>Extends the apply join operator by executing across multiple servers.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-distributed#distributed-merge-union">Distributed merge union</a></td>
<td>Distributes a query across multiple remote servers and then combines the query results to produce a sorted result.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-operators-distributed#push-broadcast-hash-join">Push broadcast hash join</a></td>
<td>Implements SQL joins using a distributed hash join.</td>
</tr>
</tbody>
</table>

### Scalar subqueries

SQL sub-expressions that return a single scalar value.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-scalar-subqueries">Scalar subqueries</a></td>
<td>SQL sub-expressions that return a single scalar value.</td>
</tr>
</tbody>
</table>

### Array subqueries

SQL sub-expressions that return an array.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-array-subqueries">Array subqueries</a></td>
<td>SQL sub-expressions that return an array.</td>
</tr>
</tbody>
</table>

### Struct constructor

An operator that creates a struct (a collection of fields) for rows resulting from a compute operation.

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/query-operators-struct-constructor">Struct constructor</a></td>
<td>An operator that creates a struct (a collection of fields) for rows resulting from a compute operation.</td>
</tr>
</tbody>
</table>
