---
name: documents/docs.cloud.google.com/spanner-omni/execution-plans
uri: https://docs.cloud.google.com/spanner-omni/execution-plans
title: View Spanner Omni execution plans
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

This document describes how to view Spanner Omni query execution plans. Query execution plans can help you understand and troubleshoot query performance issues. To diagnose latency issues by using information from execution plans, see the Spanner documentation on [using metrics to diagnose latency](https://docs.cloud.google.com/spanner/docs/latency-metrics) .

To view a Spanner Omni execution plan, use the following command:

```
  spanner sql --database DATABASE_ID \
  --execute "EXPLAIN QUERY"
```

Replace the following:

- `DATABASE_ID` : the database identifier. For example, `MY_DATABASE` .

- `QUERY` : the query to execute. For example, `SELECT * FROM Songs WHERE SingerId = 1` .

To view a Spanner Omni execution plan that includes runtime statistics, use the following command:

```
  spanner sql --database DATABASE_ID \
  --execute "EXPLAIN ANALYZE QUERY"
```
