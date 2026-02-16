This page describes details about operators used in Spanner [Query execution plans](/spanner/docs/query-execution-plans) . To learn how to retrieve an execution plan for a specific query using the Google Cloud console, see [Understanding how Spanner executes queries](/spanner/docs/sql-best-practices#how-execute-queries) .

Execution plans support GoogleSQL-dialect databases and PostgreSQL-dialect databases.

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases.

The queries and execution plans on this page are based on the following database schema:

``` text
CREATE TABLE Singers (
  SingerId   INT64 NOT NULL,
  FirstName  STRING(1024),
  LastName   STRING(1024),
  SingerInfo BYTES(MAX),
  BirthDate  DATE
) PRIMARY KEY(SingerId);

CREATE INDEX SingersByFirstLastName ON Singers(FirstName, LastName);

CREATE TABLE Albums (
  SingerId        INT64 NOT NULL,
  AlbumId         INT64 NOT NULL,
  AlbumTitle      STRING(MAX),
  MarketingBudget INT64
) PRIMARY KEY(SingerId, AlbumId),
  INTERLEAVE IN PARENT Singers ON DELETE CASCADE;

CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle);

CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget);

CREATE TABLE Songs (
  SingerId  INT64 NOT NULL,
  AlbumId   INT64 NOT NULL,
  TrackId   INT64 NOT NULL,
  SongName  STRING(MAX),
  Duration  INT64,
  SongGenre STRING(25)
) PRIMARY KEY(SingerId, AlbumId, TrackId),
  INTERLEAVE IN PARENT Albums ON DELETE CASCADE;

CREATE INDEX SongsBySingerAlbumSongNameDesc ON Songs(SingerId, AlbumId, SongName DESC), INTERLEAVE IN Albums;

CREATE INDEX SongsBySongName ON Songs(SongName);

CREATE TABLE Concerts (
  VenueId      INT64 NOT NULL,
  SingerId     INT64 NOT NULL,
  ConcertDate  DATE NOT NULL,
  BeginTime    TIMESTAMP,
  EndTime      TIMESTAMP,
  TicketPrices ARRAY<INT64>
) PRIMARY KEY(VenueId, SingerId, ConcertDate);
```

You can use the following Data Manipulation Language (DML) statements to add data to these tables:

``` text
INSERT INTO Singers (SingerId, FirstName, LastName, BirthDate)
VALUES (1, "Marc", "Richards", "1970-09-03"),
       (2, "Catalina", "Smith", "1990-08-17"),
       (3, "Alice", "Trentor", "1991-10-02"),
       (4, "Lea", "Martin", "1991-11-09"),
       (5, "David", "Lomond", "1977-01-29");

INSERT INTO Albums (SingerId, AlbumId, AlbumTitle)
VALUES (1, 1, "Total Junk"),
       (1, 2, "Go, Go, Go"),
       (2, 1, "Green"),
       (2, 2, "Forever Hold Your Peace"),
       (2, 3, "Terrified"),
       (3, 1, "Nothing To Do With Me"),
       (4, 1, "Play");

INSERT INTO Songs (SingerId, AlbumId, TrackId, SongName, Duration, SongGenre)
VALUES (2, 1, 1, "Let's Get Back Together", 182, "COUNTRY"),
       (2, 1, 2, "Starting Again", 156, "ROCK"),
       (2, 1, 3, "I Knew You Were Magic", 294, "BLUES"),
       (2, 1, 4, "42", 185, "CLASSICAL"),
       (2, 1, 5, "Blue", 238, "BLUES"),
       (2, 1, 6, "Nothing Is The Same", 303, "BLUES"),
       (2, 1, 7, "The Second Time", 255, "ROCK"),
       (2, 3, 1, "Fight Story", 194, "ROCK"),
       (3, 1, 1, "Not About The Guitar", 278, "BLUES");
```

**Note:** You can run queries and retrieve execution plans even if the tables have no data.

## Leaf operators

A *leaf* operator is an operator that has no children. The types of leaf operators are:

  - [Array unnest](#array-unnest)
  - [Generate relation](#generate-relation)
  - [Unit relation](#unit-relation)
  - [Empty relation](#empty-relation)
  - [Scan](#scan)

### Array unnest

An *array unnest* operator flattens an input array into rows of elements. Each resulting row contains up to two columns: the actual value from the array, and optionally the zero-based position in the array.

For example, using this query:

``` text
SELECT a, b FROM UNNEST([1,2,3]) a WITH OFFSET b;
```

The query flattens the array `  [1,2,3]  ` in column `  a  ` and shows the array position in column `  b  ` .

These are the results:

<table>
<thead>
<tr class="header">
<th>a</th>
<th>b</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>0</td>
</tr>
<tr class="even">
<td>2</td>
<td>1</td>
</tr>
<tr class="odd">
<td>3</td>
<td>2</td>
</tr>
</tbody>
</table>

This is the execution plan:

### Generate relation

A *generate relation* operator returns zero or more rows.

### Unit relation

The *unit relation* returns one row. It is a special case of the *generate relation* operator.

For example, using this query:

``` text
SELECT 1 + 2 AS Result;
```

The result is:

<table>
<thead>
<tr class="header">
<th>Result</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>3</td>
</tr>
</tbody>
</table>

This is the execution plan:

### Empty relation

The *empty relation* returns no rows. It is a special case of the *generate relation* operator.

For example, using this query:

``` text
SELECT *
FROM   albums
LIMIT  0
```

The result is:

No results

This is the execution plan:

### Scan

A *scan* operator returns rows by scanning a source of rows. These are the types of scan operators:

  - *Table scan* : The scan occurs on a table.
  - *Index scan* : The scan occurs on an index.
  - *Batch scan* : The scan occurs on intermediate tables created by other relational operators (for example, a table created by a [distributed cross apply](#distributed-cross-apply) ).

Whenever possible, Spanner applies predicates on keys as part of a scan. Scans execute more efficiently when predicates are applied because the scan doesn't need to read the entire table or index. Predicates appear in the execution plan in the form `  KeyPredicate: column=value  ` .

In the worst case, a query may need to look up all the rows in a table. This situation leads to a full scan, and appears in the execution plan as `  full scan: true  ` .

For example, using this query:

``` text
SELECT s.lastname
FROM   singers@{FORCE_INDEX=SingersByFirstLastName} as s
WHERE  s.firstname = 'Catalina';
```

These are the results:

<table>
<thead>
<tr class="header">
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Smith</td>
</tr>
</tbody>
</table>

This is the execution plan:

In the execution plan, the top-level [distributed union](#distributed-union) operator sends subplans to remote servers. Each subplan has a [serialize result](#serialize_result) operator and an index scan operator. The predicate `  Key Predicate: FirstName = 'Catalina'  ` restricts the scan to rows in the index `  SingersByFirstLastname  ` that have `  FirstName  ` equal to `  Catalina  ` . The output of the index scan is returned to the serialize result operator.

## Unary operators

A *unary* operator is an operator that has a single relational child.

The following operators are unary operators:

  - [Aggregate](#aggregate)
  - [Apply mutations](#apply-mutations)
  - [Create batch](#create_batch)
  - [Compute](#compute)
  - [Compute struct](#compute_struct)
  - [DataBlockToRowAdapter](#datablocktorowadapter)
  - [Filter](#filter)
  - [Filter scan](#filter_scan)
  - [Limit](#limit)
  - [Local split union](#local_split_union)
  - [Random Id Assign](#random_id_assign)
  - [RowToDataBlockAdapter](#rowtodatablockadapter)
  - [Serialize result](#serialize_result)
  - [Sort](#sort)
  - [TVF](#tvf)
  - [Union input](#union_input)

### Aggregate

An *aggregate* operator implements `  GROUP BY  ` SQL statements and aggregate functions (such as `  COUNT  ` ). The input for an aggregate operator is logically partitioned into groups arranged on key columns (or into a single group if `  GROUP BY  ` isn't present). For each group, zero or more aggregates are computed.

For example, using this query:

``` text
SELECT s.singerid,
       Avg(s.duration) AS average,
       Count(*)        AS count
FROM   songs AS s
GROUP  BY singerid;
```

The query groups by `  SingerId  ` and performs an `  AVG  ` aggregation and a `  COUNT  ` aggregation.

These are the results:

<table>
<thead>
<tr class="header">
<th>SingerId</th>
<th>average</th>
<th>count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>3</td>
<td>278</td>
<td>1</td>
</tr>
<tr class="even">
<td>2</td>
<td>225.875</td>
<td>8</td>
</tr>
</tbody>
</table>

This is the execution plan:

Aggregate operators can be *stream-based* or *hash-based* . The previous execution plan shows a stream-based aggregate. Stream-based aggregates read from already pre-sorted input (if `  GROUP BY  ` is present) and compute the group without blocking. Hash-based aggregates build hash tables to maintain incremental aggregates of multiple input rows simultaneously. Stream-based aggregates are faster and use less memory than hash-based aggregates, but require the input to be sorted (either by key columns or [secondary indexes](/spanner/docs/secondary-indexes) ).

For distributed scenarios, an aggregate operator can be separated into a local-global pair. Each remote server performs the local aggregation on its input rows, and then returns its results to the root server. The root server performs the global aggregation.

### Apply mutations

An *apply mutations* operator applies the mutations from a [Data Manipulation Statement](/spanner/docs/dml-tasks) (DML) to the table. It's the top operator in a query plan for a DML statement.

For example, using this query:

``` text
DELETE FROM singers
WHERE  firstname = 'Alice';
```

These are the results:

``` text
 4 rows deleted  This statement deleted 4 rows and did not return any rows.
```

This is the execution plan:

### Create batch

A *create batch* operator batches its input rows into a sequence. A create batch operation usually occurs as a part of a [distributed cross apply](#distributed-cross-apply) operation. The input rows can be re-ordered during the batching. The number of input rows that get batched in each execution of the batch operator varies.

See the [distributed cross apply](#distributed-cross-apply) operator for an example of a create batch operator in an execution plan.

### Compute

A *compute* operator produces output by reading its input rows and adding one or more additional columns that are computed using scalar expressions. See the [union all](#union_all) operator for an example of a compute operator in an execution plan.

### Compute struct

A *compute struct* operator creates a variable for a structure that contains fields for each of the input columns.

For example, using this query:

``` text
SELECT FirstName,
       ARRAY(SELECT AS STRUCT song.SongName, song.SongGenre
             FROM Songs AS song
             WHERE song.SingerId = singer.SingerId)
FROM singers AS singer
WHERE singer.SingerId = 3;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>FirstName</th>
<th>Unspecified</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Alice</td>
<td>[["Not About The Guitar","BLUES"]]</td>
</tr>
</tbody>
</table>

This is the execution plan:

In the execution plan, the array subquery operator receives input from a [distributed union](#distributed-union) operator, which receives input from a compute struct operator. The compute struct operator creates a structure from the `  SongName  ` and `  SongGenre  ` columns in the `  Songs  ` table.

### DataBlockToRowAdapter

A `  DataBlockToRowAdapter  ` operator is automatically inserted by the Spanner query optimizer between a pair of operators that operate using different execution methods. Its input is an operator using the batch-oriented execution method and its output is fed into an operator executing in the row-oriented execution method. For more information, see [Optimize query execution](/spanner/docs/sql-best-practices#optimize-query-execution) .

### Filter

A *filter* operator reads all rows from its input, applies a scalar predicate on each row, and then returns only the rows that satisfy the predicate.

For example, using this query:

``` text
SELECT s.lastname
FROM   (SELECT s.lastname
        FROM   singers AS s
        LIMIT  3) s
WHERE  s.lastname LIKE 'Rich%';
```

These are the results:

<table>
<thead>
<tr class="header">
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Richards</td>
</tr>
</tbody>
</table>

This is the execution plan:

The predicate for singers whose last name starts with `  Rich  ` is implemented as a filter. The filter's input is the output from an index [scan](#scan) , and the filter's output are rows where `  LastName  ` starts with `  Rich  ` .

For performance, whenever a filter is directly positioned above a [scan](#scan) , the filter impacts how data is read. For example, consider a table with key `  k  ` . A filter with predicate `  k = 5  ` directly on top of a scan of the table looks for rows that match `  k = 5  ` , without reading the entire input. This results in more efficient execution of the query. In the previous example, the filter operator reads only the rows that satisfy the `  WHERE s.LastName LIKE 'Rich%'  ` predicate.

### Filter scan

A *filter scan* operator is always on top of a [table or index scan](#scan) . It works with the scan to reduce the number of rows read from the database, and the resulting scan is typically faster than with a [filter](#filter) . Spanner applies the filter scan in certain conditions:

  - Seekable condition: The seekable condition applies if Spanner can determine a specific row to access in the table. In general, this happens when the filter is on a prefix of the primary key. For example, if the primary key consists of `  Col1  ` and `  Col2  ` , then a `  WHERE  ` clause that includes explicit values for `  Col1  ` , or `  Col1  ` and `  Col2  ` is seekable. In that case, Spanner reads data only within the key range.
  - Residual condition: Any other condition where Spanner can evaluate the scan to limit the amount of data that's read.

For example, using this query:

``` text
SELECT lastname
FROM   singers
WHERE  singerid = 1
```

These are the results:

<table>
<thead>
<tr class="header">
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Richards</td>
</tr>
</tbody>
</table>

This is the execution plan:

### Limit

A *limit* operator constrains the number of rows returned. An optional `  OFFSET  ` parameter specifies the starting row to return. For distributed scenarios, a limit operator can be separated into a local-global pair. Each remote server applies the local limit for its output rows, and then returns its results to the root server. The root server aggregates the rows sent by the remote servers and then applies the global limit.

For example, using this query:

``` text
SELECT s.songname
FROM   songs AS s
LIMIT  3;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Not About The Guitar</td>
</tr>
<tr class="even">
<td>The Second Time</td>
</tr>
<tr class="odd">
<td>Starting Again</td>
</tr>
</tbody>
</table>

This is the execution plan:

The local limit is the limit for each remote server. The root server aggregates the rows from the remote servers and then applies the global limit.

### Random ID Assign

A *random ID assign* operator produces output by reading its input rows and adding a random number to each row. It works with a `  Filter  ` or `  Sort  ` operator to achieve sampling methods. Supported sampling methods are [Bernoulli](https://en.wikipedia.org/wiki/Bernoulli_distribution) and [Reservoir](https://en.wikipedia.org/wiki/Reservoir_sampling) .

For example, the following query uses Bernoulli sampling with a sampling rate of 10 percent.

``` text
SELECT s.songname
FROM   songs AS s TABLESAMPLE bernoulli (10 PERCENT);
```

These are the results:

<table>
<thead>
<tr class="header">
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Starting Again</td>
</tr>
<tr class="even">
<td>Nothing Is The Same</td>
</tr>
</tbody>
</table>

Note that because the result is a sample, the result could vary each time the query is run even though the query is the same.

This is the execution plan:

In this execution plan, the `  Random Id Assign  ` operator receives its input from a [distributed union](#distributed-union) operator, which receives its input from an [index scan](#scan) . The operator returns the rows with random ids and the `  Filter  ` operator then applies a scalar predicate on the random ids and returns approximately 10 percent of the rows.

The following example uses [Reservoir](https://en.wikipedia.org/wiki/Reservoir_sampling)

sampling with a sampling rate of 2 rows.

``` text
SELECT s.songname
FROM   songs AS s TABLESAMPLE reservoir (2 rows);
```

These are the results:

<table>
<thead>
<tr class="header">
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>The Second Time</td>
</tr>
</tbody>
</table>

Note that because the result is a sample, the result could vary each time the query is run even though the query is the same.

This is the execution plan:

In this execution plan, the `  Random Id Assign  ` operator receives its input from a [distributed union](#distributed-union) operator, which receives its input from an [index scan](#scan) . The operator returns the rows with random ids and the `  Sort  ` operator then applies the sort order on the random ids and apply `  LIMIT  ` with 2 rows.

### Local split union

A *local split union* operator finds table [splits](/spanner/docs/schema-and-data-model#database-splits) stored on the local server, runs a subquery on each split, and then creates a union that combines all results.

A *local split union* appears in execution plans that scan a [placement](/spanner/docs/create-manage-data-placements) table. Placements can increase the number of splits in a table, making it more efficient to scan splits in batches based on their physical storage locations.

For example, suppose the `  Singers  ` table uses a placement key to partition singer data:

``` text
CREATE TABLE Singers (
    SingerId INT64 NOT NULL,
    SingerName STRING(MAX) NOT NULL,
    ...
    Location STRING(MAX) NOT NULL PLACEMENT KEY
) PRIMARY KEY (SingerId);
```

Now, consider this query:

``` text
SELECT BirthDate FROM Singers;
```

This is the execution plan:

The [distributed union](#distributed-union) sends a subquery to each batch of splits physically stored together in the same server. On each server, the *local split union* finds splits storing `  Singers  ` data, executes the subquery on each split, and returns the combined results. In this way, the distributed union and local split union work together to efficiently scan the `  Singers  ` table. Without a local split union, the distributed union would send one RPC per split instead of per split batch, resulting in redundant RPC round trips when there's more than one split per batch.

### RowToDataBlockAdapter

A `  RowToDataBlockAdapter  ` operator is automatically inserted by the Spanner query optimizer between a pair of operators that operate using different execution methods. Its input is an operator using the row-oriented execution method and its output is fed into an operator executing in the batch-oriented execution method. For more information, see [Optimize query execution](/spanner/docs/sql-best-practices#optimize-query-execution) .

### Serialize result

A *serialize result* operator is a special case of the compute struct operator that serializes each row of the final result of the query, for returning to the client.

For example, using this query:

``` text
SELECT array
  (
    select as struct so.songname,
            so.songgenre
    FROM   songs AS so
    WHERE  so.singerid = s.singerid)
FROM  singers AS s;
```

The query asks for an array of `  SongName  ` and `  SongGenre  ` based on `  SingerId  ` .

These are the results:

<table>
<thead>
<tr class="header">
<th>Unspecified</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>[]</td>
</tr>
<tr class="even">
<td>[[Let's Get Back Together, COUNTRY], [Starting Again, ROCK]]</td>
</tr>
<tr class="odd">
<td>[[Not About The Guitar, BLUES]]</td>
</tr>
<tr class="even">
<td>[]</td>
</tr>
<tr class="odd">
<td>[]</td>
</tr>
</tbody>
</table>

This is the execution plan:

The serialize result operator creates a result that contains, for each row of the `  Singers  ` table, an array of `  SongName  ` and `  SongGenre  ` pairs for the songs by the singer.

### Sort

A *sort* operator reads its input rows, orders them by column(s), and then returns the sorted results.

For example, using this query:

``` text
SELECT s.songgenre
FROM   songs AS s
ORDER  BY songgenre;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>SongGenre</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>BLUES</td>
</tr>
<tr class="even">
<td>BLUES</td>
</tr>
<tr class="odd">
<td>BLUES</td>
</tr>
<tr class="even">
<td>BLUES</td>
</tr>
<tr class="odd">
<td>CLASSICAL</td>
</tr>
<tr class="even">
<td>COUNTRY</td>
</tr>
<tr class="odd">
<td>ROCK</td>
</tr>
<tr class="even">
<td>ROCK</td>
</tr>
<tr class="odd">
<td>ROCK</td>
</tr>
</tbody>
</table>

This is the execution plan:

In this execution plan, the sort operator receives its input rows from a [distributed union](#distributed-union) operator, sorts the input rows, and returns the sorted rows to a [serialize result](#serialize_result) operator.

To constrain the number of rows returned, a sort operator can optionally have `  LIMIT  ` and `  OFFSET  ` parameters. For distributed scenarios, a sort operator with a `  LIMIT  ` or `  OFFSET  ` operator is separated into a local-global pair. Each remote server applies the sort order and the local limit or offset for its input rows, and then returns its results to the root server. The root server aggregates the rows sent by the remote servers, sorts them, and then applies the global limit/offset.

For example, using this query:

``` text
SELECT s.songgenre
FROM   songs AS s
ORDER  BY songgenre
LIMIT  3;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>SongGenre</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>BLUES</td>
</tr>
<tr class="even">
<td>BLUES</td>
</tr>
<tr class="odd">
<td>BLUES</td>
</tr>
</tbody>
</table>

This is the execution plan:

The execution plan shows the local limit for the remote servers and the global limit for the root server.

### TVF

A *table valued function* operator produces output by reading its input rows and applying the specified function. The function might implement mapping and return the same number of rows as input. It can also be a generator that returns more rows or a filter that returns less rows.

For example, using this query:

``` text
SELECT genre,
       songname
FROM   ml.predict(model genreclassifier, TABLE songs)
```

These are the results:

<table>
<thead>
<tr class="header">
<th>Genre</th>
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Country</td>
<td>Not About The Guitar</td>
</tr>
<tr class="even">
<td>Rock</td>
<td>The Second Time</td>
</tr>
<tr class="odd">
<td>Pop</td>
<td>Starting Again</td>
</tr>
<tr class="even">
<td>Pop</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="odd">
<td>Country</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="even">
<td>Pop</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="odd">
<td>Electronic</td>
<td>Blue</td>
</tr>
<tr class="even">
<td>Rock</td>
<td>42</td>
</tr>
<tr class="odd">
<td>Rock</td>
<td>Fight Story</td>
</tr>
</tbody>
</table>

This is the execution plan:

### Union input

A *union input* operator returns results to a [union all](#union_all) operator. See the [union all](#union_all) operator for an example of a union input operator in an execution plan.

## Binary operators

A *binary* operator is an operator that has two relational children. The following operators are binary operators:

  - [Cross apply](#cross-apply)
  - [Hash join](#hash-join)
  - [Merge join](#merge-join)
  - [Push broadcast hash join](#push-broadcast-hash-join)
  - [Outer apply](#outer-apply)
  - [Recursive union](#recursive-union)

### Cross apply

A *cross apply* operator runs a table query on each row retrieved by a query of another table, and returns the union of all the table query runs. Cross apply and [outer apply](#outer-apply) operators execute row-oriented processing, unlike operators that execute set-based processing such as [hash join](#hash-join) . The cross apply operator has two inputs, *input* and *map* . The cross apply operator applies each row in the input side to the map side. The result of the cross apply has columns from both the input and map sides.

For example, using this query:

``` text
 SELECT si.firstname,
       (SELECT so.songname
        FROM   songs AS so
        WHERE  so.singerid = si.singerid
        LIMIT  1)
FROM   singers AS si;
```

The query asks for the first name of each singer, along with the name of only one of the singer's songs.

These are the results:

<table>
<thead>
<tr class="header">
<th>FirstName</th>
<th>Unspecified</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Alice</td>
<td>Not About The Guitar</td>
</tr>
<tr class="even">
<td>Catalina</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="odd">
<td>David</td>
<td>NULL</td>
</tr>
<tr class="even">
<td>Lea</td>
<td>NULL</td>
</tr>
<tr class="odd">
<td>Marc</td>
<td>NULL</td>
</tr>
</tbody>
</table>

The first column is populated from the `  Singers  ` table, and the second column is populated from the `  Songs  ` table. In cases where a `  SingerId  ` existed in the `  Singers  ` table but there was no matching `  SingerId  ` in the `  Songs  ` table, the second column contains `  NULL  ` .

This is the execution plan:

The top-level node is a [distributed union](#distributed-union) operator. The distributed union operator distributes sub plans to remote servers. The subplan contains a [serialize result](#serialize_result) operator that computes the singer's first name and the name of one of the singer's songs and serializes each row of the output.

The serialize result operator receives its input from a cross apply operator. The input side for the cross apply operator is a table [scan](#scan) on the `  Singers  ` table.

The map side for the cross apply operation contains the following (from top to bottom):

  - An [aggregate](#aggregate) operator that returns `  Songs.SongName  ` .
  - A [limit](#limit) operator that limits the number of songs returned to one per singer.
  - An index [scan](#scan) on the `  SongsBySingerAlbumSongNameDesc  ` index.

The cross apply operator maps each row from the input side to a row in the map side that has the same `  SingerId  ` . The cross apply operator output is the `  FirstName  ` value from the input row, and the `  SongName  ` value from the map row. (The `  SongName  ` value is `  NULL  ` if there is no map row that matches on `  SingerId  ` .) The distributed union operator at the top of the execution plan then combines all of the output rows from the remote servers and returns them as the query results.

### Hash join

A *hash join* operator is a hash-based implementation of SQL joins. Hash joins execute set-based processing. The hash join operator reads rows from input marked as *build* and inserts them into a hash table based on a join condition. The hash join operator then reads rows from input marked as *probe* . For each row it reads from the probe input, the hash join operator looks for matching rows in the hash table. The hash join operator returns the matching rows as its result.

For example, using this query:

``` text
SELECT a.albumtitle,
       s.songname
FROM   albums AS a join@{join_method=hash_join} songs AS s
ON a.singerid = s.singerid
AND    a.albumid = s.albumid;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>AlbumTitle</th>
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>Not About The Guitar</td>
</tr>
<tr class="even">
<td>Green</td>
<td>The Second Time</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>Starting Again</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="even">
<td>Green</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>Blue</td>
</tr>
<tr class="even">
<td>Green</td>
<td>42</td>
</tr>
<tr class="odd">
<td>Terrified</td>
<td>Fight Story</td>
</tr>
</tbody>
</table>

This is the execution plan:

In the execution plan, *build* is a [distributed union](#distributed-union) that distributes [scans](#scan) on the table `  Albums  ` . *Probe* is a distributed union operator that distributes scans on the index `  SongsBySingerAlbumSongNameDesc  ` . The hash join operator reads all rows from the build side. Each build row is placed in a hash table based on the columns in the condition `  a.SingerId = s.SingerId AND a.AlbumId = s.AlbumId  ` . Next, the hash join operator reads all rows from the probe side. For each probe row, the hash join operator looks for matches in the hash table. The resulting matches are returned by the hash join operator.

Resulting matches in the hash table might also be filtered by a residual condition before they're returned. (An example of where residual conditions appear is in non-equality joins). Hash join execution plans can be complex due to memory management and join variants. The main hash join algorithm is adapted to handle inner, semi, anti, and outer join variants.

**Note:** A [visualized query plan](/spanner/docs/tune-query-with-visualizer) renders *build* and *probe* as the left and right children, respectively, of a hash join.

### Merge join

A *merge join* operator is a merge-based implementation of SQL join. Both sides of the join produce rows ordered by the columns used in the join condition. The merge join consumes both input streams concurrently and outputs rows when the join condition is satisfied. If the inputs are not originally sorted as required then the optimizer adds explicit `  Sort  ` operators to the plan.

*Merge join* isn't selected automatically by the optimizer. To use this operator, set the join method to [`  MERGE_JOIN  `](/spanner/docs/reference/standard-sql/query-syntax#join-methods) on the query hint, as shown in the following example:

``` text
SELECT a.albumtitle,
       s.songname
FROM   albums AS a join@{join_method=merge_join} songs AS s
ON     a.singerid = s.singerid
AND    a.albumid = s.albumid;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>AlbumTitle</th>
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Green</td>
<td>The Second Time</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Starting Again</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Blue</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>42</td>
</tr>
<tr class="even">
<td>Terrified</td>
<td>Fight Story</td>
</tr>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>Not About The Guitar</td>
</tr>
</tbody>
</table>

This is the execution plan:

In this execution plan, the merge join is distributed so that the join executes where the data is located. This also allows the merge join in this example to operate without the introduction of additional sort operators, because both table scans are already sorted by `  SingerId  ` , `  AlbumId  ` , which is the join condition. In this plan the left hand side scan of the `  Albums  ` table advances whenever its `  SingerId  ` , `  AlbumId  ` is comparatively less than the right hand side `  SongsBySingerAlbumSongNameDesc  ` index scan `  SingerId_1  ` , `  AlbumId_1  ` pair. Similarly, the right hand side advances whenever it's less than the left hand side. This merge advance continues searching for equivalences such that resulting matches can be returned.

Consider another *merge join* example using the following query:

``` text
SELECT a.albumtitle,
       s.songname
FROM   albums AS a join@{join_method=merge_join} songs AS s
ON a.albumid = s.albumid;
```

It yields the following results:

<table>
<thead>
<tr class="header">
<th>AlbumTitle</th>
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Total Junk</td>
<td>The Second Time</td>
</tr>
<tr class="even">
<td>Total Junk</td>
<td>Starting Again</td>
</tr>
<tr class="odd">
<td>Total Junk</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="even">
<td>Total Junk</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="odd">
<td>Total Junk</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>Total Junk</td>
<td>Blue</td>
</tr>
<tr class="odd">
<td>Total Junk</td>
<td>42</td>
</tr>
<tr class="even">
<td>Total Junk</td>
<td>Not About The Guitar</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>The Second Time</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Starting Again</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Blue</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>42</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Not About The Guitar</td>
</tr>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>The Second Time</td>
</tr>
<tr class="even">
<td>Nothing To Do With Me</td>
<td>Starting Again</td>
</tr>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="even">
<td>Nothing To Do With Me</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>Nothing To Do With Me</td>
<td>Blue</td>
</tr>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>42</td>
</tr>
<tr class="even">
<td>Nothing To Do With Me</td>
<td>Not About The Guitar</td>
</tr>
<tr class="odd">
<td>Play</td>
<td>The Second Time</td>
</tr>
<tr class="even">
<td>Play</td>
<td>Starting Again</td>
</tr>
<tr class="odd">
<td>Play</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="even">
<td>Play</td>
<td>Let's Get Back Together</td>
</tr>
<tr class="odd">
<td>Play</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>Play</td>
<td>Blue</td>
</tr>
<tr class="odd">
<td>Play</td>
<td>42</td>
</tr>
<tr class="even">
<td>Play</td>
<td>Not About The Guitar</td>
</tr>
<tr class="odd">
<td>Terrified</td>
<td>Fight Story</td>
</tr>
</tbody>
</table>

This is the execution plan:

In the preceding execution plan, additional `  Sort  ` operators have been introduced by the query optimizer to achieve the necessary properties for the merge join to execute. The `  JOIN  ` condition in this example's query is only on `  AlbumId  ` , which isn't how the data is stored, so a sort must be added. The query engine supports a Distributed Merge algorithm, allowing the sort to happen locally instead of globally, which distributes and parallelizes the CPU cost.

The resulting matches might also be filtered by a residual condition before they are returned. (An example of where residual conditions appear is in non-equality joins). Merge join execution plans can be complex due to additional sort requirements. The main merge join algorithm is adapted to handle inner, semi, anti, and outer join variants.

### Push broadcast hash join

A *push broadcast hash join* operator is a distributed hash-join-based implementation of SQL joins. The push broadcast hash join operator reads rows from the input side in order to construct a batch of data. That batch is then broadcast to all servers containing map side data. On the destination servers where the batch of data is received, a hash join is built using the batch as the build side data and the local data is then scanned as the probe side of the hash join.

*Push broadcast hash join* isn't selected automatically by the optimizer. To use this operator, set the join method to [`  PUSH_BROADCAST_HASH_JOIN  `](/spanner/docs/reference/standard-sql/query-syntax#join-methods) on the query hint, as shown in the following example:

``` text
SELECT a.albumtitle,
       s.songname
FROM   albums AS a join@{join_method=push_broadcast_hash_join} songs AS s
ON     a.singerid = s.singerid
AND    a.albumid = s.albumid;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>AlbumTitle</th>
<th>SongName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Green</td>
<td>The Second Time</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Starting Again</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>Nothing Is The Same</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Lets Get Back Together</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>I Knew You Were Magic</td>
</tr>
<tr class="even">
<td>Green</td>
<td>Blue</td>
</tr>
<tr class="odd">
<td>Green</td>
<td>42</td>
</tr>
<tr class="even">
<td>Terrified</td>
<td>Fight Story</td>
</tr>
<tr class="odd">
<td>Nothing To Do With Me</td>
<td>Not About The Guitar</td>
</tr>
</tbody>
</table>

This is the execution plan:

The input to the Push broadcast hash join is the `  AlbumsByAlbumTitle  ` index. That input is serialized into a batch of data. That batch is then sent to all the local splits of the index `  SongsBySingerAlbumSongNameDesc  ` , where the batch is then deserialized and built into a hash table. The hash table then uses the local index data as a probe returning resulting matches.

Resulting matches might also be filtered by a residual condition before they're returned. (An example of where residual conditions appear is in non-equality joins).

### Outer apply

An *outer apply* operator is similar to a [cross apply](#cross-apply) operator, except an outer apply operator ensures that each execution on the map side returns at least one row by manufacturing a NULL-padded row if needed. (In other words, it provides left outer join semantics.)

### Recursive union

A *recursive union* operator performs a union of two inputs, one that represents a `  base  ` case, and the other that represents a `  recursive  ` case. It's used in graph queries with quantified path traversals. The base input is processed first and exactly once. The recursive input is processed until the recursion terminates. The recursion terminates when the upper bound, if specified, is reached, or when the recursion doesn't produce any new results. In the following example, the `  Collaborations  ` table is added to the schema, and a property graph called `  MusicGraph  ` is created.

``` text
CREATE TABLE Collaborations (
    SingerId INT64 NOT NULL,
    FeaturingSingerId INT64 NOT NULL,
    AlbumTitle STRING(MAX) NOT NULL,
) PRIMARY KEY(SingerId, FeaturingSingerId, AlbumTitle);

CREATE OR REPLACE PROPERTY GRAPH MusicGraph
    NODE TABLES(
        Singers
            KEY(SingerId)
            LABEL Singers PROPERTIES(
                BirthDate,
                FirstName,
                LastName,
                SingerId,
                SingerInfo)
            )
EDGE TABLES(
    Collaborations AS CollabWith
        KEY(SingerId, FeaturingSingerId, AlbumTitle)
        SOURCE KEY(SingerId) REFERENCES Singers(SingerId)
        DESTINATION KEY(FeaturingSingerId) REFERENCES Singers(SingerId)
        LABEL CollabWith PROPERTIES(
          AlbumTitle,
          FeaturingSingerId,
          SingerId),
);
```

The following graph query finds singers who have collaborated with a given singer or collaborated with those collaborators.

``` text
GRAPH MusicGraph
MATCH (singer:Singers {singerId:42})-[c:CollabWith]->{1,2}(featured:Singers)
RETURN singer.SingerId AS singer, featured.SingerId AS featured
```

The *recursive union* operator filters the `  Singers  ` table to find the singer with the given `  SingerId  ` . This is the base input to the *recursive union* . The recursive input to the *recursive union* comprises a [*distributed cross apply*](#distributed_cross_apply) or other join operator for other queries that repeatedly joins the `  Collaborations  ` table with the results of the previous iteration of the join. The rows from the base input form the zeroth iteration. At each iteration, the output of the iteration is stored by the *recursive spool scan* . Rows from the *recursive spool scan* are joined with the `  Collaborations  ` table on `  spoolscan.featuredSingerId = Collaborations.SingerId  ` . Recursion terminates when two iterations are complete, since that's the specified upper bound in the query.

## N-ary operators

An *N-ary* operator is an operator that has more than two relational children. The following operators are N-ary operators:

### Union all

A *union all* operator combines all row sets of its children without removing duplicates. Union all operators receive their input from [union input](#union_input) operators that are distributed across multiple servers. The union all operator requires that its inputs have the same schema, that's, the same set of data types for each column.

For example, using this query:

``` text
SELECT 1 a,
       2 b
UNION ALL
SELECT 3 a,
       4 b
UNION ALL
SELECT 5 a,
       6 b;
```

The row type for the children consists of two integers.

These are the results:

<table>
<thead>
<tr class="header">
<th>a</th>
<th>b</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>2</td>
</tr>
<tr class="even">
<td>3</td>
<td>4</td>
</tr>
<tr class="odd">
<td>5</td>
<td>6</td>
</tr>
</tbody>
</table>

This is the execution plan:

The union all operator combines its input rows, and in this example it sends the results to a [serialize result](#serialize_result) operator.

A query such as the following would succeed, because the same set of data types is used for each column, even though the children use different variables for the column names:

``` text
SELECT 1 a,
       2 b
UNION ALL
SELECT 3 c,
       4 e;
```

A query such as the following wouldn't succeed, because the children use different data types for the columns:

``` text
SELECT 1 a,
       2 b
UNION ALL
SELECT 3 a,
  'This is a string' b;
```

## Scalar subqueries

A *scalar subquery* is a SQL sub-expression that's part of a scalar expression. Spanner attempts to remove scalar subqueries whenever possible. In certain scenarios, however, plans can explicitly contain scalar subqueries.

For example, using this query:

``` text
SELECT firstname,
  IF(firstname = 'Alice', (SELECT Count(*)
                          FROM   songs
                          WHERE  duration > 300), 0)
FROM   singers;
```

This is the SQL sub-expression:

``` text
SELECT Count(*)
FROM   songs
WHERE  duration > 300;
```

These are the results (of the complete query):

<table>
<thead>
<tr class="header">
<th>FirstName</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Alice</td>
<td>1</td>
</tr>
<tr class="even">
<td>Catalina</td>
<td>0</td>
</tr>
<tr class="odd">
<td>David</td>
<td>0</td>
</tr>
<tr class="even">
<td>Lea</td>
<td>0</td>
</tr>
<tr class="odd">
<td>Marc</td>
<td>0</td>
</tr>
</tbody>
</table>

This is the execution plan:

The execution plan contains a scalar subquery, shown as **Scalar Subquery** , over an [aggregate](#aggregate) operator.

Spanner sometimes converts scalar subqueries into another operator such as a join or cross apply, to possibly improve performance.

For example, using this query:

``` text
SELECT *
FROM   songs
WHERE  duration = (SELECT Max(duration)
                   FROM   songs);
```

This is the SQL sub-expression:

``` text
SELECT MAX(Duration)
FROM Songs;
```

These are the results (of the complete query):

<table>
<thead>
<tr class="header">
<th>SingerId</th>
<th>AlbumId</th>
<th>TrackId</th>
<th>SongName</th>
<th>Duration</th>
<th>SongGenre</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>2</td>
<td>1</td>
<td>6</td>
<td>Nothing Is The Same</td>
<td>303</td>
<td>BLUES</td>
</tr>
</tbody>
</table>

This is the execution plan:

The execution plan doesn't contain a scalar subquery because Spanner converted the scalar subquery to a cross apply.

## Array subqueries

An *array subquery* is similar to a scalar subquery, except that the subquery is allowed to consume more than one input row. The consumed rows are converted to a single scalar output array that contains one element per consumed input row.

For example, using this query:

``` text
SELECT a.albumid,
       array
       (
              select concertdate
              FROM   concerts
              WHERE  concerts.singerid = a.singerid)
FROM   albums AS a;
```

This is the subquery:

``` text
SELECT concertdate
FROM   concerts
WHERE  concerts.singerid = a.singerid;
```

The results of the subquery for each `  AlbumId  ` are converted into an array of `  ConcertDate  ` rows against that `  AlbumId  ` . The execution plan contains an array subquery, shown as **Array Subquery** , above a distributed union operator:

## Distributed operators

The operators described previously on this page execute within the boundaries of a single machine. *Distributed operators* execute across multiple servers.

The following operators are distributed operators:

  - [Distributed union](#distributed-union)
  - [Distributed merge union](#distributed-merge-union)
  - [Distributed cross apply](#distributed-cross-apply)
  - [Distributed outer apply](#distributed-outer-apply)
  - [Apply mutations](#apply-mutations2)

The distributed union operator is the primitive operator from which distributed cross apply and distributed outer apply are derived.

Distributed operators appear in execution plans with a **distributed union** variant on top of one or more **local distributed union** variants. A distributed union variant performs the remote distribution of subplans. A local distributed union variant is on top of each of the scans performed for the query, as shown in this execution plan:

The local distributed union variants ensure stable query execution when restarts occur for dynamically changing split boundaries.

Whenever possible, a distributed union variant has a split predicate that results in split pruning, meaning the remote servers execute subplans on only the splits that satisfy the predicate. This improves both latency and overall query performance.

### Distributed union

A *distributed union* operator conceptually divides one or more tables into multiple [splits](/spanner/docs/schema-and-data-model#database-splits) , remotely evaluates a subquery independently on each split, and then unions all results.

For example, using this query:

``` text
SELECT s.songname,
       s.songgenre
FROM   songs AS s
WHERE  s.singerid = 2
       AND s.songgenre = 'ROCK';
```

These are the results:

<table>
<thead>
<tr class="header">
<th>SongName</th>
<th>SongGenre</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Starting Again</td>
<td>ROCK</td>
</tr>
<tr class="even">
<td>The Second Time</td>
<td>ROCK</td>
</tr>
<tr class="odd">
<td>Fight Story</td>
<td>ROCK</td>
</tr>
</tbody>
</table>

This is the execution plan:

The distributed union operator sends subplans to remote servers, which perform a table [scan](#scan) across splits that satisfy the query's predicate `  WHERE s.SingerId = 2 AND s.SongGenre = 'ROCK'  ` . A [serialize result](#serialize_result) operator computes the `  SongName  ` and `  SongGenre  ` values from the rows returned by the table scans. The distributed union operator then returns the combined results from the remote servers as the SQL query results.

### Distributed merge union

The *distributed merge union* operator distributes a query across multiple remote servers. It then combines the query results to produce a sorted result, known as a *distributed merge sort* .

A distributed merge union executes the following steps:

1.  The root server sends a subquery to each remote server that hosts a [split](/spanner/docs/schema-and-data-model#database-splits) of the queried data. The subquery includes instructions that results are sorted in a specific order.

2.  Each remote server executes the subquery on its split, then sends the results back in the requested order.

3.  The root server merges the sorted subquery to produce a completely sorted result.

Distributed merge union is turned on, by default, for Spanner Version 3 and later.

### Distributed cross apply

A *distributed cross apply* (DCA) operator extends the [cross apply](#cross-apply) operator by executing across multiple servers. The DCA input side groups *batches* of rows (unlike a regular cross apply operator, which acts on only one input row at a time). The DCA map side is a set of cross apply operators that execute on remote servers.

For example, using this query:

``` text
SELECT albumtitle
FROM   songs
       JOIN albums
         ON albums.albumid = songs.albumid;
```

The results are in the format:

<table>
<thead>
<tr class="header">
<th>AlbumTitle</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Green</td>
</tr>
<tr class="even">
<td>Nothing To Do With Me</td>
</tr>
<tr class="odd">
<td>Play</td>
</tr>
<tr class="even">
<td>Total Junk</td>
</tr>
<tr class="odd">
<td>Green</td>
</tr>
</tbody>
</table>

This is the execution plan:

The DCA input contains an index [scan](#scan) on the `  SongsBySingerAlbumSongNameDesc  ` index that batches rows of `  AlbumId  ` . The map side for this cross apply operator is an index scan on the index `  AlbumsByAlbumTitle  ` , subject to the predicate of `  AlbumId  ` in the input row matching the `  AlbumId  ` key in the `  AlbumsByAlbumTitle  ` index. The mapping returns the `  SongName  ` for the `  SingerId  ` values in the batched input rows.

To summarize the DCA process for this example, the DCA's input is the batched rows from the `  Albums  ` table, and the DCA's output is the application of these rows to the map of the index scan.

### Distributed outer apply

A *distributed outer apply* operator extends the [outer apply operator](#outer-apply) by executing over multiple servers, similar to the way a distributed cross apply operator extends a cross apply operator.

For example, using this query:

``` text
SELECT lastname,
       concertdate
FROM   singers LEFT OUTER join@{JOIN_TYPE=APPLY_JOIN} concerts
ON singers.singerid=concerts.singerid;
```

The results are in the format:

<table>
<thead>
<tr class="header">
<th>LastName</th>
<th>ConcertDate</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Trentor</td>
<td>2014-02-18</td>
</tr>
<tr class="even">
<td>Smith</td>
<td>2011-09-03</td>
</tr>
<tr class="odd">
<td>Smith</td>
<td>2010-06-06</td>
</tr>
<tr class="even">
<td>Lomond</td>
<td>2005-04-30</td>
</tr>
<tr class="odd">
<td>Martin</td>
<td>2015-11-04</td>
</tr>
<tr class="even">
<td>Richards</td>
<td></td>
</tr>
</tbody>
</table>

This is the execution plan:

### Apply mutations

An *apply mutations* operator applies the mutations from a [Data Manipulation Statement](/spanner/docs/dml-tasks) (DML) to the table. It's the top operator in a query plan for a DML statement.

For example, using this query:

``` text
DELETE FROM singers
WHERE  firstname = 'Alice';
```

These are the results:

``` text
 4 rows deleted  This statement deleted 4 rows and did not return any rows.
```

This is the execution plan:

## Additional information

This section describes items that are not standalone operators, but instead execute tasks to support one or more of the operators previously listed. The items described here are technically operators, but they're not separated operators in your query plan.

### Struct constructor

A *struct constructor* creates a *struct* , or a collection of fields. It typically creates a struct for rows that result from a compute operation. A struct constructor isn't a standalone operator. Instead, it appears in [compute struct](#compute_struct) operators or [serialize result](#serialize_result) operators.

For a compute struct operation, the struct constructor creates a struct so columns for the computed rows can use a single variable reference to the struct.

For a serialize result operation, the struct constructor creates a struct to serialize the results.

For example, using this query:

``` text
SELECT IF(TRUE, struct(1 AS A, 1 AS B), struct(2 AS A , 2 AS B)).A;
```

These are the results:

<table>
<thead>
<tr class="header">
<th>A</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
</tr>
</tbody>
</table>

This is the execution plan:

In the execution plan, struct constructors appear inside a serialize result operator.
