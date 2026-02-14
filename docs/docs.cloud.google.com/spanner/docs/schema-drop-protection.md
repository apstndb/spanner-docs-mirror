This page describes how to protect schema objects such as tables, indexes, and columns from accidental deletion.

## Understand schema object drop safety

*Spanner's schema object drop protection* relies on schema object access statistics to determine whether an object is safe to drop. If the system determines the object is unsafe, it rejects the user's request to drop the schema object. Dropping such an object might cause client jobs that still access the object to fail. Additionally, restoring a dropped object without incurring data loss is very difficult or impossible.

To this end, Spanner tracks access statistics for each schema object. This access includes DML inserts, updates, or deletes, SQL queries, API writes, and API reads. Using these access statistics, Spanner determines whether a schema object might still be in use. If an object has been recently accessed, the object is likely still in use.

## Enable schema object drop protection

To enable drop protection, set the inactivity duration for drop protection to a non-zero duration value, such as "24h" or "2d". The minimum duration is "0s" and the maximum duration is "7d". You should set the inactivity duration to "24h" or a longer.

If you set the inactivity duration to "24h", you can drop a schema object only if the object wasn't accessed in the last 24 hours.

### GoogleSQL

For more information, see [ALTER DATABASE](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) .

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME` SET OPTIONS ( schema_drop_protection_inactivity_period="DURATION" )'
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_ID : the identifier of your database instance.
  - DURATION : between "0s" (inclusive) and "7d" (inclusive). Recommended: "24h".

### PostgreSQL

For more information, see [ALTER DATABASE](/spanner/docs/reference/postgresql/data-definition-language#alter-database) .

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME" SET spanner.schema_drop_protection_inactivity_period="DURATION" '
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_ID : the identifier of your database instance.
  - DURATION : between "0s" (inclusive) and "7d" (inclusive). Recommended: "24h".

## Disable schema object drop protection

To disable drop protection, set the inactivity duration to a zero duration value, "0s", or set the option value to null.

### GoogleSQL

For more information, see [ALTER DATABASE](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) .

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME` SET OPTIONS ( schema_drop_protection_inactivity_period=null )'
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_ID : the identifier of your database instance.

### PostgreSQL

For more information, see [ALTER DATABASE](/spanner/docs/reference/postgresql/data-definition-language#alter-database) .

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME` SET spanner.schema_drop_protection_inactivity_period=null'
```

Replace the following:

  - DATABASE\_NAME : the name of your database.
  - INSTANCE\_ID : the identifier of your database instance.

## Caveats

Schema object drop protection isn't activated in the following scenarios:

1.  You created the database within the last hour.
2.  You created the schema object within the inactivity duration.
3.  The object has low-frequency read accesses (fewer than 10 per hour).

The default read access threshold is 10 per hour, but you can override the default using the following database option:

### GoogleSQL

For more information, see [ALTER DATABASE](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) .

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME` SET OPTIONS ( schema_drop_protection_usage_lowerbound=<threshold> )'
```

### PostgreSQL

For more information, see [ALTER DATABASE](/spanner/docs/reference/postgresql/data-definition-language#alter-database) .

``` text
gcloud spanner databases ddl update DATABASE_NAME \
--instance=INSTANCE_ID \
--ddl='ALTER DATABASE `DATABASE_NAME` SET spanner.schema_drop_protection_usage_lowerbound=<threshold> '
```

Access statistics aren't collected for the following cases:

1.  Failed accesses (queries, DML, etc.) aren't counted towards user access.
2.  System accesses aren't counted.
3.  Write accesses (including DML inserts and updates) are ignored for indexes.
4.  Write accesses are ignored for stored generated columns.

## Troubleshooting

If a schema object fails to drop and you can't identify what's accessing it, see the following:

  - [Table operations statistics](/spanner/docs/introspection/table-operations-statistics)
  - [Column operations statistics](/spanner/docs/introspection/column-operations-statistics)

## What's next

Learn how to [prevent database deletion](/spanner/docs/prevent-database-deletion) .
