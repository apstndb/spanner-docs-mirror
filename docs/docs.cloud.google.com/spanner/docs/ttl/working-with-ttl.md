This page discusses how to use time to live (TTL) on Spanner tables in GoogleSQL-dialect databases and PostgreSQL-dialect databases. For more information, see [About TTL](/spanner/docs/ttl) .

## Before you begin

Before you begin, follow these best practices.

### Enable backup and point-in-time recovery

Before adding TTL to your table, we recommend enabling [Spanner backup and restore](/spanner/docs/backup) . This lets you fully restore a database in case you accidentally delete your data with the TTL policy.

If you've enabled [point-in-time recovery](/spanner/docs/pitr) , you can view and restore deleted data—without a full restore from backup—if it's within the configured [version retention period](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) . For information on reading data in the past, see [Perform a stale read](/spanner/docs/reads#perform-stale-read) .

### Clean up old data

If this is the first time you're using TTL and you expect the first run to delete many rows, consider first cleaning up old data manually using [partitioned DML](/spanner/docs/dml-partitioned) . This gives you more control over the resource usage, instead of leaving it to the TTL background process. TTL runs at a low priority, ideal for incremental clean-up. However, this likely lengthens the time it takes to delete the initial set of rows in a busy database because Spanner's internal work scheduler prioritizes other work, such as user queries.

### Verify your conditions

For GoogleSQL tables, if you want to verify the data that the row deletion policy affects before enabling TTL, you can query your table using the same conditions. For example:

### GoogleSQL

``` text
  SELECT COUNT(*)
  FROM CalculatedRoutes
  WHERE TIMESTAMP_ADD(CreatedAt, INTERVAL 30 DAY) < CURRENT_TIMESTAMP();
```

### Required permissions

To change the database's schema, you must have the **spanner.databases.updateDdl** permission. For details, see [Access control for Spanner](/spanner/docs/iam#databases) .

## Create a row deletion policy

### GoogleSQL

To create a row deletion policy using GoogleSQL, you can define a `  ROW DELETION POLICY  ` clause when you create a new table, or add a policy to an existing table. This clause contains an expression of a column and an interval.

To add a policy at the time of table creation, do the following:

``` text
CREATE TABLE MyTable(
Key INT64,
CreatedAt TIMESTAMP,
) PRIMARY KEY (Key),
ROW DELETION POLICY (OLDER_THAN(timestamp_column, INTERVAL num_days DAY));
```

Replace the following:

  - `  timestamp_column  ` must be an existing column with type `  TIMESTAMP  ` . Columns with [commit timestamps](/spanner/docs/commit-timestamp) are valid, as are [generated columns](#ttl_on_generated_columns) . However, you can't specify a generated column that references a commit timestamp column.

  - `  num_days  ` is the number of days past the timestamp in the `  timestamp_column  ` in which the row is marked for deletion. The value must be a non-negative integer and `  DAY  ` is the only supported unit.

To add a policy to an existing table, use the `  ALTER TABLE  ` statement. A table can have at most one row deletion policy. Adding a row deletion policy to a table with an existing policy fails with an error. See [TTL on generated columns](#ttl_on_generated_columns) to specify more sophisticated row deletion logic.

``` text
ALTER TABLE Albums
ADD ROW DELETION POLICY (OLDER_THAN(timestamp_column, INTERVAL num_days DAY));
```

### PostgreSQL

To create a row deletion policy using PostgreSQL, you can define a `  TTL INTERVAL  ` clause when you create a new table, or add a policy to an existing table.

To add a policy at the time of table creation, do the following:

``` text
CREATE TABLE mytable (
  key bigint NOT NULL,
  timestamp_column_name TIMESTAMPTZ,
  PRIMARY KEY(key)
) TTL INTERVAL interval_specvar> ON timestamp_column_name;
```

Replace the following:

  - `  timestamp_column_name  ` must be a column with data type `  TIMESTAMPTZ  ` . You need to create this column in the `  CREATE TABLE  ` statement. Columns with [commit timestamps](/spanner/docs/commit-timestamp) are valid, as are [generated columns](#ttl_on_generated_columns) . However, you can't specify a generated column that references a commit timestamp column.

  - `  interval_spec  ` is the number of days past the timestamp in the `  timestamp_column_name  ` on which the row is marked for deletion. The value must be a non-negative integer and it must evaluate to a whole number of days. For example, `  '3 days'  ` is allowed, but `  '3 days - 2 minutes'  ` returns an error.

To add a policy to an existing table, use the `  ALTER TABLE  ` statement. A table can have at most one TTL policy. Adding a TTL policy to a table with an existing policy fails with an error. See [TTL on generated columns](#ttl_on_generated_columns) to specify more sophisticated TTL logic.

To add a policy to an existing table, do the following:

``` text
ALTER TABLE albums
ADD COLUMN timestampcolumn TIMESTAMPTZ;

ALTER TABLE albums
ADD TTL INTERVAL '5 days' ON timestampcolumn;
```

## Restrictions

Row deletion policies have the following restrictions.

### TTL on tables referenced by a foreign key

You can't create a row deletion policy:

  - On a table that's referenced by a [foreign key](/spanner/docs/foreign-keys/overview) that doesn't include the `  ON DELETE CASCADE  ` constraint.
  - On the parent of a table that's referenced by a foreign key that doesn't include the ON DELETE CASCADE referential action.

In the following example, you can't add a row deletion policy to the `  Customers  ` table, because it's referenced by a foreign key in the `  Orders  ` table, which doesn't have the `  ON DELETE CASCADE  ` constraint. Deleting customers might violate this foreign key constraint. You also can't add a row deletion policy to the `  Districts  ` table. Deleting a row from `  Districts  ` might cause deletes to cascade in the child `  Customers  ` table, which might violate the foreign key constraint on the `  Orders  ` table.

### GoogleSQL

``` text
CREATE TABLE Districts (
  DistrictID INT64
) PRIMARY KEY (DistrictID);

CREATE TABLE Customers (
  DistrictID INT64,
  CustomerID INT64,
  CreatedAt TIMESTAMP
) PRIMARY KEY (DistrictID, CustomerID),
INTERLEAVE IN PARENT Districts ON DELETE CASCADE;

CREATE TABLE Orders (
  OrderID INT64,
  DistrictID INT64,
  CustomerID INT64,
  CONSTRAINT FK_CustomerOrder FOREIGN KEY (DistrictID, CustomerID) REFERENCES Customers (DistrictID, CustomerID)
) PRIMARY KEY (OrderID);
```

### PostgreSQL

``` text
CREATE TABLE districts (
  districtid   bigint NOT NULL,
  PRIMARY KEY(districtid)
);

CREATE TABLE customers (
  districtid   bigint NOT NULL,
  customerid   bigint NOT NULL,
  createdat  timestamptz,
  PRIMARY KEY(districtid, customerid)
) INTERLEAVE IN PARENT districts ON DELETE CASCADE;

CREATE TABLE orders (
  orderid bigint NOT NULL,
  districtid   bigint,
  customerid bigint,
  PRIMARY KEY(orderid),
  CONSTRAINT fk_customerorder FOREIGN KEY (districtid, customerid) REFERENCES customers (districtid, customerid)
);
```

You can create a row deletion policy on a table that's referenced by a foreign key constraint that uses `  ON DELETE CASCADE  ` . In the following example, you can create a row deletion policy on the `  Customers  ` table which is referenced by the foreign key constraint `  CustomerOrder  ` , defined on the `  Orders  ` table. When TTL deletes rows in `  Customers  ` , the deletion cascades down to matching rows that are in the `  Orders  ` table.

### GoogleSQL

``` text
 CREATE TABLE Districts (
  DistrictID INT64,
  CreatedAt TIMESTAMP
) PRIMARY KEY (DistrictID),
ROW DELETION POLICY (OLDER_THAN(CreatedAt, INTERVAL 1 DAY));

CREATE TABLE Customers (
  DistrictID INT64,
  CustomerID INT64,
  CreatedAt TIMESTAMP
) PRIMARY KEY (DistrictID, CustomerID),
INTERLEAVE IN PARENT Districts ON DELETE CASCADE,
ROW DELETION POLICY (OLDER_THAN(CreatedAt, INTERVAL 1 DAY));

CREATE TABLE Orders (
  OrderID INT64,
  DistrictID INT64,
  CustomerID INT64,
  CONSTRAINT FK_CustomerOrder FOREIGN KEY (DistrictID, CustomerID) REFERENCES Customers (DistrictID, CustomerID) ON DELETE CASCADE
) PRIMARY KEY (OrderID);
```

### PostgreSQL

``` text
CREATE TABLE districts (
  districtid   bigint NOT NULL,
  createdat  timestamptz,
  PRIMARY KEY(districtid)
) TTL INTERVAL '1 day' ON createdat;

CREATE TABLE customers (
  districtid   bigint NOT NULL,
  customerid   bigint NOT NULL,
  createdat  timestamptz,
  PRIMARY KEY(districtid, customerid)
) INTERLEAVE IN PARENT districts ON DELETE CASCADE
TTL INTERVAL '1 day' ON createdat;

CREATE TABLE orders (
  orderid bigint NOT NULL,
  districtid bigint,
  customerid bigint,
  PRIMARY KEY(orderid),
  CONSTRAINT fk_customerorder FOREIGN KEY (districtid, customerid) REFERENCES customers (districtid, customerid) ON DELETE CASCADE
);
```

Similarly, you can create a row deletion policy on a parent of a table that's referenced by a `  ON DELETE CASCADE  ` foreign key constraint.

### TTL on columns with default values

A row deletion policy can use a timestamp column with a default value. A typical default value is `  CURRENT_TIMESTAMP  ` . If no value is explicitly assigned to the column, or if the column is set to its default value by an `  INSERT  ` or `  UPDATE  ` statement, the default value is used in the rule calculation.

In the following example, the default value for the column `  CreatedAt  ` in table `  Customers  ` is the timestamp at which the row is created.

### GoogleSQL

``` text
CREATE TABLE Customers (
  CustomerID INT64,
  CreatedAt TIMESTAMP DEFAULT (CURRENT_TIMESTAMP())
) PRIMARY KEY (CustomerID);
```

For more information, see [DEFAULT (expression)](../reference/standard-sql/data-definition-language.md#spanner-default-clause) .

### PostgreSQL

``` text
CREATE TABLE customers (
  customerid bigint NOT NULL,
  createdat timestamptz DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY(customerid)
  );
```

For more information, see [CREATE TABLE](/spanner/docs/reference/postgresql/data-definition-language#create_table) .

### TTL on generated columns

Row deletion policies can use [generated columns](/spanner/docs/generated-column/how-to) to express more sophisticated rules. For example, you can define a row deletion policy on the `  greatest  ` timestamp ( [GoogleSQL](/spanner/docs/reference/standard-sql/mathematical_functions#greatest) or [PostgreSQL](/spanner/docs/reference/postgresql/functions-and-operators) ) of multiple columns, or map another value to a timestamp.

**Note:** You can't specify a generated column that references a [commit timestamp](/spanner/docs/commit-timestamp) column.

### GoogleSQL

The following table named `  Orders  ` tracks sales orders. The table owner wants to set up a row deletion policy that deletes cancelled orders after 30 days, and non-cancelled orders after 180 days.

Spanner TTL only allows one row deletion policy per table. To express the two criteria in a single column, you can use a generated column with an `  IF  ` statement:

``` text
CREATE TABLE Orders (
  OrderId INT64 NOT NULL,
  OrderStatus STRING(30) NOT NULL,
  LastModifiedDate TIMESTAMP NOT NULL,
  ExpiredDate TIMESTAMP AS (IF(OrderStatus = 'Cancelled',
    TIMESTAMP_ADD(LastModifiedDate, INTERVAL 30 DAY),
    TIMESTAMP_ADD(LastModifiedDate, INTERVAL 180 DAY))) STORED,
) PRIMARY KEY(OrderId),
ROW DELETION POLICY (OLDER_THAN(ExpiredDate, INTERVAL 0 DAY));
```

The statement creates a column named `  ExpiredDate  ` that adds either 30 days or 180 days to the `  LastModifiedDate  ` depending on the order status. Then, it defines the row deletion policy to expire rows on the day stored in the `  ExpiredDate  ` column by specifying `  INTERVAL 0 day  ` .

### PostgreSQL

The following table named `  Orders  ` tracks sales orders. The table owner wants to set up a row deletion policy that deletes rows after 30 days of inactivity.

Spanner TTL only allows one row deletion policy per table. To express the two criteria in a single column, you can create a generated column:

``` text
CREATE TABLE orders (
    orderid bigint NOT NULL,
    orderstatus varchar(30) NOT NULL,
    createdate timestamptz NOT NULL,
    lastmodifieddate timestamptz,
    expireddate timestamptz GENERATED ALWAYS AS (GREATEST(createdate, lastmodifieddate)) STORED,
    PRIMARY KEY(orderid)
) TTL INTERVAL '30 days' ON expireddate;
```

The statement creates a generated column named `  ExpiredDate  ` that evaluates the most recent of the two dates ( `  LastModifiedDate  ` or `  CreateDate  ` ). Then, it defines the row deletion policy to expire rows 30 days after the order was created, or if the order was modified within those 30 days, it'll extend the deletion by another 30 days.

### TTL and interleaved tables

[Interleaved tables](/spanner/docs/schema-and-data-model#parent-child_table_relationships) are a performance optimization that associates related rows in a one-to-many child table with a row in a parent table. To add a row deletion policy on a parent table, all interleaved child tables must specify `  ON DELETE CASCADE  ` , meaning the child rows are deleted atomically with the parent row. This ensures referential integrity such that deletes on the parent table also delete the related child rows in the same transaction. Spanner TTL does not support `  ON DELETE NO ACTION  ` .

### Maximum transaction size

Spanner has a [transaction size limit](/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . Cascading deletes on large parent-child hierarchies with indexed columns could exceed these limits and cause one or more TTL operations to fail. For failed operations, TTL retries with smaller batches, down to a single parent row. However, large child hierarchies for even a single parent row could still exceed the mutation limit.

Failed operations are reported in [TTL metrics](/spanner/docs/ttl/monitoring-and-metrics) .

If a single row and its interleaved children is too large to delete, you can attach a row deletion policy directly on the child tables, in addition to the one on the parent table. The policy on child tables should be configured such that child rows are deleted prior to parent rows.

Consider attaching a row deletion policy to child tables when the following two statements apply:

  - The child table has any global indexes associated with it; and
  - You expect a large number of (\>100) child rows per parent row.

## Delete a row deletion policy

You can drop an existing row deletion policy from a table. This returns an error if there's no existing row deletion policy on the table.

### GoogleSQL

``` text
ALTER TABLE MyTable
DROP ROW DELETION POLICY;
```

### PostgreSQL

``` text
ALTER TABLE mytable
DROP TTL;
```

Deleting a row deletion policy immediately aborts any TTL processes running in the background. Any rows already deleted by the in-progress processes remain deleted.

### Delete a column referenced by a row deletion policy

Spanner doesn't let you delete a column that's referenced by a row deletion policy. You must first [delete the row deletion policy](#delete) before deleting the column.

## View the row deletion policy of a table

You can view the row deletion policies of your Spanner tables.

### GoogleSQL

``` text
SELECT TABLE_NAME, ROW_DELETION_POLICY_EXPRESSION
FROM INFORMATION_SCHEMA.TABLES
WHERE ROW_DELETION_POLICY_EXPRESSION IS NOT NULL;
```

For more information, see [Information schema for GoogleSQL-dialect databases](/spanner/docs/information-schema) .

### PostgreSQL

``` text
SELECT table_name, row_deletion_policy_expression
FROM information_schema.tables
WHERE row_deletion_policy_expression is not null;
```

For more information, see [Information schema for PostgreSQL-dialect databases](/spanner/docs/information-schema-pg) .

## Modify a row deletion policy

You can alter the column or the interval expression of an existing row deletion policy. The following example switches the column from `  CreatedAt  ` to `  ModifiedAt  ` and extends the interval from `  1 DAY  ` to `  7 DAY  ` . This returns an error if there's no existing row deletion policy on the table.

### GoogleSQL

``` text
ALTER TABLE MyTable
REPLACE ROW DELETION POLICY (OLDER_THAN(ModifiedAt, INTERVAL 7 DAY));
```

### PostgreSQL

``` text
ALTER TABLE mytable
ALTER TTL INTERVAL '7 days' ON timestampcolumn;
```
