---
name: documents/docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions
uri: https://docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions
title: Client-side metrics descriptions
description: Detailed descriptions of the client-side metrics available for Spanner applications
data_source: docs.cloud.google.com
---

This page provides a detailed description of each client-side metric that Spanner offers. For more information about viewing and managing client-side metrics, see [View and manage client-side metrics](https://docs.cloud.google.com/spanner/docs/view-manage-client-side-metrics) .

Client-side metrics offer latency information about the client's RPC request *attempts* and *operations* to Spanner.

An attempt is a single RPC request from the client to Spanner. The client can make multiple attempts if a request fails.

An operation represents completion of all RPC requests, including all the attempts the client made to Spanner. An operation is completed when the client receives a response or exhausts all attempts.

An RPC request first reaches the Google Front End (GFE) and then the Spanner API frontend (AFE). For more information, see [Latency points in a Spanner request](https://docs.cloud.google.com/spanner/docs/latency-points) .

The client calculates the GFE latencies and AFE latencies metrics from timing information that the GFE and the AFE add to the response headers. If an attempt ends before the client receives response headers, the client doesn't record a GFE or AFE latency sample for that attempt. For example, this happens when the request deadline expires and the client returns a `DEADLINE_EXCEEDED` error. Instead, the client increments the corresponding connectivity error count metric. As a result, the GFE and AFE latency distributions include only attempts that received response headers and might look normal even when many attempts time out.

The client records GFE metrics only for requests that go through the GFE. If a request uses DirectPath ( `directpath_used` is `true` ), the client doesn't record GFE latency or the GFE connectivity error count for that request.

These metrics include `DISTRIBUTION` and `INT64` metric value types. For more information, see [Value types and metric kinds](https://docs.cloud.google.com/monitoring/api/v3/kinds-and-types) .

You can access client-side metrics using the Spanner client libraries for Java, Go, Node.js and Python.

The client-side metrics support all Spanner methods in the [`google.spanner.v1.Spanner`](https://docs.cloud.google.com/spanner/docs/reference/rpc#google.spanner.v1.spanner) RPC API.

All client-side metrics have the following dimensions:

- `project ID` : the Google Cloud project ID.
- `instance ID` : the Spanner instance ID.
- `database ID` : the Spanner database ID.
- `location` : the Google Cloud region where the client-side metrics are published. If your application is deployed outside Google Cloud, then the metrics are published to the `global` region.
- `method` : the RPC method name—for example, `spanner.commit` .
- `status` : the RPC status—for example, `OK` or `INTERNAL` .
- `client_name` : the library name and version—for example, `spanner-java/6.70.1` .
- `directpath_enabled` : whether the client library is configured to directly access Spanner without going through the frontend.
- `directpath_used` : whether the current request was sent directly to Spanner without going through the frontend. A `false` value while `directpath_enabled` is `true` indicates the request fell back to the standard path going through the GFE.

## AFE connectivity error count

The AFE connectivity error count metric records the number of attempts for which the client didn't receive Spanner AFE timing information in the response headers. This happens, for example, when the request doesn't reach the AFE or when the attempt times out before the client receives response headers. The [AFE latencies](https://docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions#afe-latencies) metric doesn't record a sample for these attempts.

The AFE connectivity error count metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/afe_connectivity_error_count`
- `Metric value type` : `INT64`

## AFE latencies

The AFE latencies metric measures the time in between the Spanner AFE receiving the RPC request from the client and Spanner AFE starts writing the response.

The client calculates this value from timing information in the response headers. If an attempt ends before the client receives response headers, the client doesn't record a sample and increments the [AFE connectivity error count](https://docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions#afe-connectivity-error-count) metric instead.

The AFE latencies metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/afe_latencies`
- `Metric value type` : `DISTRIBUTION`

## Attempt count

The attempt count metric records the number of attempts performed. The `attempt_count` value is typically identical to the `operation_count` value. If the client receives transient errors, the `attempt_count` value is the sum of all attempts.

The Attempt count metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/attempt_count`
- `Metric value type` : `INT64`

## Attempt latencies

The attempt latencies metric measures the duration of a single RPC attempt, from when the client sends the request until the attempt succeeds or fails. The `attempt_latencies` value is typically identical to the `operation_latencies` value. If the client receives transient errors, then the `operation_latencies` value is a sum of all the `attempt_latencies` values.

The Attempt latencies metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/attempt_latencies`
- `Metric value type` : `DISTRIBUTION`

## GFE connectivity error count

The GFE connectivity error count metric records the number of attempts for which the client didn't receive GFE timing information in the response headers. This happens, for example, when the request doesn't reach the GFE or when the attempt times out before the client receives response headers. The [GFE latencies](https://docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions#gfe-latencies) metric doesn't record a sample for these attempts.

The GFE connectivity error count metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/gfe_connectivity_error_count`
- `Metric value type` : `INT64`

## GFE latencies

The GFE latencies metric measures the time in between the GFE receiving the RPC request from the client and when the GFE receives the first byte of the response. This metric doesn't include the time taken for any TCP or TLS handshake.

The client calculates this value from timing information in the response headers. If an attempt ends before the client receives response headers, the client doesn't record a sample and increments the [GFE connectivity error count](https://docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions#gfe-connectivity-error-count) metric instead. If the GFE connectivity error count increases, GFE latencies that look normal don't mean that latency is low.

The GFE latencies metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/gfe_latencies`
- `Metric value type` : `DISTRIBUTION`

## Operation count

The operation count metric records the number of operations performed every 60 seconds.

The Operation count metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/operation_count`
- `Metric value type` : `INT64`

## Operation latencies

The operation latencies metric measures an operation's round trip from the client [gRPC](https://grpc.io/) layer to Spanner and back to the client.

The Operation latencies metric has the following attributes:

- `Metric name` : `spanner.googleapis.com/client/operation_latencies`
- `Metric value type` : `DISTRIBUTION`

## What's next

- [Client metrics overview](https://docs.cloud.google.com/spanner/docs/view-manage-client-side-metrics)
- [Signal capture overview](https://docs.cloud.google.com/spanner/docs/signal-capture-overview)
