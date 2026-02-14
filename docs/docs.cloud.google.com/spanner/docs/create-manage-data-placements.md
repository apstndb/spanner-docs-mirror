**Preview — [Geo-partitioning](/spanner/docs/geo-partitioning)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**Note:** This feature is available with the Spanner Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to create and manage data placements in Spanner.

For more information about how geo-partitioning works, see the [Geo-partitioning overview](/spanner/docs/geo-partitioning) .

## Create a data placement

After you [create your Spanner instance partitions](/spanner/docs/create-manage-partitions) and [databases](/spanner/docs/create-manage-databases) , create your placement.

### Console

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance with user-created instance partition(s).

3.  Select the database that you want to partition data.

4.  In the navigation menu, click **Spanner Studio** .

5.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

6.  Enter the `  CREATE PLACEMENT  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#create-placement) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#create-placement) ) DDL statement.
    
    For example, you can run the following to create a placement table `  europeplacement  ` in the instance partition `  europe-partition  ` :
    
    ### GoogleSQL
    
    ``` text
    CREATE PLACEMENT europeplacement OPTIONS (instance_partition="europe-partition");
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE PLACEMENT europeplacement WITH (instance_partition='europe-partition');
    ```
    
    Optional: You can also use the **Object Explorer** pane to view, search, and interact with your Placement objects. For more information, see [Explore your data](/spanner/docs/manage-data-using-console#access-spanner-studio) .

7.  Click **Run** .

### gcloud

To create a placement with the Google Cloud CLI command, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) .

For example, create a placement in the instance partition `  europe-partition  ` :

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="CREATE PLACEMENT europeplacement OPTIONS (instance_partition='europe-partition')"
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="CREATE PLACEMENT europeplacement WITH (instance_partition='europe-partition')"
```

### Set the default leader for a placement

You can set the default leader region of a placement if its location is in a dual-region or multi-region. The new leader region must be one of the two read-write regions within the dual-region or multi-region placement location. For more information, see the [Dual-region available configurations](/spanner/docs/instance-configurations#available-configurations-dual) and [Multi-region available configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) tables.

If you don't set a leader region, your placement uses the default leader region as specified by its location. For a list of the leader region for each dual-region or multi-region location, see the [Dual-region available configurations](/spanner/docs/instance-configurations#available-configurations-dual) and [Multi-region available configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) tables. The default leader region is denoted with an *L* . For example, the default leader region of `  nam8  ` is in Los Angeles( `  us-west2  ` ). The following instructions explain how to set it to Oregon( `  us-west1  ` ).

### Console

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance with user-created instance partition(s).

3.  Select the database that you want to partition data.

4.  In the navigation menu, click **Spanner Studio** .

5.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

6.  Enter the `  CREATE PLACEMENT  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#create-placement) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#create-placement) ) DDL statement.
    
    For example, you can run the following to create a placement table `  nam8placement  ` in the instance partition `  nam8-partition  ` with the default leader location set as `  us-west1  ` :
    
    ### GoogleSQL
    
    ``` text
    CREATE PLACEMENT `nam8placement`
      OPTIONS (instance_partition="nam8-partition", default_leader="us-west1");
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE PLACEMENT nam8placement WITH (instance_partition='nam8-partition', default_leader='us-west1');
    ```
    
    Optional: You can also use the **Object Explorer** pane to view, search, and interact with your Placement objects. For more information, see [Explore your data](/spanner/docs/manage-data-using-console#access-spanner-studio) .

7.  Click **Run** .

### gcloud

To create a placement with the gcloud CLI command, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) .

For example, create a placement table `  nam8placement  ` in the instance partition `  nam8-partition  ` with the default leader location set as `  us-west1  ` :

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="CREATE PLACEMENT nam8placement \
     OPTIONS (instance_partition='nam8-partition', default_leader='us-west1')"
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="CREATE PLACEMENT nam8placement WITH (instance_partition='nam8-partition', default_leader='us-west1')"
```

## Drop a data placement

Before you drop a placement, you must remove all row data from the placement. After you have done so, you can use the Google Cloud console or gcloud CLI to drop the placement.

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Enter the `  DROP PLACEMENT  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#drop-placement) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#drop-placement) ) DDL statement.
    
    For example, you can run the following to drop the placement table `  europeplacement  ` :
    
    ### GoogleSQL
    
    ``` text
    DROP PLACEMENT europeplacement;
    ```
    
    ### PostgreSQL
    
    ``` text
    DROP PLACEMENT europeplacement;
    ```

### gcloud

To drop a placement with the gcloud CLI command, use `  gcloud spanner databases ddl update  ` .

For example, drop placement `  europeplacement  ` :

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="DROP PLACEMENT europeplacement"
```

### Drop placement errors

If the placement is in use, then the `  DROP PLACEMENT  ` operation fails with an error message such as: "Statement failed: Placement PLACEMENT\_NAME cannot be dropped because it is in use by placement table PLACEMENT\_TABLE\_NAME .". If you encounter this error, do the following:

1.  Modify your application to stop inserting or updating rows with the placement you want to drop.

2.  Either:
    
      - Move existing placement rows that use the placement you want to delete to a different placement with a [partitioned DML](/spanner/docs/dml-partitioned) statement like the following:
        
        ``` text
        UPDATE PLACEMENT_TABLE_NAME SET LOCATION = NEW_PLACEMENT_NAME
        WHERE LOCATION = ORIGINAL_PLACEMENT_NAME;
        ```
    
      - Delete the placement rows with a partitioned DML statement like the following:
        
        ``` text
        DELETE FROM PLACEMENT_TABLE_NAME
        WHERE LOCATION = ORIGINAL_PLACEMENT_NAME;
        ```
        
        The previous placement-specific DML statements only work with partitioned DML. They will fail as regular DML statements. For more information, see [Limitations](/spanner/docs/geo-partitioning#limitations) . You can also use the mutation API to move or drop placement rows.

### Cancel a `     DROP PLACEMENT    ` operation

You can cancel a `  DROP PLACEMENT  ` operation anytime before the long-running operation completely deletes the placement from the database schema. For details on how to obtain the long-running operation ID to check the status, or to cancel the operation, see [Manage and observe long-running operations](/spanner/docs/manage-and-observe-long-running-operations) .

## Create a table with a placement key

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Enter the `  CREATE TABLE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#create_table) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#create_table) ) DDL statement.
    
    For example, you can create a `  Singers  ` table that uses a placement key to partition singer data:
    
    ### GoogleSQL
    
    ``` text
    CREATE TABLE Singers (
      SingerId INT64 NOT NULL,
      SingerName STRING(MAX) NOT NULL,
      ...
      Location STRING(MAX) NOT NULL PLACEMENT KEY
    ) PRIMARY KEY (SingerId);
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE TABLE Singers (
      SingerId bigint PRIMARY KEY,
      SingerName varchar(1024),
      ...
      Location varchar(1024) NOT NULL PLACEMENT KEY
    );
    ```

### gcloud

To create a table, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) .

For example, you can create a `  Singers  ` table that uses a placement key to partition singer data:

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="CREATE TABLE Singers ( SingerId INT64 NOT NULL, SingerName STRING(MAX) NOT NULL, Location STRING(MAX) NOT NULL PLACEMENT KEY ) PRIMARY KEY (SingerId);"
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="CREATE TABLE Singers ( SingerId bigint PRIMARY KEY, SingerName varchar(1024), Location varchar(1024) NOT NULL PLACEMENT KEY );"
```

## Edit a table with a placement key

You can't drop a placement key from a table. You also can't add a placement key to a table after it has been created. However, you can use the `  ALTER TABLE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#alter_table) ) DDL statement to change other fields in the table, for example, by adding and dropping non-placement key columns.

## Delete a table with a placement key

Before you delete a table with a placement key, you must first:

1.  Delete all rows in the placement table.
2.  Wait for the [`  version_retention_period  `](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) for the database to pass. For more information, see [Point-in-time recovery](/spanner/docs/pitr) . Then, following these steps:

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Enter the `  DROP TABLE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#drop_table) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#drop-table) ) DDL statement.
    
    For example, drop the `  Singers  ` table:
    
    ``` text
    DROP TABLE Singers;
    ```

### gcloud

To drop a table, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) .

For example, drop the `  Singers  ` table:

``` text
gcloud spanner databases ddl update example-db \
  --instance=test-instance \
  --ddl="DROP TABLE Singers"
```

## Insert a row in a placement table

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Enter the `  INSERT INTO  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/dml-syntax#insert-statement) , [PostgreSQL](/spanner/docs/reference/postgresql/dml-syntax#insert-statement) ) DDL statement.
    
    For example, add a singer, Marc Richards, to the `  Singers  ` table and partition it in `  europeplacement  ` :
    
    ``` text
    INSERT INTO Singers(SingerId, SingerName, Location)
    VALUES (1, 'Marc Richards', 'europeplacement')
    ```

### gcloud

To write data to a table, use [`  gcloud spanner rows insert  `](/sdk/gcloud/reference/spanner/rows/insert) .

For example, add a singer, Marc Richards, to the `  Singers  ` table and partition it in `  europeplacement  ` :

``` text
gcloud spanner rows insert --table=Singers --database=example-db \
  --instance=test-instance --data=SingerId=1,SingerName='Marc Richards',Location='europeplacement'
```

## Update a row in a placement table

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Use [DML](/spanner/docs/dml-tasks) or [mutations](/spanner/docs/modify-mutation-api) to update data in a placement table.
    
    For example, update the name of `  singerid=1  ` in the `  Singers  ` table to `  Catalina Smith  ` :
    
    ``` text
    UPDATE Singers s
    SET s.name='Catalina Smith'
    WHERE s.id=1;
    ```

### gcloud

To update data in a placement table, use [`  gcloud spanner rows update  `](/sdk/gcloud/reference/spanner/rows/update) .

For example, update the name of `  singerid=1  ` in the `  Singers  ` table to `  Catalina Smith  ` :

``` text
gcloud spanner rows update --table=Singers --database=example-db \
  --instance=test-instance --data=SingerId=1,SingerName='Catalina Smith'
```

## Move a row in a placement table

### Console

1.  Create a new instance partition and placement if you haven't already.

2.  In the navigation menu, click **Spanner Studio** .

3.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

4.  Use [DML](/spanner/docs/dml-tasks) or [mutations](/spanner/docs/modify-mutation-api) to move data to the new instance partition.
    
    For example, move `  singerid=1  ` in the `  Singers  ` table to `  asiaplacement  ` :
    
    ``` text
    UPDATE Singers s
    SET s.location='asiaplacement'
    WHERE s.id=1;
    ```

### gcloud

After creating the instance partition and placement where you want to move your data, use [`  gcloud spanner rows update  `](/sdk/gcloud/reference/spanner/rows/update) .

For example, move `  singerid=1  ` in the `  Singers  ` table to `  asiaplacement  ` :

``` text
gcloud spanner rows update --table=Singers --database=example-db \
  --instance=test-instance --data=SingerId=1,Location='asiaplacement'
```

## Delete a row in a placement table

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Use [DML](/spanner/docs/dml-tasks) or [mutations](/spanner/docs/modify-mutation-api) to delete data.
    
    For example, delete `  singerid=1  ` in the `  Singers  ` table:
    
    ``` text
    DELETE FROM Singers s
    WHERE s.id=1;
    ```

### gcloud

To delete data, use [`  gcloud spanner rows delete  `](/sdk/gcloud/reference/spanner/rows/delete) .

For example, delete `  singerid=1  ` in the `  Singers  ` table:

``` text
gcloud spanner rows delete --table=Singers --database=example-db \
  --instance=test-instance --keys=1
```

## Query data in a placement table

### Console

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Run your query.
    
    For example, query the `  Singers  ` table:
    
    ``` text
    SELECT * FROM Singers s WHERE s.SingerId=1;
    ```

### gcloud

To query data, use [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) .

For example, query the `  Singers  ` table:

``` text
gcloud spanner databases execute-sql example-db \
  --instance=test-instance \
  --sql='SELECT * FROM Singers s WHERE s.SingerId=1'
```

## What's next

  - Learn more about [geo-partitioning](/spanner/docs/geo-partitioning) .

  - Learn how to [create and manage instance partitions](/spanner/docs/create-manage-partitions) .
