This document contains information to help you pre-split your database.

Spanner manages [database splits](/spanner/docs/schema-and-data-model#database-splits) and scales in response to load and size changes. Splitting and merging are dynamic, based on traffic. As a split receives more traffic, Spanner subdivides it into smaller ranges and redistributes the resulting splits over other available resources in the instance. Spanner merges the split when the split consistently receives less traffic.

Splitting is not instantaneous. If the splitting and rebalancing can't keep up with the traffic, a split can potentially use up its available compute and memory resources. When this happens, Spanner's work scheduler queues up further requests, increasing latency and potentially leading to timeouts and aborted transactions.

By pre-splitting the database, Spanner can be ready for a predictable increased traffic. You can pre-split your database by creating split points.

Consider pre-splitting your database in the following scenarios:

  - You're loading a large dataset into new tables and indexes in a Spanner database for the first time, such as a one-time bulk load.
  - You're expecting a traffic load increase in an existing Spanner database in the near future. For example, you might need to support a large traffic event like a product launch or a sales campaign.

## Determine split count

We recommend creating 10 split points per node. Because Spanner can split and adjust to the traffic quickly for smaller instances, you don't need to pre-split smaller instances.

## Determine split points

Consider the following when determining split points for your database:

  - If your traffic is evenly distributed across the key range, such as when using UUIDs or bit-reversed sequence keys, choose split points that divide the post traffic key space evenly.

  - If your traffic is concentrated on a set of known key ranges, split and isolate those key ranges.

  - If you're expecting traffic on your indexes, use split points on the corresponding index.

  - Interleaved tables are split if split points are added to the parent table. If you're expecting a higher traffic on the interleaved table, be sure to use split points in the corresponding interleaved table.

  - You can allocate the split points to schema objects proportionally to their increased traffic.

### Sample workflow for determining split points

Assume your database has the table structures defined by the following DDL:

``` text
CREATE TABLE UserInfo (
 UserId INT64 NOT NULL,
 Info BYTES(MAX),
) PRIMARY KEY (UserId);


CREATE TABLE UserLocationInfo (
 UserId INT64 NOT NULL,
 LocationId STRING(MAX) NOT NULL,
 ActivityData BYTES(MAX),
) PRIMARY KEY (UserId, LocationId), INTERLEAVE IN PARENT UserInfo ON DELETE CASCADE;


CREATE INDEX UsersByLocation ON UserLocationInfo(LocationId);
```

`  UserId  ` is a randomly generated hash in the `  INT64  ` space, and you need to add a 100 split points to evenly distribute the anticipated increase in traffic on the `  UserInfo  ` table and its interleaved tables. Because the split points are evenly distributed, you need to find the number of rows, or `  offset  ` between each split point:

`  offset  ` = the maximum value of the `  UserId  ` range / 99

Then, the split points for the table `  UserInfo  ` are determined from the first row of `  UserId  ` , or `  UserId_first  ` . To determine the Nth split point, use the following calculation:

Split point N: `  UserId_first  ` + ( `  offset  ` \* (N-1))

For example, the first split point is `  UserId_first  ` + ( `  offset  ` \* 0) and the third split point is `  UserId_first  ` + ( `  offset  ` \* 2).

Because the `  UserLocationInfo  ` table is an interleaved table of the `  UserInfo  ` table, it's also split at the `  UserId  ` boundaries. You might also want to create split points in the `  UserLocationInfo  ` table on the `  LocationId  ` column.

Consider that the `  LocationId  ` follows the `  $COUNTRY_$STATE_$CITY_$BLOCK_$NUMBER  ` format, for example, `  US_CA_SVL_MTL_1100_7  ` .

For a `  UserId  ` , based on the prefix of the `  LocationId  ` string you can determine splits to put the `  UserLocationInfo  ` table for the `  UserId  ` at 3 different countries in 3 different splits:

  - Split point 1: (1000, "CN")
  - Split point 2: (1000, "FR")
  - Split point 3: (1000, "US")

You can add new split points using just a prefix and don't need to match the specified format for a column or index. In this example, the split points don't match the specified format for `  LocationId  ` , and only use the `  $COUNTRY  ` as the prefix.

If you want to split the `  UsersByLocation  ` index, you can evenly spread the split points on the `  LocationId  ` column, or isolate a few `  LocationId  ` column values that are expected to receive increased traffic:

  - Split point 1: "CN"
  - Split point 2: "US"
  - Split point 3: "US\_NYC"

You can further split the index by using the indexed table key parts for locations that receive even more increased traffic. For example, if you're expecting the `  CN  ` location to receive increased traffic, you could introduce the following split points:

  - Split point 1: "CN" and TableKey: (1000, "CN")
  - Split point 2: "CN" and TableKey: (2000, "CN")
  - Split point 3: "CN" and TableKey: (3000, "CN")

## Split point expiration

You can set an expiration time for each split point. Depending on your use case, set your split points to expire after your anticipated increased traffic subsides.

The default expiration time is 10 days from when the split is created or updated. The maximum allowed expiration time is 30 days from when you create or update the split.

After the split expires, Spanner takes over managing the split and you can no longer view the split. Spanner might merge the split depending on the traffic.

You can also update the expiration time for a split point before it expires. For example, if your increased traffic hasn't subsided, you can increase the split expiration time. If you no longer need a split point, you can set it to expire immediately. To learn how to set the expiration time of split points, see [How to expire a split point](/spanner/docs/create-manage-split-points#expire-splits) .

## Outcomes of pre-splitting your database

The following outcomes are likely after adding split points:

  - **Latency changes** : Adding split points is a way of simulating increases in traffic on the database. When a database has more splits, there can be permanent increases in read and write latency due to more transaction participants and query splits. You can also expect an increase in compute and query usage per read or write request.

  - **Split point efficacy** : To determine whether the added split points are beneficial, monitor the [latency profile](/spanner/docs/monitoring-console#available_scorecards) for minimal changes, and [key visualiser](/spanner/docs/key-visualizer) for hotspots. If you notice hotspots, you can expire the split points immediately and create new ones. For more information about expiring split points, see [How to expire a split point](/spanner/docs/create-manage-split-points#expire-splits) . Consider introducing a smaller number of splits in the next iteration of adding splits and observe the latency profile.

  - **Split point behavior after the traffic increase** : The added split points should be removed after the traffic increase stabilizes. The split distribution might not converge to where it was before the load increase. The database might settle on a different latency profile due to the traffic change and the splitting that is required to support the traffic.

## Example use case

Consider that you are a database administrator at a gaming company and are anticipating an increased traffic for a new game launch. You're expecting the traffic on new tables that are empty.

You need to ensure there's no service disruption when the traffic comes through so that there's no observable impact on the latency or error rates.

Consider the following high-level pre-splitting strategy for this use case:

1.  Identify the number of nodes that the instance needs to support the increased traffic. To learn how to identify node count, see [Performance overview](/spanner/docs/performance) . If you're using autoscaler, set the [maximum limit parameter](/spanner/docs/managed-autoscaler#determine-maximum) to the node count you identified. Also, set the [minimum limit parameter](/spanner/docs/managed-autoscaler#determine-minimum) to (node count you identified / 5).

2.  Identify the tables and indexes that have the most traffic and can benefit the most from using split points. Analyze the current data and choose between using custom split points or evenly distributed split points.

3.  [Create the split points](/spanner/docs/create-manage-split-points#create-split-points) no earlier than seven days and no later than 12 hours before the anticipated traffic increase.

4.  Verify that the splits are created. To view created split points on an instance, see [View split points](/spanner/docs/create-manage-split-points#view-split-points) .

## Caveats

Consider the following caveats when creating split points:

  - **Table, index, and database deletion** : Before deleting a table, index, or database, you need to ensure all the corresponding added split points are expired. You can do this by setting the split expiration date to the current time. This is necessary for the instance level quota to be reclaimed. For more information about expiring split points, see [How to expire a split point](/spanner/docs/create-manage-split-points#expire-splits) .

  - **Backing up and restoring databases** : Added splits aren't backed up. You need to create splits on a restored database.

  - **Asymmetric auto scaling** : If you're using [asymmetric auto scaling](/spanner/docs/autoscaling-overview) , the node count used to determine the split point count is the minimum node count across all the regions.

  - **Temporary increase in storage usage metrics** : Adding split points temporarily increases the [total database storage](/spanner/docs/storage-utilization#metrics) metric until Spanner completes compaction. For more information, see [Storage utilization](/spanner/docs/storage-utilization) . This only happens when existing key ranges are split further, and not when new key ranges are split.

  - You should create split points no earlier than seven days and no later than 12 hours before the expected traffic increase.

## Pre-split limits

Pre-splitting your database has the following limitations:

  - You can't pre-split search indexes. You only need to pre-split the base table. For more information, see [Search index sharding](/spanner/docs/full-text-search/search-indexes#search_index_sharding) .

  - You cannot pre-split vector indexes. For more information about vector indexes, see [Vector index](/spanner/docs/find-approximate-nearest-neighbors#vector-index) .

  - To learn about the quotas for split points, see [Quotas and limits](/spanner/quotas#split-point-limits) .

## What's next?

  - [Create and manage split points](/spanner/docs/create-manage-split-points)
