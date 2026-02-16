This page discusses Spanner time to live (TTL) metrics. To learn more, see [About TTL](/spanner/docs/ttl) .

## Metrics

Spanner provides information about TTL activities in a system table that can be read with SQL queries, and as metrics accessed through [Cloud Monitoring](/spanner/docs/monitoring-cloud) .

The system table reports TTL information per table for a database, while Cloud Monitoring reports metrics at a database level.

### Use a SQL query

Spanner provides a built-in table that tracks information related to TTL. The table is named `  SPANNER_SYS.ROW_DELETION_POLICIES  ` and has the following schema.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>TABLE_NAME</td>
<td>STRING</td>
<td>The name of the table that contains this TTL policy.</td>
</tr>
<tr class="even">
<td>PROCESSED_WATERMARK</td>
<td>TIMESTAMP</td>
<td>This policy has run against all rows in the table as of this time. Some table partitions may have been processed more recently, so this timestamp represents the least-recently processed partition. Typically, this is within 72 hours.</td>
</tr>
<tr class="odd">
<td>UNDELETABLE_ROWS</td>
<td>INT64</td>
<td>The number of rows that cannot be deleted by the TTL policy. See <a href="#undeletable_rows">Undeletable rows</a> for more details.</td>
</tr>
<tr class="even">
<td>MIN_UNDELETABLE_TIMESTAMP</td>
<td>TIMESTAMP</td>
<td>The oldest timestamp for undeletable rows that was observed during the last processing cycle.</td>
</tr>
</tbody>
</table>

The deletion policy information is returned per table for your database.

You can query this data with a SQL query similar to the following:

``` text
SELECT TABLE_NAME, UNDELETABLE_ROWS
FROM SPANNER_SYS.ROW_DELETION_POLICIES
WHERE UNDELETABLE_ROWS > 0
```

The `  SPANNER_SYS  ` tables are only accessible through SQL interfaces; for example:

  - The **Spanner Studio** page in the Google Cloud console
  - The `  gcloud spanner databases execute-sql  ` command
  - The `  executeQuery  ` API

Other single read methods that Spanner provides don't support `  SPANNER_SYS  ` .

### Use Cloud Monitoring

Spanner provides the following metrics to monitor TTL activity at a database level:

  - `  row_deletion_policy/deleted_rows  ` is the number of rows deleted by the TTL policy.
  - `  row_deletion_policy/undeletable_rows  ` is the number of rows that match the row deletion (GoogleSQL) or `  TTL INTERVAL  ` (PostgreSQL) statement, but that cannot be deleted. This is usually because the row had too many child rows, causing the action to exceed Spanner's [transaction limit](/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) .
  - `  row_deletion_policy/processed_watermark_age  ` is the time between now and the read timestamp used by the last successful cycle (with or without undeletable rows).

These metrics are available through [Cloud Monitoring](/spanner/docs/monitoring-cloud) and the [Google Cloud console](/spanner/docs/monitoring-console) .

## Monitor

You can also monitor other TTL activities.

### Find last successful scan

You can find the last snapshot time at which Spanner completed a scan of the table looking for expired rows. To do so as a SQL query:

``` text
SELECT PROCESSED_WATERMARK
FROM SPANNER_SYS.ROW_DELETION_POLICIES
WHERE TABLE_NAME = $name
```

Alternatively, the `  row_deletion_policy/process_watermark_age  ` metric displays similar information, but is expressed as the difference between the current time and the last scan time. The metric is not broken down by table, but represents the oldest scan time of any TTL-enabled tables in the database.

Rows that match a TTL policy are typically deleted within 72 hours of their expiration date. You can [set an alert](/monitoring/alerts/using-alerting-ui) on `  processed_watermark_age  ` so that you are notified if it exceeds 72 hours.

If `  processed_watermark_age  ` is older than 72 hours, it may indicate that higher-priority tasks are preventing TTL from running. In this case, we recommend checking [CPU utilization](/spanner/docs/cpu-utilization#recommended-max) and [adding more compute capacity](/spanner/docs/create-manage-instances#change-compute-capacity) if required. If CPU utilization is within the recommended range, check for hotspotting using [Key Visualizer](/spanner/docs/key-visualizer) .

### Monitor deleted rows

To monitor TTL activity on your table, graph the `  row_deletion_policy/deleted_rows  ` metric. This metric displays the number of rows deleted over time.

If no data has expired, this metric is empty.

### Monitor undeletable rows

When TTL is unable to delete a row, Spanner automatically retries. If, upon retry, the TTL action cannot be processed, Spanner skips the row and reports it in the `  row_deletion_policy/undeletable_rows_count  ` metric.

You can [set an alert](/monitoring/alerts/using-alerting-ui#create-policy) on the `  row_deletion_policy/undeletable_rows_count  ` to be notified of a non-zero count.

If you find a non-zero count, you can create a query to break down the count by table:

``` text
SELECT TABLE_NAME, UNDELETABLE_ROWS, MIN_UNDELETABLE_TIMESTAMP
FROM SPANNER_SYS.ROW_DELETION_POLICIES
WHERE UNDELETABLE_ROWS > 0
```

To look up the contents of the undeletable row:

``` text
SELECT *
FROM $TABLE_NAME
WHERE $EXPIRE_COL >= $MIN_UNDELETABLE_TIMESTAMP
```

Most commonly, a row deletion failure is due to cascading updates to interleaved tables and indexes such that the resulting transaction size exceeds Spanner's [mutation limits](/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . You can resolve the issue by updating your schema to add separate [TTL policies on interleaved tables](/spanner/docs/ttl/working-with-ttl#ttl_and_interleaved_tables) .
