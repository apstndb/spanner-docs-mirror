This page describes the metrics that you can view in Key Visualizer heatmaps.

## Metrics

Key Visualizer automtically captures and aggregates metrics about performance and resource usage for Spanner databases that have enabled it. All metrics in Key Visualizer are normalized by the number of rows. This means that the metric value inside a row range is the aggregated count divided by the number of rows inside the range.

<table>
<thead>
<tr class="header">
<th>Metrics</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>CPU seconds</td>
<td>Number of CPU seconds spent per (wall clock) second reading / writing in a row range.</td>
</tr>
<tr class="even">
<td>Logical stored bytes</td>
<td>Bytes used to store data in a row range.</td>
</tr>
<tr class="odd">
<td>Number of rows read</td>
<td>Number of rows read per second from a row range.</td>
</tr>
<tr class="even">
<td>Number of bytes read</td>
<td>Number of bytes read (not bytes returned) per second from a row range.</td>
</tr>
<tr class="odd">
<td>Number of rows written</td>
<td>Number of rows inserted / updated per second in a row range.</td>
</tr>
<tr class="even">
<td>Number of bytes written</td>
<td>Number of bytes inserted / updated per second in a row range.</td>
</tr>
</tbody>
</table>

## Time window granularity

Spanner Key Visualizer aggregates metrics at several different granularities over time. The most recent data is aggregated at the finest granularity, while earlier data is aggregated more coarsely.

<table>
<thead>
<tr class="header">
<th>Period</th>
<th>Granularity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Past 6 hours</td>
<td>1 minute</td>
</tr>
<tr class="even">
<td>Past 4 days</td>
<td>10 minutes</td>
</tr>
<tr class="odd">
<td>Past 28 days</td>
<td>1 hour</td>
</tr>
</tbody>
</table>

## Data retention and wipeout

Spanner stores key ranges and metrics in an internal metadata table as part of the user's database. Spanner Key Visualizer data retention is as follows:

  - Data that is collected every minute will be deleted after six hours.
  - Data that is collected every ten minutes will be deleted after four days.
  - Data that is collected every hour will be deleted after 28 days.

Key Visualizer stores row key data in the metadata table. If your data retention requirements aren't compatible with Key Visualizer's data retention schedule, you can [disable Key Visualizer](/spanner/docs/key-visualizer/getting-started#enabling_key_visualizer) .

## Limitations

There are some situations in which new data might not be immediately available in Key Visualizer.

  - Storage and traffic statistics for newly created databases might not be available immediately.
  - Traffic statistics for newly inserted rows might be attributed to existing key ranges.

## What's next

  - [Get started with Key Visualizer](/spanner/docs/key-visualizer/getting-started) .
  - Learn [how Key Visualizer displays data in heatmaps](/spanner/docs/key-visualizer/exploring-heatmaps#heatmaps) .
  - Find out [how to explore Key Visualizer heatmaps](/spanner/docs/key-visualizer/exploring-heatmaps) .
