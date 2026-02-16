This page describes and provides a history of the various Spanner query optimizer versions. The current default version is 8. To learn more about the query optimizer, see [Query optimizer overview](/spanner/docs/query-optimizer/overview) .

Spanner rolls out query optimizer updates as new query optimizer versions. By default, each database starts using the latest version of the optimizer no sooner than 30 days after that version has been released.

You can manage the query optimizer version that your queries use for GoogleSQL-dialect and PostgreSQL-dialect databases. Before committing to the latest version, you can compare query performance profiles between earlier versions and the latest version. To learn more, see [Manage the query optimizer](/spanner/docs/query-optimizer/manage-query-optimizer) .

## Query optimizer version history

The following is a summary of the updates made to the query optimizer in each release.

### Version 8: October 28th, 2024 (default and latest)

  - `  WITH  ` clauses are considered when making cost-based plan choices.

  - Improved performance of distributed cross apply and indexed lookup queries.

  - Improved `  JOIN  ` reordering.

  - Improved performance of queries with large `  IN (...)  ` clauses.

  - Improved `  GROUP BY  ` performance in certain cases.

  - Other improvements including more efficient handling of queries with `  LIMIT  ` , foreign keys, and index selection.

### Version 7: May 22nd, 2024

  - Added support for cost-based selection of index union plans.

  - Added support for the smart selection of seek versus scan plans based on statistics for queries that don't have seekable predicates for all key parts.

  - Added support for cost-based selection of hash joins.

### Version 6: September 11th, 2023

  - Improved limit pushing and predicate pushing through full outer joins.

  - Improved cardinality estimation and cost model.

  - Enabled cost-based optimization for DML queries.

### Version 5: July 15th, 2022

  - Improved cost model for index selection, distribution management, sort placement and `  GROUP BY  ` selection.

  - Added support for cost-based join algorithm selection that chooses between hash and apply join. Merge join still requires using a query hint.

  - Added support for cost-based join commutativity.

### Version 4: March 1st, 2022

  - Improvements to secondary index selection.
    
      - Improved secondary index usage under a join between interleaved tables.
      - Improved covering secondary index usage.
      - Improved index selection when optimizer statistics are outdated.
      - Prefer secondary indexes with predicates on leading indexed columns even if the optimizer statistics are not available or report the base table is small.

  - Introduces single pass hash join, enabled by the new hint `  hash_join_execution  ` .
    
    Join Hint:
    
    ### GoogleSQL
    
    ``` text
    SELECT ...
    FROM (...)
    JOIN@{join_method=hash_join, hash_join_execution=one_pass} (...)
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT ...
    FROM (...)
    JOIN/*@ join_method=hash_join, hash_join_execution=one_pass */ (...)
    ```
    
    The new mode is beneficial when the [build side input of the hash join](/spanner/docs/query-execution-operators#hash-join) is large. One pass hash join is expected to have better performance when you observe the following in the [query execution profile](/spanner/docs/tune-query-with-visualizer) :
    
      - The number of executions on the right child of the hash join is larger than the number of executions on the hash join operator.
      - The latency on the right child of the hash join operator is also high.
    
    By default ( `  hash_join_execution=multi_pass  ` ), if the build side input of the hash join is too large to fit in memory, the build side is split into multiple batches and we might scan the probe side multiple times. With the new mode ( `  hash_join_execution=one_pass  ` ), a hash join will spill to disk if its build side input cannot fit in memory and will always scan the probe side only once.

  - An improvement in selecting how many keys are used for seeking.

### Version 3: August 1st, 2021

  - Adds a new join algorithm, merge join, enabled using a new [JOIN METHOD](/spanner/docs/reference/standard-sql/query-syntax#join-methods) query hint value.
    
    Statement hint:
    
    ### GoogleSQL
    
    ``` text
    @{join_method=merge_join}
    SELECT ...
    ```
    
    ### PostgreSQL
    
    ``` text
    /*@ join_method=merge_join */
    SELECT ...
    ```
    
    Join hint:
    
    ### GoogleSQL
    
    ``` text
    SELECT ...
    FROM (...)
    JOIN@{join_method=merge_join} (...)
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT ...
    FROM (...)
    JOIN/*@ join_method=merge_join */ (...)
    ```

  - Adds a new join algorithm, push broadcast hash join, enabled using a new [JOIN METHOD](/spanner/docs/reference/standard-sql/query-syntax#join-methods) query hint value.
    
    Join hint:
    
    ### GoogleSQL
    
    ``` text
    SELECT ...
    FROM (...)
    JOIN@{join_method=push_broadcast_hash_join} (...)
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT ...
    FROM (...)
    JOIN/*@ join_method=push_broadcast_hash_join} */ (...)
    ```

  - Introduces the [distributed merge union](/spanner/docs/query-execution-operators#distributed-merge-union) operator, which is enabled by default when applicable. This operation improves the performance of queries.

  - A small improvement to the performance of a scan under a `  GROUP BY  ` when there is no MAX or MIN aggregate (or HAVING MAX/MAX) in the SELECT list. Prior to this change, Spanner loaded the extra non-grouped column even if it was not required by the query.
    
    For example, consider the following table:
    
    ### GoogleSQL
    
    ``` text
    CREATE TABLE myTable(
      a INT64,
      b INT64,
      c INT64,
      d INT64)
    PRIMARY KEY (a, b, c);
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE TABLE myTable(
      a bigint,
      b bigint,
      c bigint,
      d bigint,
      PRIMARY KEY(a, b, c)
    );
    ```
    
    Prior to this change, the following query would have loaded column `  c  ` even though it is not required by the query.
    
    ``` text
    SELECT a, b
    FROM myTable
    GROUP BY a, b
    ```

  - Improves the performance of some queries with `  LIMIT  ` when there is a cross apply operator introduced by joins and the query asks for sorted results with LIMIT. After this change, the optimizer applies the sorting with the limit on the input side of cross apply first.
    
    Example:
    
    ### GoogleSQL
    
    ``` text
    SELECT a2.*
    FROM Albums@{FORCE_INDEX=_BASE_TABLE} a1
    JOIN Albums@{FORCE_INDEX=_BASE_TABLE} a2 USING(SingerId)
    ORDER BY a1.AlbumId
    LIMIT 2;
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT a2.*
    FROM albums/*@ force_index=_base_table */ a1
    JOIN albums/*@ force_index=_base_table */ a2 USING(singerid)
    ORDER BY a1.albumid
    LIMIT 2;
    ```

  - Improves query performance by pushing more computations through `  JOIN  ` .
    
    Pushes more computations which may include a subquery or struct construction through join. That improves the query performance in a few ways such as: More computations can be done in a distributed fashion and more operations that depend on the pushed computations can be pushed down as well. For example, the query has a limit and the sort order depends on those computations, then the limit can be pushed through join as well.
    
    Example:
    
    ``` text
    SELECT
      t.ConcertDate,
      (
        SELECT COUNT(*) FROM UNNEST(t.TicketPrices) p WHERE p > 10
      ) AS expensive_tickets,
      u.VenueName
    FROM Concerts t
    JOIN Venues u ON t.VenueId = u.VenueId
    ORDER BY expensive_tickets
    LIMIT 2;
    ```

### Version 2: March 1st, 2020

  - Adds optimizations in index selection.
  - Improves the performance of `  REGEXP_CONTAINS  ` and `  LIKE  ` predicates under certain circumstances.
  - Improves the performance of a scan under a `  GROUP BY  ` in certain situations.

### Version 1: June 18th, 2019

  - Includes many rule based optimizations such as predicate pushdown, limit pushdown, redundant join and redundant expression removal, to name a few.

  - Uses statistics on user data to select which index to use to access each table.

## What's next

  - To learn more about the query optimizer, see [About query optimizer](/spanner/docs/query-optimizer/overview) .
  - To manage both the optimizer version and statistics package for your scenario, see [Manage the query optimizer](/spanner/docs/query-optimizer/manage-query-optimizer) .
