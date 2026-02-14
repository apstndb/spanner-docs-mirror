**Preview — [Spanner columnar engine](/spanner/docs/columnar-engine)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This page describes how to enable Spanner columnar engine on a database, table, or index, and accelerate file format generation.

## Enable Spanner columnar engine

Spanner columnar engine is enabled using a `  SET OPTIONS  ` clause with the `  columnar_policy  ` option. You can apply this option when you create or alter `  DATABASE  ` , `  TABLE  ` , or `  INDEX  ` schema objects. `  SEARCH INDEX  ` and `  VECTOR INDEX  ` schema objects are never in columnar format.

Lower levels in a database and table hierarchy inherit the `  columnar_policy  ` from its parent. You can change this setting at lower levels.

The `  columnar_policy  ` option has the following flags:

  - `  'enabled'  ` or `  'disabled'  ` turns the columnar engine on or off for the specific schema object.
  - `  NULL  ` (default) uses the columnar engine policy from the parent object, if one exists. `  NULL  ` clears previous settings on a table object.

You can also omit `  OPTIONS  ` to inherit the `  columnar_policy  ` from the parent object.

The following example shows how to:

  - Create a database with the columnar policy enabled.
  - Define a `  Singers  ` table that inherits the columnar policy from the database (omit the `  columnar_policy = NULL  ` for the table option).
  - Define a `  Concerts  ` table with the columnar policy explicitly disabled.

<!-- end list -->

``` text
CREATE DATABASE Music;

ALTER DATABASE Music SET OPTIONS (columnar_policy = 'enabled');

CREATE TABLE Singers(
  SingerId INT64 NOT NULL,
  FirstName STRING(1024),
  LastName STRING(1024),
  BirthDate DATE,
  Status STRING(1024),
  LastUpdated TIMESTAMP,)
  PRIMARY KEY(SingerId);

CREATE TABLE Concerts(
  VenueId INT64 NOT NULL,
  SingerId INT64 NOT NULL,
  ConcertDate DATE NOT NULL,
  BeginTime TIMESTAMP,
  EndTime TIMESTAMP,)
  PRIMARY KEY(VenueId, SingerId, ConcertDate),
  OPTIONS (columnar_policy = 'disabled');
```

You can also use `  ALTER TABLE  ` with the `  SET OPTIONS  ` clause to enable or disable the `  columnar_policy  ` on a table. The following example shows how to disable the policy in the `  Singers  ` table:

``` text
ALTER TABLE Singers SET OPTIONS (columnar_policy = 'disabled');
```

**Note:** Enabling Spanner columnar engine increases the storage usage of the target database or table (depending on the enabling option used) by approximately 60%. The exact increase depends on the type of data and its compressibility properties. It's important to ensure that the Spanner instance has sufficient storage capacity to accommodate the increase in storage usage. For more information, see [Database limits](/spanner/quotas#database-limits) .

## Columnar file format generation

Spanner generates the columnar file format at compaction time. Compaction is a background process that typically is spread out over multiple days, but it might happen sooner if the size of the database grows substantially. For more information, see [Optimal columnar coverage](/spanner/docs/columnar-engine#optimal_columnar_coverage) .

If you create a new database without data and enable columnar engine, Spanner stores data in columnar format as you insert it and as compactions occur in the background.

Columnar data format isn't generated for backups.

When you enable Spanner columnar engine on an existing database that has data in it, Spanner provides a mechanism to manually trigger compactions. For more information, see [Manually trigger a data compaction](/spanner/docs/manual-data-compaction#trigger-compaction) .

## What's next

  - Learn about [columnar engine](/spanner/docs/columnar-engine) .
  - Learn how to [query columnar data](/spanner/docs/query-columnar-data) .
  - Learn how to [monitor columnar engine](/spanner/docs/monitor-columnar-engine) .
