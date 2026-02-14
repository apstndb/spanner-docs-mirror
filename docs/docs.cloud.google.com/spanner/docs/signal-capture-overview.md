This document describes telemetry signals, including metrics, logs, traces, and statistics, that are captured for Spanner.

Spanner generates observability data, including metrics, logs, traces, and statistics. You can set up capturing for some signals that are not captured. You can use these signals to get a complete picture of how your system is performing.

## Spanner metrics

Metrics are numeric data about database health or performance, such as request latency or CPU utilization, that you measure at regular intervals over time.

[Cloud Monitoring](/monitoring/docs/monitoring-overview) regularly measures your service and usage of the Google Cloud resources. To view all the server-side and client-side metrics collected by Spanner, see [Metrics list](/spanner/docs/metrics) .

Spanner includes client-side metrics, such as latencies for Google Front End (GFE) and API Frontend (AFE). For more information, see [Client-side metrics descriptions](/spanner/docs/client-side-metrics-descriptions) .

If you need custom client-side metrics in addition to the metrics collected by Spanner, such as query latency, you can configure custom client-side metrics using [OpenTelemetry](/spanner/docs/capture-custom-metrics-opentelemetry) .

Custom client-side metrics can incur charges through your observability backend. For example, if you use Cloud Monitoring as your backend, you are billed according to [Cloud Monitoring pricing](https://cloud.google.com/stackdriver/pricing#monitoring-pricing-summary) .

## Spanner logs

A log is a generated record of system activity over time. Each log is a collection of time-stamped log entries, and each log entry describes an event at a specific point in time. For more information about enabling logs, see [Enable Data Access audit logs](/logging/docs/audit/configure-data-access) . [Cloud Logging](/logging/docs/overview) collects logging data from common application components. For a list of log types collected by Spanner, see [Spanner audit logs](/spanner/docs/audit-logging) .

## Spanner traces

Traces represent the path of a request through your application. Traces let you follow the flow of a request and help you identify the root cause of an issue. You can configure Spanner client libraries to export client-side and server-side traces using OpenTelemetry APIs. For more information about trace collection using OpenTelemetry, see [Trace collection overview](/spanner/docs/tracing-overview) .

## Spanner statistics tables

Spanner offers a set of built-in statistic tables that you can query to gain more information about the following:

  - Queries
  - Reads
  - Transactions
  - Locks
  - Table sizes
  - Table operations

For more information about the available tables, see [Spanner built-in statistics tables overview](/spanner/docs/introspection) .

## What's next

  - [Spanner metrics list](/spanner/docs/metrics)
  - [Client-side metrics overview](/spanner/docs/view-manage-client-side-metrics)
  - [Set up trace collection using OpenTelemetry](/spanner/docs/set-up-tracing)
  - [Spanner built-in statistics tables overview](/spanner/docs/introspection)
  - [OpenTelemetry documentation](https://opentelemetry.io/docs/what-is-opentelemetry/)
