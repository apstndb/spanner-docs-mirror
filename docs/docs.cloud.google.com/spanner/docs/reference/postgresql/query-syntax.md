This page defines the syntax of the SQL statement supported for PostgreSQL-dialect databases.

## Notations used in the syntax

  - Square brackets `  [ ]  ` indicate optional clauses.
  - Curly braces `  { }  ` enclose a set of options.
  - The vertical bar `  |  ` indicates a logical OR.
  - A comma followed by an ellipsis indicates that the preceding `  item  ` can repeat in a comma-separated list.
      - `  item [, ...]  ` indicates one or more items, and
      - `  [item, ...]  ` indicates zero or more items.
  - Purple-colored text, such as `  item  ` , marks Spanner extensions to open source PostgreSQL.
  - Parentheses `  ( )  ` indicate literal parentheses.
  - A comma `  ,  ` indicates the literal comma.
  - Angle brackets `  <>  ` indicate literal angle brackets.
  - Uppercase words, such as `  INSERT  ` , are keywords.

## CALL

Use the `  CALL  ` statement to invoke a stored system procedure.

``` text
CALL procedure_name (procedure_argument[, …])
```

`  CALL  ` executes a stored system procedure ***procedure\_name*** . You can't create your own stored system procedure. You can only use system procedures. For more information, see [stored system procedures](/spanner/docs/reference/postgresql/stored-procedures-pg) .

#### Parameters

The `  CALL  ` statement uses the following parameters:

  - ***procedure\_name***  
    The name of the [stored system procedure](/spanner/docs/reference/postgresql/stored-procedures-pg) .
  - ***procedure\_argument***  
    An argument expression for the stored system procedure call.

## SELECT

Use the `  SELECT  ` statement to retrieve data from a database.

``` text
[ /*@ hint_expression [, ...] */ ] [ WITH cte[, ...] ] [, ...] ] select

where cte is:

     cte_name AS ( select )

and select is:

    SELECT select-list
        [ FROM from_item [, ...] ]
        [ WHERE condition ]
        [ GROUP BY grouping_element [, ...] ]
        [ HAVING condition ]
        [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
        [ ORDER BY expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...] ]
        [ LIMIT count ]
        [ OFFSET start ]
        [ FOR UPDATE ]

and select-list is:

    [ { ALL | DISTINCT } ] { * | expression [ [ AS ] output_name ] [, ...] }

and from_item is one of:

    table_name [ /*@ table_hint_expression [, ...] */ ]
         [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    ( select ) [ AS ] alias [ ( column_alias [, ...] ) ]
    from_item join_type [ /*@ join_hint_expression [, ...] */ ]
        from_item [ ON join_condition | USING ( join_column [, ...] ) ]
    from_item unnest_operator

and join_type is one of:

    [ INNER ] JOIN
    LEFT [ OUTER ] JOIN
    RIGHT [ OUTER ] JOIN
    FULL [ OUTER ] JOIN
    CROSS JOIN

and grouping_element is one of:

    ( )
    expression
    ( expression [, ...] )

and hint_expression is one of:

    statement_hint_key = statement_hint_value
    table_hint_key = table_hint_value
    join_hint_key = join_hint_value

and table_hint_expression is:

    table_hint_key = table_hint_value

and join_hint_expression is:

    join_hint_key = join_hint_value

and statement_hint_key is:

    USE_ADDITIONAL_PARALLELISM | LOCK_SCANNED_RANGES | SCAN_METHOD |
    EXECUTION_METHOD | ALLOW_TIMESTAMP_PREDICATE_PUSHDOWN

and table_hint_key is:

    FORCE_INDEX | GROUPBY_SCAN_OPTIMIZATION | SCAN_METHOD

and join_hint_key is:

    FORCE_JOIN_ORDER | JOIN_METHOD | HASH_JOIN_BUILD_SIDE | BATCH_MODE
```

### Common table expressions (CTEs)

***cte\_name*** AS ( ***select*** )

A common table expression (CTE) includes a CTE name and `  SELECT  ` statement.

  - A CTE cannot reference itself.
  - A CTE can be referenced by the query expression that contains the `  WITH  ` clause, but rules apply. Those rules are described later in this topic.

#### Examples

In this example, a `  WITH  ` clause defines two CTEs that are referenced in the related set operation, where one CTE is referenced by each of the set operation's input query expressions:

``` text
WITH subQ1 AS (SELECT SchoolID FROM Roster),
     subQ2 AS (SELECT OpponentID FROM PlayerStats)
SELECT * FROM subQ1
UNION ALL
SELECT * FROM subQ2
```

`  WITH  ` is not supported in a subquery. This returns an error:

``` text
SELECT account
FROM (
  WITH result AS (SELECT * FROM NPCs)
  SELECT *
  FROM result)
```

The `  WITH  ` clause is not supported in DML statements.

Temporary tables defined by the `  WITH  ` clause are stored in memory. Spanner dynamically allocates memory for all temporary tables created by a query. If the available resources are not sufficient then the query will fail.

#### CTE rules and constraints

Common table expressions (CTEs) can be referenced inside the query expression that contains the `  WITH  ` clause.

Here are some general rules and constraints to consider when working with CTEs:

  - Each CTE in the same `  WITH  ` clause must have a unique name.
  - A CTE defined in a `  WITH  ` clause is only visible to other CTEs in the same `  WITH  ` clause that were defined after it.
  - A local CTE overrides an outer CTE or table with the same name.
  - A CTE on a subquery may not reference correlated columns from the outer query.

#### CTE visibility

References between CTEs in the `  WITH  ` clause can be backward references, but not forward references.

The following is what happens when you have two CTEs that reference themselves or each other in a `  WITH  ` clause. Assume that A is the first CTE and B is the second CTE in the clause:

  - A references A = Invalid
  - A references B = Invalid
  - B references A = Valid
  - A references B references A = Invalid (cycles are not allowed)

This produces an error. A cannot reference itself because self-references are not supported:

``` text
WITH
  A AS (SELECT 1 AS n UNION ALL (SELECT n + 1 FROM A WHERE n < 3))
SELECT * FROM A

-- Error
```

This produces an error. A cannot reference B because references between CTEs can go backwards but not forwards:

``` text
WITH
  A AS (SELECT * FROM B),
  B AS (SELECT 1 AS n)
SELECT * FROM B

-- Error
```

B can reference A because references between CTEs can go backwards:

``` text
WITH
  A AS (SELECT 1 AS n),
  B AS (SELECT * FROM A)
SELECT * FROM B

+---+
| n |
+---+
| 1 |
+---+
```

This produces an error. `  A  ` and `  B  ` reference each other, which creates a cycle:

``` text
WITH
  A AS (SELECT * FROM B),
  B AS (SELECT * FROM A)
SELECT * FROM B

-- Error
```

### `     FOR UPDATE    ` clause

``` text
SELECT
...
FOR UPDATE;
```

In [serializable isolation](/spanner/docs/isolation-levels#serializable) , when you use the `  SELECT  ` query to scan a table, add a `  FOR UPDATE  ` clause to enable exclusive locks at the row-and-column granularity level, otherwise known as cell-level. The lock remains in place for the lifetime of the read-write transaction. During this time, the `  FOR UPDATE  ` clause prevents other transactions from modifying the locked cells until the current transaction completes. For more information, see [Use SELECT FOR UPDATE in serializable isolation](/spanner/docs/use-select-for-update-serializable) .

Unlike in serializable isolation, `  FOR UPDATE  ` doesn't acquire locks under repeatable read isolation. For more information, see [Use SELECT FOR UPDATE in repeatable read isolation](/spanner/docs/use-select-for-update-repeatable-read) .

To be consistent with other PostgreSQL-dialect databases, you can't use the `  FOR UPDATE  ` clause with set operators `  UNION  ` , `  INTERSECT  ` and `  EXCEPT  ` , combined with `  GROUP BY  ` , `  HAVING  ` and `  DISTINCT  ` clauses, and aggregate functions like `  array_agg()  ` .

Example:

``` text
SELECT marketingbudget
FROM albums
WHERE singerid = 1 and albumid = 1
FOR UPDATE;

UPDATE albums
SET marketingbudget = 100000
WHERE singerid = 1 and albumid = 1;
```

You can't use the `  FOR UPDATE  ` clause in the following ways:

  - In combination with the [`  LOCK_SCANNED_RANGES  ` hint](#statement-hints)
  - In read-only transactions
  - Within DDL statements

For more information, see [Use SELECT FOR UPDATE](/spanner/docs/use-select-for-update#unsupported-use-cases) .

## Spanner query hint extensions to open source PostgreSQL

Spanner has extensions for [statement hints](#statement-hints) , [table hints](#table-hints) , and [join hints](#join-hints) .

### Statement hints

``` text
[ /*@ statement_hint_key = statement_hint_value [, ...] */ ]

where statement_hint_key is:

    USE_ADDITIONAL_PARALLELISM | LOCK_SCANNED_RANGES | SCAN_METHOD |
    EXECUTION_METHOD | ALLOW_TIMESTAMP_PREDICATE_PUSHDOWN
```

Spanner supports the following statement hints as extensions to open source PostgreSQL.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Hint key</th>
<th>Possible values</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       USE_ADDITIONAL_PARALLELISM      </code></td>
<td><code dir="ltr" translate="no">       TRUE      </code> |<br />
<code dir="ltr" translate="no">       FALSE      </code> (default)</td>
<td>If <code dir="ltr" translate="no">       TRUE      </code> , the execution engine favors using more parallelism when possible.
<p>Because this can reduce resources available to other operations, you may want to avoid this hint if you run latency-sensitive operations on the same instance.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       LOCK_SCANNED_RANGES      </code></td>
<td><code dir="ltr" translate="no">       exclusive      </code> |<br />
<code dir="ltr" translate="no">       shared      </code> (default)</td>
<td>Use this hint to request an exclusive lock on a set of ranges scanned by a transaction. Acquiring an exclusive lock helps in scenarios when you observe high write contention, that is, you notice that multiple transactions are concurrently trying to read and write to the same data, resulting in a large number of aborts.
<p>Without the hint, it's possible that multiple simultaneous transactions will acquire shared locks, and then try to upgrade to exclusive locks. This will cause a deadlock, because each transaction's shared lock is preventing the other transaction(s) from upgrading to exclusive. Spanner aborts all but one of the transactions.</p>
<p>When requesting an exclusive lock using this hint, one transaction acquires the lock and proceeds to execute, while other transactions wait their turn for the lock. Throughput is still limited because the conflicting transactions can only be performed one at a time, but in this case Spanner is always making progress on one transaction, saving time that would otherwise be spent aborting and retrying transactions.</p>
<p>This hint is supported on all statement types, both query and DML.</p>
<p>Spanner always enforces <a href="/spanner/docs/transactions#serializability_and_external_consistency">serializability</a> . Lock mode hints can affect which transactions wait or abort in contended workloads, but don't change the isolation level.</p>
<p>Because this is just a hint, it shouldn't be considered equivalent to a mutex. In other words, you shouldn't use Spanner exclusive locks as a mutual exclusion mechanism for the execution of code outside of Spanner. For more information, see <a href="/spanner/docs/transactions#locking">Locking</a> .</p>
<p>You can't use both the <code dir="ltr" translate="no">        FOR UPDATE       </code> clause and the <code dir="ltr" translate="no">        LOCK_SCANNED_RANGES       </code> hint in the same query. An error is returned. For more information, see <a href="/spanner/docs/use-select-for-update#unsupported-use-cases">Use SELECT FOR UPDATE</a> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SCAN_METHOD      </code></td>
<td><code dir="ltr" translate="no">       AUTO      </code> (default) |<br />
<code dir="ltr" translate="no">       BATCH      </code> |<br />
<code dir="ltr" translate="no">       ROW      </code></td>
<td>Use this hint to enforce the query scan method.
<p>The default Spanner scan method is <code dir="ltr" translate="no">        AUTO       </code> (automatic). The <code dir="ltr" translate="no">        AUTO       </code> setting specifies that depending on the heuristics of the query, batch or row-oriented query processing might be used to improve query performance. If you want to change the default scanning method, you can use a statement hint to enforce the <code dir="ltr" translate="no">        BATCH       </code> -oriented or <code dir="ltr" translate="no">        ROW       </code> -oriented processing method. You can't manually set the scan method to <code dir="ltr" translate="no">        AUTO       </code> . However, if you remove the statement hint, then Spanner uses the <code dir="ltr" translate="no">        AUTO       </code> scan method. For more information, see <a href="/spanner/docs/sql-best-practices#optimize-scans">Optimize scans</a> .</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       EXECUTION_METHOD      </code></td>
<td><code dir="ltr" translate="no">       DEFAULT      </code> |<br />
<code dir="ltr" translate="no">       BATCH      </code> |<br />
<code dir="ltr" translate="no">       ROW      </code></td>
<td>Use this hint to enforce the query execution method.
<p>The default Spanner query execution method is <code dir="ltr" translate="no">        DEFAULT       </code> . The <code dir="ltr" translate="no">        DEFAULT       </code> setting specifies that batch-oriented execution might be used to improve query performance, depending on the heuristics of the query. If you want to change the default execution method, you can use a statement hint to enforce the <code dir="ltr" translate="no">        BATCH       </code> -oriented or <code dir="ltr" translate="no">        ROW       </code> -oriented execution method. You can't manually set the query execution method to <code dir="ltr" translate="no">        DEFAULT       </code> . However, if you remove the statement hint, then Spanner uses the <code dir="ltr" translate="no">        DEFAULT       </code> execution method. For more information, see <a href="/spanner/docs/sql-best-practices#optimize-query-execution">Optimize query execution</a> .</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ALLOW_TIMESTAMP_PREDICATE_PUSHDOWN      </code></td>
<td><code dir="ltr" translate="no">       TRUE      </code> |<br />
<code dir="ltr" translate="no">       FALSE      </code> (default)</td>
<td>If set to <code dir="ltr" translate="no">       TRUE      </code> , the query execution engine uses the timestamp predicate pushdown optimization technique. This technique improves the efficiency of queries that use timestamps and data with an age-based tiered storage policy. For more information, see <a href="/spanner/docs/sql-best-practices#optimize-timestamp-predicate-pushdown">Optimize queries with timestamp predicate pushdown</a> .</td>
</tr>
</tbody>
</table>

### Table hints

``` text
[ /*@ table_hint_key = table_hint_value [, ...] */ ]

where table_hint_key is:

    FORCE_INDEX | GROUPBY_SCAN_OPTIMIZATION | SCAN_METHOD
```

Spanner supports the following table hints as extensions to open source PostgreSQL.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Hint key</th>
<th>Possible values</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       FORCE_INDEX      </code></td>
<td>String. The name of an existing index in the database or <code dir="ltr" translate="no">       _BASE_TABLE      </code> to use the base table rather than an index.</td>
<td><ul>
<li>If set to the name of an index, use that index instead of the base table. If the index cannot provide all needed columns, perform a back join with the base table.</li>
<li>If set to the string <code dir="ltr" translate="no">         _BASE_TABLE        </code> , use the base table for the index strategy instead of an index. Note that this is the only valid value when <code dir="ltr" translate="no">         FORCE_INDEX        </code> is used in a statement hint expression.</li>
</ul>
<p>Note: <code dir="ltr" translate="no">        FORCE_INDEX       </code> is actually a directive, not a hint, which means an error is raised if the index does not exist.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       GROUPBY_SCAN_OPTIMIZATION      </code></td>
<td><code dir="ltr" translate="no">       TRUE      </code> |<br />
<code dir="ltr" translate="no">       FALSE      </code></td>
<td><p>The group by scan optimization can make queries faster if they use <code dir="ltr" translate="no">        GROUP BY       </code> . It can be applied if the grouping keys can form a prefix of the underlying table or index key, and if the query requires only the first row from each group.</p>
<p>The optimization is applied if the optimizer estimates that it will make the query more efficient. The hint overrides that decision. If the hint is set to <code dir="ltr" translate="no">        FALSE       </code> , the optimization is not considered. If the hint is set to <code dir="ltr" translate="no">        TRUE       </code> , the optimization will be applied as long as it is legal to do so.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SCAN_METHOD      </code></td>
<td><code dir="ltr" translate="no">       AUTO      </code> (default) |<br />
<code dir="ltr" translate="no">       BATCH      </code> |<br />
<code dir="ltr" translate="no">       ROW      </code></td>
<td>Use this hint to enforce the query scan method.
<p>By default, Spanner sets the scan method as <code dir="ltr" translate="no">        AUTO       </code> (automatic) which means depending on the heuristics of the query, batch-oriented query processing might be used to improve query performance. If you want to change the default scanning method from <code dir="ltr" translate="no">        AUTO       </code> , you can use the hint to enforce a <code dir="ltr" translate="no">        ROW       </code> or <code dir="ltr" translate="no">        BATCH       </code> oriented processing method. For more information see <a href="/spanner/docs/sql-best-practices#optimize-scans">Optimize scans</a> .</p></td>
</tr>
</tbody>
</table>

### Join hints

``` text
[ /*@ join_hint_key = join_hint_value [, ...] */ ]

where join_hint_key is:

    FORCE_JOIN_ORDER | JOIN_METHOD | HASH_JOIN_BUILD_SIDE | BATCH_MODE
```

Spanner supports the following join hints as extensions to open source PostgreSQL.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Hint key</th>
<th>Possible values</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       FORCE_JOIN_ORDER      </code></td>
<td><code dir="ltr" translate="no">       TRUE      </code> |<br />
<code dir="ltr" translate="no">       FALSE      </code> (default)</td>
<td>If set to true, use the join order that's specified in the query.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       JOIN_METHOD      </code></td>
<td><code dir="ltr" translate="no">       HASH_JOIN      </code> |<br />
<code dir="ltr" translate="no">       APPLY_JOIN      </code> |<br />
<code dir="ltr" translate="no">       MERGE_JOIN      </code> |<br />
<code dir="ltr" translate="no">       PUSH_BROADCAST_HASH_JOIN      </code></td>
<td>When implementing a logical join, choose a specific alternative to use for the underlying join method. Learn more in <a href="#join-methods">Join methods.</a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       HASH_JOIN_BUILD_SIDE      </code></td>
<td><code dir="ltr" translate="no">       BUILD_LEFT      </code> |<br />
<code dir="ltr" translate="no">       BUILD_RIGHT      </code></td>
<td>Specifies which side of the hash join is used as the build side. Can only be used with <code dir="ltr" translate="no">       JOIN_METHOD=HASH_JOIN      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       BATCH_MODE      </code></td>
<td><code dir="ltr" translate="no">       TRUE (default)      </code> |<br />
<code dir="ltr" translate="no">       FALSE      </code></td>
<td>Used to disable batched apply join in favor of row-at-a-time apply join. Can only be used with <code dir="ltr" translate="no">       JOIN_METHOD=APPLY_JOIN      </code> .</td>
</tr>
</tbody>
</table>

### Function hints

``` text
function_name() [ /*@ function_hint_key = function_hint_value [, ...] */ ]

where function_hint_key is:

    DISABLE_INLINE
```

Spanner supports the following function hints as extensions to open source PostgreSQL.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Hint key</th>
<th>Possible values</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       DISABLE_INLINE      </code></td>
<td><code dir="ltr" translate="no">       TRUE      </code> |<br />
<code dir="ltr" translate="no">       FALSE      </code> (default)</td>
<td><p>If set to true, the function is computed once instead of each time another part of a query references it.</p>
<p><code dir="ltr" translate="no">        DISABLE_INLINE       </code> works with top-level functions.</p>
<p>You can't use <code dir="ltr" translate="no">        DISABLE_INLINE       </code> with a few functions, including those that don't produce a scalar value and casting. Although you can't use <code dir="ltr" translate="no">        DISABLE_INLINE       </code> with a casting function, you can use it with the first expression inside the function.</p></td>
</tr>
</tbody>
</table>

**Examples**

In the following example, inline expressions are enabled by default for `  x  ` . `  x  ` is computed twice, once by each reference:

``` text
SELECT
  SUBSTRING(x, 2, 5) AS w,
  SUBSTRING(x, 3, 7) AS y
FROM (SELECT SHA512(z) AS x FROM t) AS subquery
```

In the following example, inline expressions are disabled for `  x  ` . `  x  ` is computed once, and the result is used by each reference:

``` text
SELECT
  SUBSTRING(x, 2, 5) AS w,
  SUBSTRING(x, 3, 7) AS y
FROM (SELECT SHA512(z) /*@ DISABLE_INLINE = TRUE */ AS x FROM t) AS subquery
```

## Join methods

Join methods are specific implementations of the various logical join types. Some join methods are available only for certain join types. The choice of which join method to use depends on the specifics of your query and of the data being queried. The best way to figure out if a particular join method helps with the performance of your query is to try the method and view the resulting [query execution plan](/spanner/docs/query-execution-plans) . See [Query Execution Operators](/spanner/docs/query-execution-operators) for more details.

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 40%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Join Method</th>
<th>Description</th>
<th>Operands</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       HASH_JOIN      </code></td>
<td>The hash join operator builds a hash table out of one side (the build side), and probes in the hash table for all the elements in the other side (the probe side).</td>
<td>Different variants are used for various join types. View the query execution plan for your query to see which variant is used. Read more about the <a href="/spanner/docs/query-execution-operators#hash_join">Hash join operator</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       APPLY_JOIN      </code></td>
<td>The apply join operator gets each item from one side (the input side), and evaluates the subquery on other side (the map side) using the values of the item from the input side.</td>
<td>Different variants are used for various join types. Cross apply is used for inner join, and outer apply is used for left joins. Read more about the <a href="/spanner/docs/query-execution-operators#cross_apply">Cross apply</a> and <a href="/spanner/docs/query-execution-operators#outer_apply">Outer apply</a> operators.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       MERGE_JOIN      </code></td>
<td>The merge join operator joins two streams of sorted data. The optimizer will add Sort operators to the plan if the data is not already providing the required sort property for the given join condition. The engine provides a distributed merge sort by default, which when coupled with merge join may allow for larger joins, potentially avoiding disk spilling and improving scale and latency.</td>
<td>Different variants are used for various join types. View the query execution plan for your query to see which variant is used. Read more about the <a href="/spanner/docs/query-execution-operators#merge_join">Merge join operator</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       PUSH_BROADCAST_HASH_JOIN      </code></td>
<td>The push broadcast hash join operator builds a batch of data from the build side of the join. The batch is then sent in parallel to all the local splits of the probe side of the join. On each of the local servers, a hash join is executed between the batch and the local data. This join is most likely to be beneficial when the input can fit within one batch, but is not strict. Another potential area of benefit is when operations can be distributed to the local servers, such as an aggregation that occurs after a join. A push broadcast hash join can distribute some aggregation where a hash join cannot.</td>
<td>Different variants are used for various join types. View the query execution plan for your query to see which variant is used. Read more about the <a href="/spanner/docs/query-execution-operators#push_broadcast_hash_join">Push broadcast hash join operator</a> .</td>
</tr>
</tbody>
</table>

## `     UNNEST    ` operator

``` text
unnest_operator:
  {
    UNNEST( array_expression )
    | UNNEST( array_path )

  }
  [ table_hint_expr ]
  [ as_alias ]

as_alias:
  [AS] alias
```

The `  UNNEST  ` operator takes an array and returns a table, with one row for each element in the array. For input arrays of most element types, the output of `  UNNEST  ` generally has one column.

Input values:

  - `  array_expression  ` : an expression that produces an array.

  - `  table_name  ` : The name of a table.

  - `  array_path  ` : The path to an `  ARRAY  ` type.
    
    Example:
    
    ``` text
    SELECT * FROM UNNEST (ARRAY[10,20,30]) as numbers;
    
    /*---------*
     | numbers |
     +---------+
     | 10      |
     | 20      |
     | 30      |
     *---------*/
    ```

  - `  alias  ` : An alias for a value table. An input array that produces a single column can have an optional alias, which you can use to refer to the column elsewhere in the query.
