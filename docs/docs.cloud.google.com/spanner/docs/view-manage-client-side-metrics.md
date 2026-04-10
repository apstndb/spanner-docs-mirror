This page offers information about viewing and managing client-side metrics. Client-side metrics offer latency information about the client's RPC requests to Spanner.

Spanner provides client-side metrics that you can use along with server-side metrics to optimize performance and troubleshoot performance issues if they occur.

Client-side metrics are measured from the time a request leaves your application to the time your application receives the response. In contrast, server-side metrics are measured from the time Spanner receives a request until the last byte of data is sent to the client.

## Before you begin

1.  Client-side metrics are available after you enable the Cloud Monitoring API.
    
    [Enable the Monitoring API](https://console.cloud.google.com/flows/enableapi?apiid=monitoring.googleapis.com)

2.  To ensure that your service account has the necessary permission to access client-side metrics, ask your administrator to grant the [Monitoring Metric Writer](https://docs.cloud.google.com/iam/docs/roles-permissions/monitoring#monitoring.metricWriter) ( `  roles/monitoring.metricWriter  ` ) IAM role to your service account on the project.
    
    **Important:** You must grant this role to your service account, *not* to your user account. Failure to grant the role to the correct principal might result in permission errors.
    
    For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .
    
    This predefined role contains the `  monitoring.timeSeries.create  ` permission, which is required to access client-side metrics.
    
    Your administrator might also be able to give your service account this permission with [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

## Access client-side metrics

You can access client-side metrics using the following client libraries:

  - [Go](https://pkg.go.dev/cloud.google.com/go/spanner) in version 1.71.0 and later.
  - [Java](https://docs.cloud.google.com/java/docs/reference/google-cloud-spanner/latest/overview) in version 6.81.0 and later.
  - [Node](https://docs.cloud.google.com/nodejs/docs/reference/spanner/latest/overview) in version 8.1.0 or later.
  - [Python](https://docs.cloud.google.com/python/docs/reference/spanner/latest/overview) in version 3.60.0 or later.

To view the client-side metrics in the Metrics Explorer, do the following:

1.  In the Google Cloud console, go to the Metrics Explorer page.
    
    [Go to Metrics Explorer](https://console.cloud.google.com/projectselector/monitoring/metrics-explorer?supportedpurview=project,folder,organizationId)

2.  Select your project.

3.  Click **Select a metric** .

4.  Search for `  spanner.googleapis.com/client  ` .

5.  Select the metric, and then click **Apply** .

For more information about grouping or aggregating your metric, see [Build queries using menus](https://docs.cloud.google.com/monitoring/charts/metrics-selector#basic-advanced-mode) .

Your application needs to run for at least a minute before you can view any published metrics.

## Opt out of client-side metrics

If you are already using OpenTelemetry to capture custom client metrics, you can choose to opt out of using client-side metrics by using the following code:

### Go

``` 
    client, err := spanner.NewClientWithConfig(ctx, database, spanner.ClientConfig{
    DisableNativeMetrics: true,
    })
```

### Java

    Spanner spanner =
      SpannerOptions.newBuilder()
        .setProjectId("test-project")
        .setBuiltInMetricsEnabled(false)
        .build()
        .getService();

### Node.js

``` 
  const spanner = new Spanner({
      disableBuiltInMetrics: true
  });
```

### Python

    spanner_client = spanner.Client(
        disable_builtin_metrics=True
    )

## Pricing

There is no charge to view client-side metrics in Cloud Monitoring. Use of the Monitoring API might incur charges. For more information, see [Google Cloud Observability pricing](https://cloud.google.com/stackdriver/pricing) .

## What's next

  - [Client-side metrics descriptions](https://docs.cloud.google.com/spanner/docs/client-side-metrics-descriptions)
  - [Trace collection overview](https://docs.cloud.google.com/spanner/docs/tracing-overview)
