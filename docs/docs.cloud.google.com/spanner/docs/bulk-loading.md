This page provides guidelines for efficiently bulk loading large amounts of data into Spanner.

You have several options for bulk loading data into Spanner:

  - [Insert rows using Data Manipulation Language (DML)](/spanner/docs/dml-tasks) .
  - [Insert rows using mutations](/spanner/docs/modify-mutation-api) .
  - [Import data using the Dataflow connector](/spanner/docs/dataflow-connector) .
  - [Import a database using Avro files](/spanner/docs/import) .
  - [Import data in CSV format](/spanner/docs/import-export-csv) .

While you can also [insert rows using the Google Cloud CLI](/spanner/docs/modify-gcloud#modify-data) , we don't recommend that you use the gcloud CLI for bulk loading.

## Performance guidelines for bulk loading

To achieve optimal bulk loading performance, maximize your use of partitioning to distribute writing the data across worker tasks.

Spanner uses [load-based splitting](/spanner/docs/schema-and-data-model#load-based_splitting) to evenly distribute your data load across the instance's compute resources. After a few minutes of high load, Spanner introduces split boundaries between rows. In general, if your data load is well-distributed and you follow best practices for schema design and bulk loading, your write throughput should double every few minutes until you saturate the available CPU resources in your instance.

## Partition your data by primary key

Spanner automatically partitions tables into smaller ranges. The primary key for a row determines where it is partitioned.

To get optimal write throughput for bulk loads, partition your data by primary key with this pattern:

  - Each partition contains a range of consecutive rows, as determined by the key columns.
  - Each commit contains data for only a single partition.

We recommend that the number of partitions be 10 times the number of nodes in your Spanner instance. To assign rows to partitions:

  - Sort your data by primary key.
  - Divide the data into 10 \* (number of nodes) separate, equally sized partitions.
  - Create and assign a separate worker task to each partition. Creating the worker tasks happens in your application. It is not a Spanner feature.

Following this pattern, you should see a maximum overall bulk write throughput of 10-20 MB per second per node for large loads.

As you load data, Spanner creates and updates splits to balance the load on the nodes in your instance. During this process, you may experience temporary drops in throughput.

### Example

You have a regional configuration with 3 nodes. You have 90,000 rows in a non-interleaved table. The primary keys in the table range from 1 to 90000.

  - Rows: 90,000 rows
  - Nodes: 3
  - Partitions: 10 \* 3 = 30
  - Rows per partition: 90000 / 30 = 3000.

The first partition includes the key range 1 to 3000. The second partition includes the key range 3001 to 6000. The 30th partition includes the key range 87001 to 90000. (You shouldn't use sequential keys in a large table. This example is only for demonstration.)

Each worker task sends the writes for a single partition. Within each partition, you should write the rows sequentially by primary key. Writing rows randomly, with respect to the primary key, should also provide reasonably high throughput. Measuring test runs will give you insight into which approach provides the best performance for your dataset.

### Bulk load without partitioning

Writing a contiguous set of rows in a commit can be faster than writing random rows. Random rows also likely include data from different partitions.

When more partitions are written into a commit, more coordination across servers is required, raising the commit latency and overhead.

Multiple partitions are likely involved because each random row could belong to a different partition. In the worst case scenario, each write involves every partition in your Spanner instance. As mentioned previously, write throughput is lowered when more partitions are involved.

## Avoid overload

It's possible to send more write requests than Spanner can handle. Spanner handles the overload by aborting transactions, which is called pushback. For write-only transactions, Spanner automatically retries the transaction. In those cases, the pushback shows up as high latency. During heavy loads, pushback can last for up to a minute. During severely heavy loads, pushback can last for several minutes. To avoid pushback, you should throttle write requests to keep CPU utilization within [reasonable limits](/spanner/docs/cpu-utilization#recommended-max) . Alternatively, users can increase the number of nodes so that CPU utilization stays within the limits.

## Commit between 1 MB to 5 MB of mutations at a time

Each write to Spanner contains some overhead, whether the write is big or small. To maximize throughput, maximize the amount of data stored per write. Larger writes lower the ratio of overhead per write. A good technique is for each commit to mutate hundreds of rows. When writing relatively large rows, a commit size of 1 MB to 5 MB usually provides the best performance. When writing small values, or values that are indexed, it is generally best to write at most a few hundred rows in a single commit. Independently from the commit size and number of rows, be aware that there is a limitation of 80,000 [mutations per commit](/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . To determine optimal performance, you should [test and measure](#test-measure) the throughput.

Commits larger than 5 MB or more than a few hundred rows don't provide extra benefit, and they risk exceeding the Spanner [limits](/spanner/docs/limits) on commit size and mutations per commit.

## Guidelines for secondary indexes

If your database has [secondary indexes](/spanner/docs/secondary-indexes) , you must choose between adding the indexes to the database schema before or after loading the table data.

  - Adding the index before data is loaded allows the schema change to complete immediately. However, each write affecting the index takes longer since it also needs to update the index. When the data load is complete, the database is immediately usable with all indexes in place. To create a table and its indexes at the same time, send the DDL statements for the new table and the new indexes in a single request to Spanner.

  - Adding the index after loading the data means that each write is efficient. However, the schema change for each index backfill can take a long time. The database is not fully usable, and queries can't use the indexes until all of the schema changes are complete. The database can still serve writes and queries but at a slower rate.

We recommend adding indexes that are critical to your business application before you load the data. For all non-critical indexes, add them after the data is migrated.

## Use `     INTERLEAVE IN    ` during bulk load

For schemas with many parent child references across tables, you must always load the parent before the children to ensure referential integrity. This orchestration can be tricky, especially with multiple levels of hierarchy. This complexity also makes batching and parallelizing much more difficult and can greatly impact the overall bulk load time. In Spanner, these relationships are enforced using `  INTERLEAVE IN PARENT  ` , or foreign keys. See the [`  CREATE TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#create_table) documentation for more details.

Adding a foreign key after bulk load creates an index under the covers, so follow the guidelines in [secondary-indexes](#secondary-indexes) .

However, for `  INTERLEAVE IN PARENT  ` tables it is recommended that you create all tables using `  INTERLEAVE IN  ` semantics during bulk load, which physically interleaves rows, but does not enforce referential integrity. This gives you the performance benefits of locality, but does not require the up-front ordering. Now that child rows can be inserted before the corresponding parent, this lets Spanner write to all tables in parallel.

Once all tables are loaded, you can then migrate the interleaved tables to start enforcing the parent-child relationship using the `  ALTER TABLE t1 SET INTERLEAVE IN PARENT t2  ` statement. This validates referential integrity, failing if there are any orphaned child rows. If validation fails, identify missing parent rows using the following query.

``` text
      SELECT pk1, pk2 FROM child
      EXCEPT DISTINCT
      SELECT pk1, pk2 FROM parent;
```

## Test and measure the throughput

Predicting throughput can be difficult. We recommend that you test your bulk loading strategy before running the final load. For a detailed example using partitioning and monitoring performance, see [Maximizing data load throughput](https://medium.com/google-cloud/cloud-spanner-maximizing-data-load-throughput-23a0fc064b6d) .

## Best practices for periodic bulk loading to an existing database

If you are updating an existing database that contains data but does not have any [secondary indexes](/spanner/docs/secondary-indexes) , then the recommendations in this document still apply.

If you do have secondary indexes, the instructions might yield reasonable performance. Performance depends on how many [splits](/spanner/docs/schema-and-data-model#database-splits) , on average, are involved in your transactions. If throughput drops too low, you can try the following:

  - Include a smaller number of mutations in each commit, which might increase throughput.
  - If your upload is larger than the total current size of the table being updated, delete your secondary indexes and then add them again after you upload the data. This step is usually not necessary, but it might improve the throughput.
