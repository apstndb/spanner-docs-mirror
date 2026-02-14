This document describes how to capture custom client metrics using [OpenTelemetry](https://opentelemetry.io) . Custom client metrics are available using the [Java](/java/docs/reference/google-cloud-spanner/latest/overview) and [Go](https://pkg.go.dev/cloud.google.com/go/spanner) client libraries.

Custom client-side metrics can help you find the source of latency in your system. For more information, see [Latency points in a Spanner request](/spanner/docs/latency-points) .

Spanner client libraries also provide statistics and traces using the OpenTelemetry observability framework. For more information, see [Set up trace collection using OpenTelemetry](/spanner/docs/set-up-tracing) .

OpenTelemetry is an open source observability framework and toolkit that lets you create and manage telemetry data such as traces, metrics, and logs.

## Before you begin

You need to configure the OpenTelemetry SDK with appropriate options for exporting your telemetry data. We recommend using the OpenTelemetry Protocol (OTLP) exporter.

To set up custom client-side metrics using OpenTelemetry, you need to configure the OpenTelemetry SDK and OTLP exporter:

1.  Add the necessary dependencies to your application using the following code:
    
    ### Java
    
    ``` xml
    <dependencyManagement>
      <dependencies>
        <dependency>
          <groupId>com.google.cloud</groupId>
          <artifactId>libraries-bom</artifactId>
          <version>26.32.0</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
        <dependency>
          <groupId>io.opentelemetry</groupId>
          <artifactId>opentelemetry-bom</artifactId>
          <version>1.35.0</version>
          <type>pom</type>
          <scope>import</scope>
        </dependency>
      </dependencies>
    </dependencyManagement>
    
    <dependencies>
      <dependency>
        <groupId>com.google.cloud</groupId>
        <artifactId>google-cloud-spanner</artifactId>
      </dependency>
      <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-sdk</artifactId>
      </dependency>
      <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-sdk-metrics</artifactId>
      </dependency>
      <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-sdk-trace</artifactId>
      </dependency>
      <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-otlp</artifactId>
      </dependency>
    </dependencies>
    ```
    
    ### Go
    
    ``` modula2
    go.opentelemetry.io/otel v1.34.0
    go.opentelemetry.io/otel/exporters/otlp/otlpmetric/otlpmetricgrpc v1.28.0
    go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc v1.28.0
    go.opentelemetry.io/otel/metric v1.34.0
    go.opentelemetry.io/otel/sdk v1.34.0
    go.opentelemetry.io/otel/sdk/metric v1.34.0
    ```

2.  Create an OpenTelemetry object with the OTLP exporter and inject it into Spanner using [`  SpannerOptions  `](/java/docs/reference/google-cloud-spanner/latest/com.google.cloud.spanner.SpannerOptions) :
    
    ### Java
    
    ``` java
    // Enable OpenTelemetry metrics and traces before Injecting OpenTelemetry
    SpannerOptions.enableOpenTelemetryMetrics();
    SpannerOptions.enableOpenTelemetryTraces();
    
    // Create a new meter provider
    SdkMeterProvider sdkMeterProvider = SdkMeterProvider.builder()
        // Use Otlp exporter or any other exporter of your choice.
        .registerMetricReader(
            PeriodicMetricReader.builder(OtlpGrpcMetricExporter.builder().build()).build())
        .build();
    
    // Create a new tracer provider
    SdkTracerProvider sdkTracerProvider = SdkTracerProvider.builder()
        // Use Otlp exporter or any other exporter of your choice.
        .addSpanProcessor(SimpleSpanProcessor.builder(OtlpGrpcSpanExporter
            .builder().build()).build())
            .build();
    
    // Configure OpenTelemetry object using Meter Provider and Tracer Provider
    OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
        .setMeterProvider(sdkMeterProvider)
        .setTracerProvider(sdkTracerProvider)
        .build();
    
    // Inject OpenTelemetry object via Spanner options or register as GlobalOpenTelemetry.
    SpannerOptions options = SpannerOptions.newBuilder()
        .setOpenTelemetry(openTelemetry)
        .build();
    Spanner spanner = options.getService();
    
    DatabaseClient dbClient = spanner
        .getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
    
    captureGfeMetric(dbClient);
    captureQueryStatsMetric(openTelemetry, dbClient);
    
    // Close the providers to free up the resources and export the data. */
    sdkMeterProvider.close();
    sdkTracerProvider.close();
    ```
    
    ### Go
    
    ``` go
    // Ensure that your Go runtime version is supported by the OpenTelemetry-Go compatibility policy before enabling OpenTelemetry instrumentation.
    // Refer to compatibility here https://github.com/googleapis/google-cloud-go/blob/main/debug.md#opentelemetry
    
    import (
     "context"
     "fmt"
     "io"
     "log"
     "strconv"
     "strings"
    
     "cloud.google.com/go/spanner"
     "go.opentelemetry.io/otel"
     "go.opentelemetry.io/otel/exporters/otlp/otlpmetric/otlpmetricgrpc"
     "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
     "go.opentelemetry.io/otel/metric"
     sdkmetric "go.opentelemetry.io/otel/sdk/metric"
     "go.opentelemetry.io/otel/sdk/resource"
     sdktrace "go.opentelemetry.io/otel/sdk/trace"
     semconv "go.opentelemetry.io/otel/semconv/v1.24.0"
     "google.golang.org/api/iterator"
    )
    
    func enableOpenTelemetryMetricsAndTraces(w io.Writer, db string) error {
     // db = `projects/<project>/instances/<instance-id>/database/<database-id>`
     ctx := context.Background()
    
     // Create a new resource to uniquely identify the application
     res, err := newResource()
     if err != nil {
         log.Fatal(err)
     }
    
     // Enable OpenTelemetry traces by setting environment variable GOOGLE_API_GO_EXPERIMENTAL_TELEMETRY_PLATFORM_TRACING to the case-insensitive value "opentelemetry" before loading the client library.
     // Enable OpenTelemetry metrics before injecting meter provider.
     spanner.EnableOpenTelemetryMetrics()
    
     // Create a new tracer provider
     tracerProvider, err := getOtlpTracerProvider(ctx, res)
     defer tracerProvider.ForceFlush(ctx)
     if err != nil {
         log.Fatal(err)
     }
     // Register tracer provider as global
     otel.SetTracerProvider(tracerProvider)
    
     // Create a new meter provider
     meterProvider := getOtlpMeterProvider(ctx, res)
     defer meterProvider.ForceFlush(ctx)
    
     // Inject meter provider locally via ClientConfig when creating a spanner client or set globally via setMeterProvider.
     client, err := spanner.NewClientWithConfig(ctx, db, spanner.ClientConfig{OpenTelemetryMeterProvider: meterProvider})
     if err != nil {
         return err
     }
     defer client.Close()
     return nil
    }
    
    func getOtlpMeterProvider(ctx context.Context, res *resource.Resource) *sdkmetric.MeterProvider {
     otlpExporter, err := otlpmetricgrpc.New(ctx)
     if err != nil {
         log.Fatal(err)
     }
     meterProvider := sdkmetric.NewMeterProvider(
         sdkmetric.WithResource(res),
         sdkmetric.WithReader(sdkmetric.NewPeriodicReader(otlpExporter)),
     )
     return meterProvider
    }
    
    func getOtlpTracerProvider(ctx context.Context, res *resource.Resource) (*sdktrace.TracerProvider, error) {
     traceExporter, err := otlptracegrpc.New(ctx)
     if err != nil {
         return nil, err
     }
    
     tracerProvider := sdktrace.NewTracerProvider(
         sdktrace.WithResource(res),
         sdktrace.WithBatcher(traceExporter),
         sdktrace.WithSampler(sdktrace.AlwaysSample()),
     )
    
     return tracerProvider, nil
    }
    
    func newResource() (*resource.Resource, error) {
     return resource.Merge(resource.Default(),
         resource.NewWithAttributes(semconv.SchemaURL,
             semconv.ServiceName("otlp-service"),
             semconv.ServiceVersion("0.1.0"),
         ))
    }
    ```

## Capture GFE latency

Google Front End (GFE) latency is the duration in milliseconds between when the Google network receives a remote procedure call from the client and when the GFE receives the first byte of the response.

You can capture the GFE latency using the following code:

### Java

``` java
static void captureGfeMetric(DatabaseClient dbClient) {
  // GFE_latency and other Spanner metrics are automatically collected
  // when OpenTelemetry metrics are enabled.

  try (ResultSet resultSet =
      dbClient
          .singleUse() // Execute a single read or query against Cloud Spanner.
          .executeQuery(Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"))) {
    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
    }
  }
}
```

### Go

``` go
// GFE_Latency and other Spanner metrics are automatically collected
// when OpenTelemetry metrics are enabled.
func captureGFELatencyMetric(ctx context.Context, client spanner.Client) error {
 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
 }
}
```

The code sample appends the string `  spanner/gfe_latency  ` to the metric name when it's exported to Cloud Monitoring. You can search for this metric on Monitoring using the appended string.

## Capture Cloud Spanner API request latency

Cloud Spanner API request latency is the time in seconds between the first byte of client request that the Cloud Spanner API frontend receives and the last byte of response that the Cloud Spanner API frontend sends.

This latency metric is available as part of [Monitoring metrics](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

## Capture client round-trip latency

Client round-trip latency is the duration in milliseconds between the first byte of the Cloud Spanner API request that the client sends to the database (through both the GFE and the Cloud Spanner API frontend), and the last byte of response that the client receives from the database.

The Spanner client round-trip latency metric is not supported using OpenTelemetry. You can view the operation latency client-side metric instead. For more information, see [Client-side metric descriptions](/spanner/docs/client-side-metrics-descriptions) .

You can also instrument the metric using OpenCensus with a [bridge](https://opentelemetry.io/blog/2023/sunsetting-opencensus/#how-to-migrate-to-opentelemetry) and migrate the data to OpenTelemetry.

## Capture query latency

Query latency is the duration in milliseconds to run SQL queries in the Spanner database.

You can capture query latency using the following code:

### Java

``` java
static void captureQueryStatsMetric(OpenTelemetry openTelemetry, DatabaseClient dbClient) {
  // Register query stats metric.
  // This should be done once before start recording the data.
  Meter meter = openTelemetry.getMeter("cloud.google.com/java");
  DoubleHistogram queryStatsMetricLatencies =
      meter
          .histogramBuilder("spanner/query_stats_elapsed")
          .setDescription("The execution of the query")
          .setUnit("ms")
          .build();

  // Capture query stats metric data.
  try (ResultSet resultSet = dbClient.singleUse()
      .analyzeQuery(Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"),
          QueryAnalyzeMode.PROFILE)) {

    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
    }

    String value = resultSet.getStats().getQueryStats()
        .getFieldsOrDefault("elapsed_time", Value.newBuilder().setStringValue("0 msecs").build())
        .getStringValue();
    double elapsedTime = value.contains("msecs")
        ? Double.parseDouble(value.replaceAll(" msecs", ""))
        : Double.parseDouble(value.replaceAll(" secs", "")) * 1000;
    queryStatsMetricLatencies.record(elapsedTime);
  }
}
```

### Go

``` go
func captureQueryStatsMetric(ctx context.Context, mp metric.MeterProvider, client spanner.Client) error {
 meter := mp.Meter(spanner.OtInstrumentationScope)
 // Register query stats metric with OpenTelemetry to record the data.
 // This should be done once before start recording the data.
 queryStats, err := meter.Float64Histogram(
     "spanner/query_stats_elapsed",
     metric.WithDescription("The execution of the query"),
     metric.WithUnit("ms"),
     metric.WithExplicitBucketBoundaries(0.0, 0.01, 0.05, 0.1, 0.3, 0.6, 0.8, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 8.0, 10.0, 13.0,
         16.0, 20.0, 25.0, 30.0, 40.0, 50.0, 65.0, 80.0, 100.0, 130.0, 160.0, 200.0, 250.0,
         300.0, 400.0, 500.0, 650.0, 800.0, 1000.0, 2000.0, 5000.0, 10000.0, 20000.0, 50000.0,
         100000.0),
 )
 if err != nil {
     fmt.Print(err)
 }

 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := client.Single().QueryWithStats(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         // Record query execution time with OpenTelemetry.
         elapasedTime := iter.QueryStats["elapsed_time"].(string)
         elapasedTimeMs, err := strconv.ParseFloat(strings.TrimSuffix(elapasedTime, " msecs"), 64)
         if err != nil {
             return err
         }
         queryStats.Record(ctx, elapasedTimeMs)
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
 }
}
```

The code sample appends the string `  spanner/query_stats_elapsed  ` to the metric name when it's exported to Monitoring. You can search for this metric on Monitoring using the appended string.

## View metrics in the Metrics Explorer

1.  In the Google Cloud console, go to the Metrics Explorer page.

2.  Select your project.

3.  Click **Select a metric** .

4.  Search for a latency metrics using the following strings:
    
      - `  roundtrip_latency  ` : for the client round-trip latency metric.
      - `  spanner/gfe_latency  ` : for the GFE latency metric.
      - `  spanner/query_stats_elapsed  ` : for the query latency metric.

5.  Select the metric, then click **Apply** .

For more information on grouping or aggregating your metric, see [Build queries using menus](/monitoring/charts/metrics-selector#basic-advanced-mode) .

## What's next

  - Learn more about [OpenTelemetry](https://opentelemetry.io/) .
  - Learn how to [configure the OpenTelemetry SDK](https://opentelemetry.io/docs/concepts/instrumentation/manual) .
  - Learn how to [migrate to OpenTelemetry](/stackdriver/docs/managed-prometheus/setup-managed) .
  - Learn how to use [metrics](/spanner/docs/latency-metrics) to diagnose latency.
