This page shows you how to identify and troubleshoot latency issues in your Spanner components.To learn more about possible latency points in a Spanner request, see [Latency points in a Spanner request](/spanner/docs/latency-points) .

You can measure and compare the request latencies between different components and the database to determine which component is causing the latency. These latencies include [End-to-end latency](/spanner/docs/latency-points#end-to-end-latency) , [Google Front End (GFE) latency](/spanner/docs/latency-points#google-front-end-latency) , [Spanner API request latency](/spanner/docs/latency-points#spanner-api-request-latency) , and [Query latency](/spanner/docs/latency-points#query-latency) .

**Note:** You can also use OpenTelemetry to capture and visualize end-to-end, GFE latency, and query latency. For more information, see [Capture custom client-side metrics using OpenTelemetry](/spanner/docs/capture-custom-metrics-opentelemetry) .

1.  In your client application that uses your service, confirm there's a latency increase from end-to-end latency. Check the following dimensions from your client-side metrics. For more information, see [Client-side metrics descriptions](/spanner/docs/client-side-metrics-descriptions) .
    
      - `  client_name  ` : the client library name and version.
      - `  location  ` : the Google Cloud region where the client-side metrics are published. If your application is deployed outside Google Cloud, then the metrics are published to the `  global  ` region.
      - `  method  ` : the RPC method name—for example, `  spanner.commit  ` .
      - `  status  ` : the RPC status—for example, `  OK  ` or `  INTERNAL  ` .
    
    Group by these dimensions to see if the issue is limited to a specific client, status, or method. For dual-region or multi-regional workloads, see if the issue is limited to a specific client or Spanner region.

2.  Check your client application health, especially the computing infrastructure on the client side (for example, VM, CPU, or memory utilization, connections, file descriptors, and so on).

3.  Check latency in Spanner components by viewing the [client-side metrics](/spanner/docs/view-manage-client-side-metrics) :
    
    a. Check end-to-end latency using the [`  spanner.googleapis.com/client/operation_latencies  `](/spanner/docs/client-side-metrics-descriptions#operation-latencies) metric.
    
    b. Check [Google Front End](https://cloud.google.com//security/infrastructure/design#google-frontend-service.) (GFE) latency using the [`  spanner.googleapis.com/client/gfe_latencies  `](/spanner/docs/client-side-metrics-descriptions#gfe-latencies) metric.

4.  Check the following dimensions for [Spanner metrics](/spanner/docs/latency-metrics) :
    
      - `  database  ` : the Spanner database name.
      - `  method  ` : the RPC method name—for example, `  spanner.commit  ` .
      - `  status  ` : the RPC status—for example, `  OK  ` or `  INTERNAL  ` .
    
    Group by these dimensions to see if the issue is limited to a specific database, status, or method. For dual-region or multi-regional workloads, check to see if the issue is limited to a specific region.

5.  Check Spanner API request latency using the `  spanner.googleapis.com/api/request_latencies  ` metric. For more information, see [Spanner metrics](/spanner/docs/metrics) .
    
    If you have high end-to-end latency, but low GFE latency, and a low Spanner API request latency, the application code might have an issue. It could also indicate a networking issue between the client and regional GFE. If your application has a performance issue that causes some code paths to be slow, then the end-to-end latency for each API request might increase. There might also be an issue in the client computing infrastructure that was not detected in the previous step.
    
    If you have a high GFE latency, but a low Spanner API request latency, it might have one of the following causes:
    
      - Accessing a database from another region. This action can lead to high GFE latency and low Spanner API request latency. For example, traffic from a client in the `  us-east1  ` region that has an instance in the `  us-central1  ` region might have a high GFE latency but a lower Spanner API request latency.
    
      - There's an issue at the GFE layer. Check the [Google Cloud Status Dashboard](https://status.cloud.google.com/) to see if there are any ongoing networking issues in your region. If there aren't any issues, then open a support case and include this information so that support engineers can help with troubleshooting the GFE.

6.  [Check the CPU utilization of the instance](/spanner/docs/cpu-utilization) . If the CPU utilization of the instance is above the recommended level, you should manually add more nodes, or set up auto scaling. For more information, see [Autoscaling overview](/spanner/docs/autoscaling-overview) .

7.  Observe and troubleshoot potential hotspots or unbalanced access patterns using [Key Visualizer](/spanner/docs/key-visualizer) and try to roll back any application code changes that strongly correlate with the issue timeframe.
    
    **Note:** We recommend you follow [Schema design best practices](/spanner/docs/schema-design) to ensure your access is balanced across Spanner computing resources.

8.  Check any traffic pattern changes.

9.  Check [Query insights](/spanner/docs/using-query-insights) and [Transaction insights](/spanner/docs/use-lock-and-transaction-insights) to see if there might be any query or transaction performance bottlenecks.

10. Use procedures in [Oldest active queries](/spanner/docs/introspection/oldest-active-queries) to see any expense queries that might cause a performance bottleneck and cancel the queries as needed.

11. Use procedures in the troubleshooting sections in the following topics to troubleshoot the issue further using Spanner introspection tools:
    
      - [Query statistics](/spanner/docs/introspection/query-statistics)
      - [Read statistics](/spanner/docs/introspection/read-statistics)
      - [Transaction statistics](/spanner/docs/introspection/transaction-statistics)
      - [Lock statistics](/spanner/docs/introspection/lock-statistics)

## What's next

  - Now that you've identified the component that contains the latency, explore the problem further using the [built-in client-side metrics](/spanner/docs/view-manage-client-side-metrics) .
  - Learn how to use [metrics](/spanner/docs/latency-metrics) to diagnose latency.
  - Learn how to [troubleshoot Spanner deadline exceeded errors](/spanner/docs/deadline-exceeded) .
