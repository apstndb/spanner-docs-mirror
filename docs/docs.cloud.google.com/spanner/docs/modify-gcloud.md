This page describes how to insert, update, and delete data using the [`  gcloud  `](/sdk) command-line tool.

## Modifying data using DML

To execute Data Manipulation Language (DML) statements, use the `  gcloud spanner databases execute-sql  ` command . The following example adds a new row to the `  Singers  ` table.

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql="INSERT Singers (SingerId, FirstName, LastName) VALUES (1, 'Marc', 'Richards')"
```

To execute a Partitioned DML statement, use the `  gcloud spanner databases execute-sql  ` command with the `  --enable-partitioned-dml  ` option. The following example updates rows in the `  Albums  ` table.

``` text
gcloud spanner databases execute-sql example-db \
    --instance=test-instance --enable-partitioned-dml \
    --sql='UPDATE Albums SET MarketingBudget = 0 WHERE MarketingBudget IS NULL'
```

**Note:** Spanner does not support `  --query-mode=PLAN  ` and `  --query-mode=PROFILE  ` for partitioned DML.

For the Spanner DML reference, see [Data Manipulation Language syntax](/spanner/docs/reference/standard-sql/dml-syntax) .

## Modifying rows using the rows command group

Use the `  gcloud spanner rows  ` command group to modify data in a database:

  - Insert new rows into a table.
  - Update columns in existing rows in a table.
  - Delete rows from a table.

The `  rows  ` command group recognizes literals for all [valid column types](/spanner/docs/reference/standard-sql/data-types#allowable_types) .

### Insert a row in a table

To insert a new row in a table, you must include values for the key columns and any other required columns:

``` text
gcloud spanner rows insert --instance=INSTANCE_ID --database=DATABASE_ID \
    --table=TABLE_NAME \
    --data=COL_NAME_1=COL_VALUE_1,COL_NAME_2=COL_VALUE_2,COL_NAME_3=COL_VALUE_3,...,COL_NAME_N=COL_VALUE_N
```

The following example inserts a new row in the `  Singers  ` table:

``` text
gcloud spanner rows insert --instance=test-instance --database=example-db \
    --table=Singers \
    --data=SingerId=1,FirstName='Marc',LastName='Richards'
```

### Update a row in a table

To update a row in a table, you must include values for the key columns and the columns you want to update:

``` text
gcloud spanner rows update --instance=INSTANCE_ID --database=DATABASE_ID \
    --table=TABLE_NAME \
    --data=COL_NAME_1=COL_VALUE_1,COL_NAME_2=COL_VALUE_2,COL_NAME_3=COL_VALUE_3,...,COL_NAME_N=COL_VALUE_N
```

The following example updates a row in the `  Singers  ` table:

``` text
gcloud spanner rows update --instance=test-instance --database=example-db \
    --table=Singers \
    --data=SingerId=1,FirstName='Marc',LastName='Richards'
```

You cannot change the key values using the `  update  ` command. To update a key value, you must create a new row and delete the existing row.

### Delete a row from a table

To delete a row, you must specify the values for the primary key columns:

``` text
gcloud spanner rows delete --instance=INSTANCE_ID --database=DATABASE_ID  \
    --table=TABLE_NAME \
    --keys=KEY_VALUE_1,KEY_VALUE_2,KEY_VALUE_3
```

The following example deletes a row from the \`Singers\` table:

``` text
gcloud spanner rows delete --instance=test-instance --database=example-db \
    --table=Singers \
    --keys=1
```

### Specify ARRAY values

To insert or update values in an [`  ARRAY  ` column](/spanner/docs/reference/standard-sql/arrays) , put the data in a YAML file and use the [`  --flags-file  `](/sdk/gcloud/reference/topic/flags-file) option.

For example, this YAML file specifies the array `  [1,2,3]  ` for the `  Numbers  ` column:

``` text
# stats.yaml
--data:
    Id: 1
    Locked: True
    Numbers:
        -   1
        -   2
        -   3
```

To insert a row with the YAML data, use the `  --flags-file  ` option:

``` text
gcloud spanner rows insert --instance=test-instance --database=example-db \
     --table=Stats \
     --flags-file stats.yaml
```

For a `  NULL  ` array, don't include a value for `  Numbers  ` in the file:

``` text
# stats.yaml
--data:
    Id: 1
    Locked: True
```

For an empty array, define the array as `  []  ` :

``` text
# stats.yaml
--data:
    Id: 1
    Locked: True
    Numbers: []
```

### Specify commit timestamps

To insert or update a value automatically in a [commit timestamp](/spanner/docs/commit-timestamp) column, pass `  spanner.commit_timestamp()  ` as the value of the column. The following example writes the commit timestamp in the `  LastUpdated  ` column when the row is inserted.

``` text
gcloud spanner rows insert --instance=test-instance --database=example-db \
    --table=Singers \
    --data=SingerId=1,LastUpdated='spanner.commit_timestamp()'
```

The following example writes a specific timestamp value in the `  LastUpdated  ` column:

``` text
gcloud spanner rows update --instance=test-instance --database=example-db \
    --table=Singers \
    --data=SingerId=1,LastUpdated=2017-01-02T12:34:00.45Z
```
