This page describes the Spanner query optimizer and its benefits. Query optimizer is available for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Overview

The Spanner SQL query optimizer converts a declarative SQL statement, that describes what data the query wants, into an imperative execution plan, that describes one way to precisely obtain that data. The process of transforming a declarative statement into a [query execution plan](/spanner/docs/query-execution-plans) involves performing transformations to tree structures used to represent the query. The optimizer, in the process of producing an execution plan, preserves the logical meaning of the original SQL query so that the correct rows are returned.

Another important role of the optimizer is to produce an execution plan that is efficient.

## How does the Spanner optimizer produce efficient execution plans?

Spanner's optimizer uses a combination of well-established heuristics and cost-based optimization to produce efficient plans. Some heuristics are very straightforward, such as "perform filters on remote machines rather than pulling data to the local machine". Other heuristics are more complex, but still embody the principle of moving logic that reduces data volume closer to the data. That principle is particularly important in a system that shards data across multiple machines.

Not all execution decisions can be made effectively using such fixed rules, so Spanner's optimizer also makes decisions based on an *estimated cost of alternatives* . Such cost estimates are calculated using the structure of the query, the schema of the database, and estimates of the data volume that will be produced by fragments of the query. For example, Spanner estimates how many rows of the Songs table qualify the filter `  SongGenre = "Country"  ` if that filter appears in a query. To help the computation of such estimates, Spanner periodically gathers statistics to characterize the distribution of the data in a database.

In addition, Spanner optimizes query execution by automatically determining if a row- or column-oriented processing method should be used for the query. For more information, see [Optimize scans](/spanner/docs/sql-best-practices#optimize-scans) .

To learn more about query execution plans and how they are used by Spanner to perform queries in a distributed environment, see [Query execution plans](/spanner/docs/query-execution-plans) .

## Query optimizer versioning

Over time, the Spanner query optimizer will evolve, broadening the set of choices in the query execution plan and improving the accuracy of the estimates that inform those choices, leading to more efficient query execution plans.

Spanner releases optimizer updates as new query optimizer versions to improve the efficiency of its query execution plans. To learn more about the different versions, see [Spanner query optimizer versions](/spanner/docs/query-optimizer/versions) .

## Query optimizer statistics packages

Spanner maintains statistics on the data distribution of table columns to help estimate how many rows will be produced by a query. The query optimizer uses these estimates to help choose the best query execution plan. These statistics are periodically updated by Spanner. Because the statistics are used to choose query execution plans, when the statistics are updated, it's possible for Spanner to change the query plan that it uses for a query.

**Note:** These statistics are only available to the query optimizer, and cannot be queried by the end user.

By default, databases automatically use the latest generated statistics package. You can pin your database to an earlier statistics package version. You also have the option of running individual queries with a statistics package other than the latest.

### Construct a new statistics package

Spanner automatically generates a new statistics packages every three days. To construct a new statistics package manually, use the GoogleSQL [`  ANALYZE  `](/spanner/docs/reference/standard-sql/data-definition-language#analyze-statistics) DDL statement or the PostgreSQL [`  ANALYZE  `](/spanner/docs/reference/postgresql/data-definition-language#analyze-statistics) DDL statement.

After significant changes to your database's data or schema, constructing a new statistics package can benefit query performance. As a best practice, construct a new statistics package if the following occurs:

  - The database processes a large amount of inserts, updates, or deletes.
  - You add a new index to the database.
  - You add a new column to a table.

Running an `  ANALYZE  ` DDL statement [updates your schema](/spanner/docs/schema-updates) , initiates a [long-running operation](/spanner/docs/manage-long-running-operations#get_the_status_of_a_long-running_database_operation) , and cancels the creation of any automatically triggered statistics.

After Spanner finishes executing the statement, it takes up to ten minutes for the query optimizer to account for a new statistics package in its query planning.

### Garbage collection of statistics packages

Statistics packages in Spanner are kept for a period of 30 days since their creation, after which they are subject to garbage collection.

The Spanner built-in [`  INFORMATION_SCHEMA.SPANNER_STATISTICS  `](/spanner/docs/information-schema#information_schemaspanner_statistics) table contains a list of available statistics packages. Each row in this table lists a statistics package by name, and the name contains the creation timestamp of the given package. Each entry also contains a field called `  ALLOW_GC  ` which defines whether a package can be garbage collected or not.

You can pin your entire database to any one of the packages listed in that table. The pinned statistics package won't be garbage collected and the value of `  ALLOW_GC  ` is ignored as long as the database is pinned to this package. To use a particular statistics package for an individual query, the package must be listed with `  ALLOW_GC=FALSE  ` or pinned. This prevents queries from failing after the statistics package has been garbage collected. You can change the value of `  ALLOW_GC  ` using the GoogleSQL [`  ALTER STATISTICS  `](/spanner/docs/reference/standard-sql/data-definition-language#alter-statistics) or PostgreSQL [`  ALTER STATISTICS  `](/spanner/docs/query-optimizer/manage-query-optimizer#statement-hint) DDL statement.

### Package retention and Personally Identifiable Information (PII)

A statistics package contains histograms of the column data, as per standard industry practice. This helps the query optimizer select the optimal query plans. The histogram is constructed using a small sample of values. This small dataset can potentially contain PII.

Spanner creates a new statistics package on a regular basis and retains it for 30 days by default. Thus a small sample of values deleted from the database may be retained for additional 30 days in statistics histograms. Statistics packages pinned with `  optimizer_statistics_package  ` database option or packages with `  ALLOW_GC=FALSE  ` option won't be garbage collected. Histograms in these packages may contain values deleted from the database for a longer period. In addition, the content of statistics packages are included in database backups.

The optimizer statistics is stored encrypted in the same way as user data.

The total amount of storage required for these packages is usually less than 100 MB, and does count towards your total storage costs.

## What's next

  - To learn more about the history of the query optimizer, see [Query optimizer version history](/spanner/docs/query-optimizer/versions) .
  - To manage both the optimizer version and statistics package for your scenario, see [Manage the query optimizer](/spanner/docs/query-optimizer/manage-query-optimizer) .
