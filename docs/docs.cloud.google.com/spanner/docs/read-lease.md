This page describes and explains how to use Spanner read leases. Read leases help your databases and [placements](/spanner/docs/geo-partitioning) reduce strong read latency in read-write or read-only regions at the cost of higher write latency.

By default, when Spanner receives a [strong read request](/spanner/docs/reads#read_types) in a non-leader region, the replica serving the read contacts the instance's [leader read-write region](/spanner/docs/region-types#read-write) . This contact confirms that its data is up-to-date before serving the request. This process incurs a network round trip between the region that receives the request and the leader region. Unlike communication within a single region, the geographic distance between regions adds additional latency to the request.

Using Spanner read leases eliminate the need for this round trip. When you set one or more read lease regions for your database or placement, Spanner gives those regions the right to serve consistent reads locally. This allows the non-leader regions to directly serve strong reads without communicating with the leader region. Serving strong reads from a non-leader region that is closer to the client reduces latency across regions. This achieves intra-region latency for strong reads in dual-region or multi-region instances. When read leases are enabled, strong reads within read-only transactions are served locally without contacting the leader. For read-write transactions, reads are still directed to the leader region unless you [use repeatable read isolation with leader aware routing disabled](#limitations) .

Enabling or disabling the read leases feature on a region doesn't require downtime. However, writes experience higher latency when you use the feature, because enabling read lease requires the leader to contact read lease regions when serving writes. As a side effect, writes hold locks for longer, which can impact high-contention write workloads. For more information, see [When to use read leases](#when-to-use) . Read lease is most suitable for applications that are willing to trade off increased write latency for faster strong reads. For example, an access control system where the workload has frequent reads but rare writes.

To learn how to enable read leases, see [Use read leases](#use-read-leases) .

## When to use read leases

Enable read leases if your application and workload meet the following criteria:

  - Low latency for strong reads is more important than low latency for writes.
  - Your workload can tolerate longer write [lock](/spanner/docs/introspection/lock-statistics) durations, or has low write contention.

When there are concurrent writes, the choice between using the [query APIs](/spanner/docs/samples/spanner-query-data) or the [read APIs](/spanner/docs/samples/spanner-read-data) affects the performance of a database that uses read lease regions.

To learn more about monitoring latency, see [Monitor](#monitor) .

## Example use case

Consider a globally deployed application that performs writes in the US and has clients in the US, Europe, and Asia. You can configure a multi-region Spanner instance, such as `  nam-eur-asia1  ` , with a leader region in `  us-central1  ` and read-only replicas in `  europe-west1  ` and `  asia-east1  ` .

When you enable read lease in the `  europe-west1  ` and `  asia-east1  ` read-only regions, Spanner serves strong reads from Europe and Asia from those local replicas, reducing latency. The trade-off is an increase in write latency for all writes. The increased latency is equivalent to the round-trip time between the leader `  us-central1  ` region and the farthest read lease regions.

## Limitations

Spanner read leases have the following limitations:

  - Read leases don't reduce latency for reads that are part of a read-write transaction. If you want to achieve intra-region latency for reads inside read-write transactions, then you must use the [repeatable read isolation level](/spanner/docs/isolation-levels#repeatable-read) and disable [leader aware routing](/spanner/docs/leader-aware-routing#disable) to use read leases. Even when using repeatable read isolation, reads in read-write transactions might still be directed to the leader region, particularly after a write has occurred within the transaction, to ensure read-your-writes consistency.
  - If you [move your instance](/spanner/docs/move-instance) to a different instance configuration, the read lease settings aren't preserved. You must re-enable read lease on the database after the move completes.
  - You can't disable or change the read lease region on an existing placement. This limitation is subject to change or removal upon the [geo-partitioning](/spanner/docs/geo-partitioning) GA release or after. If you want to disable read lease in an existing placement, do the following:
      - [Create a new placement](/spanner/docs/create-manage-data-placements#create-placement) without specifying the read lease option or with a different read lease region that you want to use.
      - Use [partitioned DML](/spanner/docs/dml-partitioned) to update the placement of your data to the new placement. This update initiates a background process to [move](/spanner/docs/create-manage-data-placements#move-row) the data. Spanner can move approximately 10 placement rows per second for every node in your destination instance partition. Your CPU usage might be impacted during this period due to the move.
      - [Drop the original placement](/spanner/docs/create-manage-data-placements#drop-placement) after the data move is complete.

## Use read leases

You must [enable read leases](#enable-read-leases) before you can use it.

### Access control with IAM

To set read lease regions, a user needs the `  spanner.databases.create  ` or `  spanner.databases.updateDdl  ` IAM permission. The predefined [Database Admin role ( `  roles/spanner.databaseAdmin  ` )](/spanner/docs/iam#roles) includes these permissions. For more information, see [IAM overview for Spanner](/spanner/docs/iam) .

For information on how to grant permissions, see [Apply IAM permissions](/spanner/docs/grant-permissions) .

### Before you begin for PostgreSQL database users

If you want to use read lease in a PostgreSQL database, make one of the following configuration changes to your database. Otherwise, your reads are still served by the leader region even if you have set read lease regions.

  - If you only use read-only transactions, configure your PostgreSQL connection so that the default status of each new transaction in the database is set to read-only. To do so, set the [`  default_transaction_read_only  `](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-DEFAULT-TRANSACTION-READ-ONLY) option to `  true  ` .
    
    ``` text
    postgres://USER_ID:PASSWORD@localhost:5432/DATABASE_ID?sslmode=disable&options=-c \
      default_transaction_read_only=true
    host=/tmp port=5432 database=DATABASE_ID \
      options='-c default_transaction_read_only=true'
    ```
    
    Replace the following:
    
      - USER\_ID with the unique identifier of your user.
    
      - PASSWORD with your password.
    
      - DATABASE\_ID with the unique identifier of your database.

  - If you want to achieve intra-region latency read within read-write transactions or you can't always switch your connection option, use the [repeatable read isolation level](/spanner/docs/isolation-levels#repeatable-read) and disable [leader aware routing](/spanner/docs/leader-aware-routing#disable) . These settings are required even if your read-write transaction only has reads. Otherwise the reads inside read-write transactions are always directed to the leader region. In such a transaction, read lease is disabled after the first write DML statement appears. This is because writes are always directed at the leader region. Therefore, in order to read your write, subsequent reads need to go to the leader region, too.
    
    ``` text
    postgres://USER_ID:PASSWORD@localhost:5432/DATABASE_ID?sslmode=disable&options=-c \
      default_isolation_level=REPEATABLE_READ -c routeToLeader=false
    host=/tmp port=5432 database=DATABASE_ID \
      options='-c default_isolation_level=REPEATABLE_READ -c routeToLeader=false'
    ```

### Enable read leases

To enable read leases when you create a new database, set the `  read_lease_regions  ` option in the `  ALTER DATABASE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#alter_database) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#alter-database) ) DDL statement:

### Console

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance in which you want to enable read lease.

3.  In the **Instance overview** page that opens, click **Create database** .

4.  For the database name, enter a name.

5.  Select a database dialect.

6.  Click **Create** .
    
    The Google Cloud console displays the **Overview** page for the database you created.

7.  In the navigation menu, click **Spanner Studio** .

8.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

9.  Enter the following `  ALTER DATABASE  ` DDL statement.
    
    ### GoogleSQL
    
    ``` text
    ALTER DATABASE DATABASE_ID
    SET OPTIONS (read_lease_regions = 'READ_LEASE_REGION');
    ```
    
    Replace the following:
    
      - DATABASE\_ID with the unique identifier of your database.
    
      - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .
    
    ### PostgreSQL
    
    ``` text
    ALTER DATABASE DATABASE_ID
    SET "spanner.read_lease_regions" = 'READ_LEASE_REGION';
    ```
    
    Replace the following:
    
      - DATABASE\_ID with the unique identifier of your database.
    
      - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

10. Click **Run** .

### gcloud

To set the `  read_lease_regions  ` database option when creating your database, use [`  gcloud spanner databases create  `](/sdk/gcloud/reference/spanner/databases/create) .

### GoogleSQL

``` text
gcloud spanner databases create DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="ALTER DATABASE DATABASE_ID SET OPTIONS (read_lease_regions = 'READ_LEASE_REGION');"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.
  - `  READ_LEASE_REGION  ` : the region where you want to enable read lease. For example, `  europe-west1  ` . You can enable read lease for multiple regions. Separate each region with a comma.

### PostgreSQL

``` text
gcloud spanner databases create DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="ALTER DATABASE DATABASE_ID \
    SET "spanner.read_lease_regions" = 'READ_LEASE_REGION';"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.
  - `  READ_LEASE_REGION  ` : the region where you want to enable read lease. For example, `  europe-west1  ` . You can enable read lease for multiple regions. Separate each region with a comma.

To enable read lease when you update an existing database, set the `  read_lease_regions  ` option in the `  ALTER DATABASE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#alter_database) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#alter-database) ) DDL statement:

### Console

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance in which you want to enable read lease.

3.  Select the database in which you want to enable read lease.

4.  In the navigation menu, click **Spanner Studio** .

5.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

6.  Enter the following `  ALTER DATABASE  ` DDL statement.
    
    ### GoogleSQL
    
    ``` text
    ALTER DATABASE DATABASE_ID \
    SET OPTIONS (read_lease_regions = 'READ_LEASE_REGION');
    ```
    
    Replace the following:
    
      - DATABASE\_ID with the unique identifier of your database.
    
      - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .
    
    ### PostgreSQL
    
    ``` text
    ALTER DATABASE DATABASE_ID \
    SET "spanner.read_lease_regions" = 'READ_LEASE_REGION';
    ```
    
    Replace the following:
    
      - DATABASE\_ID with the unique identifier of your database.
    
      - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

7.  Click **Run** .

### gcloud

To set the `  read_lease_regions  ` database option, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) .

### GoogleSQL

``` text
gcloud spanner databases ddl update DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="ALTER DATABASE DATABASE_ID \
    SET OPTIONS (read_lease_regions = 'READ_LEASE_REGION');"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.
  - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

### PostgreSQL

``` text
gcloud spanner databases ddl update DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="ALTER DATABASE DATABASE_ID \
    SET "spanner.read_lease_regions" = 'READ_LEASE_REGION';"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.
  - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

To enable read lease when you create a new placement, use the `  read_lease_regions  ` option in the `  CREATE PLACEMENT  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#create-placement) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#create-placement) ) DDL statement to set one or more regions where you want to use read lease:

**Preview — [Geo-partitioning](/spanner/docs/geo-partitioning)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

### Console

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance in which you want to enable read lease.

3.  Select the database in which you want to enable read lease for a placement.

4.  In the navigation menu, click **Spanner Studio** .

5.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

6.  Enter the following `  CREATE PLACEMENT  ` DDL statement.
    
    ### GoogleSQL
    
    ``` text
    CREATE PLACEMENT PLACEMENT_NAME
    OPTIONS (instance_partition="PARTITION_ID",
    read_lease_regions = 'READ_LEASE_REGION');
    ```
    
    Replace the following:
    
      - PLACEMENT\_NAME with the name of your placement.
      - PARTITION\_ID with the unique identifier of the partition to associate with the placement.
      - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .
    
    ### PostgreSQL
    
    ``` text
    CREATE PLACEMENT PLACEMENT_NAME
    WITH (instance_partition='PARTITION_ID',
          read_lease_regions = 'READ_LEASE_REGION';
    ```
    
    Replace the following:
    
      - PLACEMENT\_NAME with the name of your placement.
      - PARTITION\_ID with the unique identifier of the partition to associate with the placement.
      - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

7.  Click **Run** .

### gcloud

To set the `  read_lease_regions  ` database option for a placement, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) with a `  CREATE PLACEMENT  ` statement.

### GoogleSQL

``` text
gcloud spanner databases ddl update DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="CREATE PLACEMENT PLACEMENT_NAME OPTIONS (instance_partition=\"PARTITION_ID\", read_lease_regions = 'READ_LEASE_REGION');"`
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.
  - `  PLACEMENT_NAME  ` : the name of your placement.
  - `  PARTITION_ID  ` : the unique identifier of the partition to associate with the placement.
  - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

### PostgreSQL

``` text
gcloud spanner databases ddl update DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="CREATE PLACEMENT PLACEMENT_NAME WITH (instance_partition='PARTITION_ID', read_lease_regions = 'READ_LEASE_REGION';"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.
  - `  PLACEMENT_NAME  ` : the name of your placement.
  - `  PARTITION_ID  ` : the unique identifier of the partition to associate with the placement.
  - READ\_LEASE\_REGION with one or more regions where you want to enable read lease. For example, `  europe-west1, europe-west4  ` .

### Disable read leases

**Note:** You can't disable read lease in a placement.

Read lease is disabled by default.

To update and disable the feature on an existing database, set the `  read_lease_regions  ` option in the `  ALTER DATABASE  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#alter_database) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#alter-database) ) DDL statement to `  NULL  ` :

### Console

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance in which you want to disable read lease.

3.  Select the database in which you want to disable read lease.

4.  In the navigation menu, click **Spanner Studio** .

5.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

6.  Enter the following `  ALTER DATABASE  ` DDL statement.
    
    ### GoogleSQL
    
    ``` text
    ALTER DATABASE DATABASE_ID SET OPTIONS (read_lease_regions = NULL);
    ```
    
    Replace DATABASE\_ID with the unique identifier of your database.
    
    ### PostgreSQL
    
    ``` text
    ALTER DATABASE DATABASE_ID SET "spanner.read_lease_regions" = NULL;
    ```
    
    Replace DATABASE\_ID with the unique identifier of your database.

7.  Click **Run** .

### gcloud

To set the `  read_lease_regions  ` database option, use [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) .

### GoogleSQL

``` text
gcloud spanner databases ddl update DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="ALTER DATABASE DATABASE_ID SET OPTIONS (read_lease_regions = NULL);"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.

### PostgreSQL

``` text
gcloud spanner databases ddl update DATABASE_ID \
  --instance=INSTANCE_ID \
  --ddl="ALTER DATABASE DATABASE_ID SET "spanner.read_lease_regions" = NULL;"
```

Replace the following:

  - `  DATABASE_ID  ` : the identifier for your Spanner database.
  - `  INSTANCE_ID  ` : the identifier for your Spanner instance.

## Best practices

To maximize the benefits of using this feature, use [multiplexed sessions](/spanner/docs/sessions#multiplexed_sessions) , which let you create a large number of concurrent requests on a single session.

## Monitor

After enabling read lease, it's important to monitor latency to confirm that the feature achieves the intended effect. To do so, identify the leader region and the regions with read lease enabled by querying the `  data_options  ` information schema table ( [GoogleSQL](/spanner/docs/information-schema#database_options) , [PostgreSQL](/spanner/docs/information-schema-pg#database_options) ) or your database. Regions that have read lease enabled expect strong reads to have intra-region latency. Simultaneously, write latency increases with one round-trip time between the leader region and the farthest region with read lease enabled.

You can also use the following Spanner latency metric to help you monitor read request latencies in your instances:

  - `  spanner.googleapis.com/api/read_request_latencies_by_serving_location  `

You can filter this metric using the `  /serving_location  ` field. The `  /serving location  ` field indicates the location of the Spanner server where the request is served from.

For a full list of available metrics, see [metrics list for Spanner](/monitoring/api/metrics_gcp#gcp-spanner) .

## Cost considerations

Strong reads served from regions with the read lease feature enabled use slightly fewer compute resources. On the other hand, writes for databases with the read lease feature enabled use slightly more compute resources. For more information, see [Spanner compute capacity pricing](https://cloud.google.com/spanner/pricing#compute-capacity) .

The feature doesn't impact other pricing components such as storage and network.

## What's next

  - Learn more about [Spanner replication](/spanner/docs/replication) .
  - Learn more about [Reads outside of transactions](/spanner/docs/reads) .
