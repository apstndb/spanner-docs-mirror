---
name: documents/docs.cloud.google.com/spanner-omni/transaction-latency
uri: https://docs.cloud.google.com/spanner-omni/transaction-latency
title: Identify transactions that might cause high latencies
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Use the system insights dashboards and built-in statistics tables to identify transactions that cause high latencies in Spanner Omni. To ensure the consistency of multiple concurrent transactions, Spanner Omni uses [locks](https://docs.cloud.google.com/spanner/docs/transactions#locking) to control access to the data. Lock contention occurs when many transactions require frequent access to the same lock, leading to high latencies.

Spanner Omni operations acquire locks when the operations are part of a [read-write](https://docs.cloud.google.com/spanner/docs/transactions#read-write_transactions) transaction. Read-only transactions don't acquire locks.

To identify transactions that cause high latencies, follow these steps:

1.  [Check for a spike in latencies using system insights](https://docs.cloud.google.com/spanner-omni/transaction-latency#check-spike-latencies) .

2.  [Identify lock contention issues using the lock wait time metric](https://docs.cloud.google.com/spanner-omni/transaction-latency#check-lock-contention) .

3.  [Identify problematic transactions](https://docs.cloud.google.com/spanner-omni/transaction-latency#identify-contending-transactions) .

## Before you begin

If you haven't already, download and install the Spanner Omni console. For more information see [Spanner Omni downloads](https://docs.cloud.google.com/spanner-omni/download) and [Start the Spanner Omni console](https://docs.cloud.google.com/spanner-omni/use-console#start-the-console) .

## Check for a spike in latencies using system insights

Spanner Omni doesn't use Cloud Monitoring. Instead, use the [system insights dashboard](https://docs.cloud.google.com/spanner-omni/use-console#system_insights) in the Spanner Omni console or [Grafana](https://docs.cloud.google.com/spanner-omni/grafana-dashboards) . To check for latency spikes in the Spanner Omni console, do the following:

1.  In the Spanner Omni console, click **System Insights** in the navigation pane.

2.  In the system insights dashboard, check the latency charts for the following:
    
      - Request latency ( `P50` , `P90` , `P99` )
    
      - Transaction latency ( `P50` , `P90` , `P99` )
    
    Check the 99th percentile ( `P99` ) for write operations on the latency charts. If you observe a spike in latency without a corresponding spike in CPU utilization or errors, the latency is likely due to lock contention issues.

## Check for lock contention issues

To check if high latencies are due to lock contention, use the lock wait time metric available in the **System Insights** dashboard.

### Check for high lock wait time

Check for high lock wait time in the Spanner Omni console system insights dashboard:

1.  Locate the **Lock wait time** chart, which shows the total lock wait time for lock conflicts for the selected database in a 5-minute interval.

2.  Check if this metric shows an increase that correlates with the latency spike you observed.

### Analyze the lock wait data using system tables

After you confirm that lock contention is the cause of high latencies, use system statistics tables to analyze lock wait data and identify the transactions that are causing the contention. You can use the [Spanner Omni CLI](https://docs.cloud.google.com/spanner-omni/cli-quickstart) to [query statistics tables](https://docs.cloud.google.com/spanner/docs/introspection/query-statistics) in the Spanner documentation.

For details on lock statistics tables, see [Lock statistics](https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics) in the Spanner documentation.

## Identify contending transactions

To pinpoint the specific transactions that are contributing to high latencies, examine the transaction statistics for your database. Focus on transactions with high average latency. Optimize the transaction shape to reduce latencies. Consider applying the recommended practices to [reduce lock contention](https://docs.cloud.google.com/spanner/docs/introspection/lock-statistics#applying_best_practices_to_reduce_lock_contention) in the Spanner documentation.

For details on transaction statistics tables, see [Transaction statistics](https://docs.cloud.google.com/spanner/docs/introspection/transaction-statistics) in the Spanner documentation.

## What's next

  - [Use Prometheus alerts to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/prometheus-alerts) .

  - [Use Grafana dashboards to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/grafana-dashboards) .
