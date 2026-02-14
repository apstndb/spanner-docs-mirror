This page describes the latency metrics that Spanner provides. If your application experiences high latency, use these metrics to help you diagnose and resolve the issue.

You can view these metrics [in the Google Cloud console](/spanner/docs/monitoring-console) and [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

## Overview of latency metrics

The latency metrics for Spanner measure how long it takes for the Spanner service to process a request. The metric captures the actual amount of time that elapses, not the amount of CPU time that Spanner uses.

These latency metrics do not include latency that occurs outside of Spanner, such as network latency or latency within your application layer. To measure other types of latency, you can use Cloud Monitoring to [instrument your application with custom metrics](/monitoring/custom-metrics) .

You can view charts of latency metrics [in the Google Cloud console](/spanner/docs/monitoring-console#view-history) and [in the Cloud Monitoring console](/spanner/docs/monitoring-cloud) . You can view combined latency metrics that include both reads and writes, or you can view separate metrics for reads and writes.

Based on the latency of each request, Spanner groups the requests into percentiles. You can view latency metrics for 50th percentile and 99th percentile latency:

  - **50th percentile latency** : The maximum latency, in seconds, for the fastest 50% of all requests. For example, if the 50th percentile latency is 0.5 seconds, then Spanner processed 50% of requests in less than 0.5 seconds.
    
    This metric is sometimes called the *median latency* .

  - **99th percentile latency** : The maximum latency, in seconds, for the fastest 99% of requests. For example, if the 99th percentile latency is 2 seconds, then Spanner processed 99% of requests in less than 2 seconds.

### Latency and operations per second

When an instance processes a small number of requests during a period of time, the 50th and 99th percentile latencies during that time are not meaningful indicators of the instance's overall performance. Under these conditions, a very small number of outliers can drastically change the latency metrics.

For example, suppose that an instance processes 100 requests during an hour. In this case, the 99th percentile latency for the instance during that hour is the amount of time it took to process the slowest request. A latency measurement based on a single request is not meaningful.

## How to diagnose latency issues

The following sections describe how to diagnose several common issues that could cause your application to experience high end-to-end latency.

For a quick look at an instance's latency metrics, [use the Google Cloud console](/spanner/docs/monitoring-console) . To examine the metrics more closely and [find correlations](/spanner/docs/monitoring-cloud#create-charts) between latency and other metrics, [use the Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

### High total latency, low Spanner latency

If your application experiences latency that is higher than expected, but the latency metrics for Spanner are significantly lower than the total end-to-end latency, there might be an issue in your application code. If your application has a performance issue that causes some code paths to be slow, the total end-to-end latency for each request might increase.

To check for this issue, benchmark your application to identify code paths that are slower than expected.

You can also comment out the code that communicates with Spanner, then measure the total latency again. If the total latency doesn't change very much, then Spanner is unlikely to be the cause of the high latency.

### High total latency, high Spanner latency

If your application experiences latency that is higher than expected, and the Spanner latency metrics are also high, there are a few likely causes:

  - **Your instance needs more compute capacity.** If your instance does not have enough CPU resources, and its CPU utilization exceeds the [recommended maximum](/spanner/docs/cpu-utilization#recommended-max) , then Spanner might not be able to process your requests quickly and efficiently.

  - **Some of your queries cause high CPU utilization.** If your queries do not take advantage of Spanner features that improve efficiency, such as [query parameters](/spanner/docs/reference/standard-sql/lexical#query_parameters) and [secondary indexes](/spanner/docs/secondary-indexes) , or if they include a large number of [joins](/spanner/docs/reference/standard-sql/query-syntax#join_types) or other CPU-intensive operations, the queries can use a large portion of the CPU resources for your instance.

To check for these issues, use the Cloud Monitoring console to [look for a correlation](/spanner/docs/monitoring-cloud#create-charts) between high CPU utilization and high latency. Also, check the [query statistics](/spanner/docs/introspection/query-statistics) for your instance to identify any CPU-intensive queries during the same time period.

If you find that CPU utilization and latency are both high at the same time, take action to address the issue:

  - If you did not find many CPU-intensive queries, [add compute capacity to the instance](/spanner/docs/create-manage-instances#change-compute-capacity) .
    
    Adding [compute capacity](/spanner/docs/compute-capacity) provides more CPU resources and enables Spanner to handle a larger workload.

  - If you found CPU-intensive queries, review the [query execution plans](/spanner/docs/query-execution-plans) to learn why the queries are slow, then update your queries to follow the [SQL best practices for Spanner](/spanner/docs/sql-best-practices) .
    
    You might also need to review the [schema design](/spanner/docs/schema-design) for the database and update the schema to allow for more efficient queries.

## What's next

  - Monitor your instance with the [Google Cloud console](/spanner/docs/monitoring-console) or the [Cloud Monitoring console](/spanner/docs/monitoring-cloud) .

  - Learn how to
    
    [find correlations between high latency and other metrics](/spanner/docs/monitoring-cloud#create-charts) .

  - Understand how to reduce read latency by following [SQL best practices](/spanner/docs/sql-best-practices) and using [timestamp bounds](/spanner/docs/timestamp-bounds) .

  - Find out about [latency metrics in query statistics tables](/spanner/docs/introspection/query-statistics) , which you can retrieve using SQL statements.

  - Understand [how instance configuration affects latency](/spanner/docs/instances) .
