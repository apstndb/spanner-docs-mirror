**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes the built-in Spanner table that stores statistics about your vector indexes. You can use vector index statistics to review the performance of your vector index, identify areas for improvement, and tune your index based on the metrics.

You can retrieve statistics from the `  SPANNER_SYS.VECTOR_INDEX_STATS  ` table using SQL statements.

Spanner provides the vector index statistics in the `  SPANNER_SYS  ` schema. You can use the following ways to access `  SPANNER_SYS  ` data:

  - A database's Spanner Studio page in the Google Cloud console.

  - The [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) command.

  - The [`  executeSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql) or the [`  executeStreamingSql  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) method.

## Vector index statistics

`  SPANNER_SYS.VECTOR_INDEX_STATS  ` contains the statistics about each vector index in your database, sorted by `  STATISTICS_COLLECTION_TIMESTAMP  ` . Spanner collects data about each vector index every three days. Shortly after data is collected, Spanner makes the data available in the `  VECTOR_INDEX_STATS  ` table. The information found in this table provides insights into your index, allowing you to tune and apply recommendations for faster vector search performance.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Description</th>
<th>Tuning guidance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       START_TIME      </code></td>
<td>The time at which Spanner collected the index metric.</td>
<td>Not applicable</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       VECTOR_INDEX_NAME      </code></td>
<td>The name of the vector index, as given in the <code dir="ltr" translate="no">       CREATE VECTOR INDEX      </code> statement.</td>
<td>Not applicable</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       NUM_LEAVES      </code></td>
<td>The total number of leaves (also known as clusters) used in this vector index. This number is determined at the time of vector index creation. It might change over time as Spanner adaptively reshapes the vector index.</td>
<td>If many rows are added to the indexed table after creation, it's possible that some clusters might become quite large, which can impact query latency. We recommend rebuilding the index with more clusters if the number of clusters in the index is equal to or more than the square root of the number of rows in the table.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       NUM_ZERO_SIZE_CLUSTERS_SAMPLED      </code></td>
<td>The sampled number of empty clusters in the vector index. Empty clusters don't have any rows assigned to them.</td>
<td><p>As rows are added and deleted from a table over time, some clusters might become empty. If there are many empty clusters, the vector index is likely stale. If 10% or more of the clusters are empty, consider rebuilding the index with fewer clusters. For the larger vector indexes, Spanner uses random sampling to bound the cost of measurements. Therefore, this metric represents the number of sampled size-zero clusters. To convert this to the estimated number of size-zero clusters, you can use the following query:</p>
<p><code dir="ltr" translate="no">        SELECT NUM_ZERO_SIZE_CLUSTERS_SAMPLED / NUM_CLUSTERS_SAMPLED AS ESTIMATED_PERCENT_OF_CLUSTERS_WITH_SIZE_ZERO       </code></p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       CLUSTER_SIZE_PERCENTILES      </code></td>
<td>A vector index divides the indexed embeddings into clusters of nearby vectors. This distribution column gives some percentile values of the sizes of these clusters.</td>
<td>The K-means clustering process, which powers vector indexing, typically produces clusters of varying size. As a result, some imbalance is expected. If there are severe deviations in cluster size, query performance becomes slower when searching the larger clusters. Deviations are severe if the 99th percentile cluster size is at least 8 times larger than the 50th percentile cluster size. See <a href="#example-query-1">example query 1</a> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       CLUSTER_AVERAGE_DISTANCE_TO_CENTROID_PERCENTILES      </code></td>
<td>Each cluster has a center point, or <em>centroid</em> . To achieve good recall, vectors are generally assigned to a nearby centroid, according to the configuration you chose during index creation. For each cluster, Spanner computes the estimated average distance from the centroid to all assigned vectors. Due to natural variance among clusters, Spanner reports this metric as a distribution, given as a list of percentile values.</td>
<td>This metric is not comparable between vector indexes (in particular, Euclidean indexes and cosine indexes are scaled differently). Track this metric over time for each vector index individually. If the 99th percentile value doubles from its historical low, consider rebuilding the index to bring it back down. See <a href="#example-query-2">example query 2</a> .</td>
</tr>
</tbody>
</table>

### Example query 1

The following query reports the 99th percentile cluster size divided by the median cluster size for a specific vector index, over time. If the 99th percentile cluster size is at least 8 times larger than the 50th percentile cluster size, consider rebuilding your index."

``` text
 SELECT VECTOR_INDEX_NAME,
        START_TIME,
        SUM(CASE WHEN percentile = 99 THEN value_at_percentile ELSE 0 END)
        / SUM(CASE WHEN percentile = 50 THEN value_at_percentile ELSE 0 END) as cluster_size_imbalance
FROM (
  SELECT v.VECTOR_INDEX_NAME,
         v.START_TIME, d.percentile,
         d.value_at_percentile
  FROM SPANNER_SYS.VECTOR_INDEX_METRICS_HISTORY AS v,
  UNNEST(v.CLUSTER_AVERAGE_DISTANCE_TO_CENTROID_PERCENTILES)
AS d)
WHERE VECTOR_INDEX_NAME = "VecIndex2"
GROUP BY 1, 2;
```

**Query output**

<table>
<thead>
<tr class="header">
<th>VECTOR_INDEX_NAME</th>
<th>START_TIME</th>
<th>cluster_size_imbalance</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>VecIndex2</td>
<td>2025-12-10 10:09:55.322415+00:00</td>
<td>3.288</td>
</tr>
<tr class="even">
<td>VecIndex2</td>
<td>2025-12-07 10:10:02.362223+00:00</td>
<td>3.11</td>
</tr>
<tr class="odd">
<td>VecIndex2</td>
<td>2025-12-04 10:09:55.180895+00:00</td>
<td>2.98</td>
</tr>
<tr class="even">
<td>VecIndex2</td>
<td>2025-12-01 10:10:01.680543+00:00</td>
<td>3.01</td>
</tr>
<tr class="odd">
<td>VecIndex2</td>
<td>2025-11-28 10:10:02.130079+00:00</td>
<td>2.99</td>
</tr>
</tbody>
</table>

### Example query 2

The following query reports the 99th percentile distance to centroid for each vector index, over time. This metric is an indirect indication of recall performance. The exact meaning of the values depends on your embeddings. Track each vector index individually, and as a time series. If the 99th percentile value doubles from its historical low for a given index, consider rebuilding the index.

``` text
SELECT v.VECTOR_INDEX_NAME,
       v.START_TIME,
       d.value_at_percentile AS distance_to_centroid_99p
FROM SPANNER_SYS.VECTOR_INDEX_METRICS_HISTORY AS v,
UNNEST(v.CLUSTER_AVERAGE_DISTANCE_TO_CENTROID_PERCENTILES) AS d
WHERE d.percentile=99
ORDER BY VECTOR_INDEX_NAME, START_TIME DESC;
```

**Query output**

<table>
<thead>
<tr class="header">
<th>VECTOR_INDEX_NAME</th>
<th>START_TIME</th>
<th>distance_to_centroid_99p</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>ANNVectorBase_VI</td>
<td>2025-12-10 10:09:55.322415+00:00</td>
<td>1.109</td>
</tr>
<tr class="even">
<td>ANNVectorBase_VI</td>
<td>2025-12-07 10:10:02.362223+00:00</td>
<td>1.11</td>
</tr>
<tr class="odd">
<td>ANNVectorBase_VI</td>
<td>2025-12-04 10:09:55.180895+00:00</td>
<td>1.18</td>
</tr>
<tr class="even">
<td>ANNVectorBase_VI</td>
<td>2025-12-01 10:10:01.680543+00:00</td>
<td>1.11</td>
</tr>
<tr class="odd">
<td>ANNVectorBase_VI</td>
<td>2025-11-28 10:10:02.130079+00:00</td>
<td>1.04</td>
</tr>
</tbody>
</table>

## Data retention

The `  SPANNER_SYS.VECTOR_INDEX_METRICS_HISTORY  ` system table retains data covering the previous 30 days.

## What's next

  - Learn how to [rebuild a vector index](/spanner/docs/find-approximate-nearest-neighbors#rebuild) .
