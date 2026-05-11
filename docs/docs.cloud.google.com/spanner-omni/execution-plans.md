---
name: documents/docs.cloud.google.com/spanner-omni/execution-plans
uri: https://docs.cloud.google.com/spanner-omni/execution-plans
title: View Spanner Omni execution plans
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to view Spanner Omni query execution plans. Query execution plans can help you understand and troubleshoot query performance issues. To diagnose latency issues by using information from execution plans, see the Spanner documentation on [using metrics to diagnose latency](https://docs.cloud.google.com/spanner/docs/latency-metrics) .

To view a Spanner Omni execution plan, use the following command:

``` 
  spanner sql --database DATABASE_ID \
  --execute "EXPLAIN QUERY"
```

Replace the following:

  - `  DATABASE_ID  ` : the database identifier. For example, `MY_DATABASE` .

  - `  QUERY  ` : the query to execute. For example, `SELECT * FROM Songs WHERE SingerId = 1` .

To view a Spanner Omni execution plan that includes runtime statistics, use the following command:

``` 
  spanner sql --database DATABASE_ID \
  --execute "EXPLAIN ANALYZE QUERY"
```
