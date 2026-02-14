This document describes how to capture custom client-side metrics using [OpenCensus](https://opencensus.io/) . Custom client-side metrics can help find the source of latency in your system. For more information, see [Identify the latency point](/spanner/docs/latency-points) .

**Note:** [OpenCensus is deprecated](https://opentelemetry.io/blog/2023/sunsetting-opencensus/) . We recommend using OpenTelemetry to capture and visualize Spanner observability metrics. For more information, see [Capture custom metrics using OpenTelemetry](/spanner/docs/capture-custom-metrics-opentelemetry) .

Spanner client libraries also provide statistics and traces using the OpenCensus observability framework. By default, the framework is disabled.

You need to be familiar with custom metrics associated with OpenCensus and have the OpenCensus metrics libraries and the Google Cloud Observability exporter available to your application before capturing custom metrics.

## Capture client round-trip latency

Client round-trip latency is the duration in milliseconds between the first byte of the Spanner API request that the client sends to the database and the last byte of the response that the client receives from the database. The API request can be sent through the Google Front End (GFE) or theCloud Spanner API frontend.

You can capture client round-trip latency using the following code:

### Java

``` java
static void captureGrpcMetric(DatabaseClient dbClient) {
  // Add io.grpc:grpc-census and io.opencensus:opencensus-exporter-stats-stackdriver
  //  dependencies to enable gRPC metrics.

  // Register basic gRPC views.
  RpcViews.registerClientGrpcBasicViews();

  // Enable OpenCensus exporters to export metrics to Stackdriver Monitoring.
  // Exporters use Application Default Credentials to authenticate.
  // See https://developers.google.com/identity/protocols/application-default-credentials
  // for more details.
  try {
    StackdriverStatsExporter.createAndRegister();
  } catch (IOException | IllegalStateException e) {
    System.out.println("Error during StackdriverStatsExporter");
  }

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
import (
 "context"
 "fmt"
 "io"
 "regexp"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"

 "contrib.go.opencensus.io/exporter/stackdriver"
 "go.opencensus.io/plugin/ocgrpc"
 "go.opencensus.io/stats/view"
)

var validDatabasePattern = regexp.MustCompile("^projects/(?P<project>[^/]+)/instances/(?P<instance>[^/]+)/databases/(?P<database>[^/]+)$")

func queryWithGRPCMetric(w io.Writer, db string) error {
 projectID, _, _, err := parseDatabaseName(db)
 if err != nil {
     return err
 }

 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Register OpenCensus views.
 if err := view.Register(ocgrpc.DefaultClientViews...); err != nil {
     return err
 }

 // Create OpenCensus Stackdriver exporter.
 sd, err := stackdriver.NewExporter(stackdriver.Options{
     ProjectID: projectID,
 })
 if err != nil {
     return err
 }
 // It is imperative to invoke flush before your main function exits
 defer sd.Flush()

 // Start the metrics exporter
 sd.StartMetricsExporter()
 defer sd.StopMetricsExporter()

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
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}

func parseDatabaseName(databaseUri string) (project, instance, database string, err error) {
 matches := validDatabasePattern.FindStringSubmatch(databaseUri)
 if len(matches) == 0 {
     return "", "", "", fmt.Errorf("failed to parse database name from %q according to pattern %q",
         databaseUri, validDatabasePattern.String())
 }
 return matches[1], matches[2], matches[3], nil
}
```

The code sample appends the string `  roundtrip_latency  ` to the metric name when it's exported to Cloud Monitoring. You can search for this metric in Monitoring using the appended string.

## Capture GFE latency

GFE latency is the duration in milliseconds between when the Google network receives a remote procedure call from the client and when the GFE receives the first byte of the response.

You can capture GFE latency using the following code:

### Java

``` java
static void captureGfeMetric(DatabaseClient dbClient) {
  // Capture GFE Latency.
  SpannerRpcViews.registerGfeLatencyView();

  // Capture GFE Latency and GFE Header missing count.
  // SpannerRpcViews.registerGfeLatencyAndHeaderMissingCountViews();

  // Capture only GFE Header missing count.
  // SpannerRpcViews.registerGfeHeaderMissingCountView();

  // Enable OpenCensus exporters to export metrics to Stackdriver Monitoring.
  // Exporters use Application Default Credentials to authenticate.
  // See https://developers.google.com/identity/protocols/application-default-credentials
  // for more details.
  try {
    StackdriverStatsExporter.createAndRegister();
  } catch (IOException | IllegalStateException e) {
    System.out.println("Error during StackdriverStatsExporter");
  }

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
// We are in the process of adding support in the Cloud Spanner Go Client Library
// to capture the gfe_latency metric.

import (
 "context"
 "fmt"
 "io"
 "strconv"
 "strings"

 spanner "cloud.google.com/go/spanner/apiv1"
 sppb "cloud.google.com/go/spanner/apiv1/spannerpb"
 gax "github.com/googleapis/gax-go/v2"
 "google.golang.org/grpc"
 "google.golang.org/grpc/metadata"

 "contrib.go.opencensus.io/exporter/stackdriver"
 "go.opencensus.io/stats"
 "go.opencensus.io/stats/view"
 "go.opencensus.io/tag"
)

// OpenCensus Tag, Measure and View.
var (
 KeyMethod    = tag.MustNewKey("grpc_client_method")
 GFELatencyMs = stats.Int64("cloud.google.com/go/spanner/gfe_latency",
     "Latency between Google's network receives an RPC and reads back the first byte of the response", "ms")
 GFELatencyView = view.View{
     Name:        "cloud.google.com/go/spanner/gfe_latency",
     Measure:     GFELatencyMs,
     Description: "Latency between Google's network receives an RPC and reads back the first byte of the response",
     Aggregation: view.Distribution(0.0, 0.01, 0.05, 0.1, 0.3, 0.6, 0.8, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 8.0, 10.0, 13.0,
         16.0, 20.0, 25.0, 30.0, 40.0, 50.0, 65.0, 80.0, 100.0, 130.0, 160.0, 200.0, 250.0,
         300.0, 400.0, 500.0, 650.0, 800.0, 1000.0, 2000.0, 5000.0, 10000.0, 20000.0, 50000.0,
         100000.0),
     TagKeys: []tag.Key{KeyMethod}}
)

func queryWithGFELatency(w io.Writer, db string) error {
 projectID, _, _, err := parseDatabaseName(db)
 if err != nil {
     return err
 }

 ctx := context.Background()
 client, err := spanner.NewClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 // Register OpenCensus views.
 err = view.Register(&GFELatencyView)
 if err != nil {
     return err
 }

 // Create OpenCensus Stackdriver exporter.
 sd, err := stackdriver.NewExporter(stackdriver.Options{
     ProjectID: projectID,
 })
 if err != nil {
     return err
 }
 // It is imperative to invoke flush before your main function exits
 defer sd.Flush()

 // Start the metrics exporter
 sd.StartMetricsExporter()
 defer sd.StopMetricsExporter()

 // Create a session.
 req := &sppb.CreateSessionRequest{Database: db}
 session, err := client.CreateSession(ctx, req)
 if err != nil {
     return err
 }

 // Execute a SQL query and retrieve the GFE server-timing header in gRPC metadata.
 req2 := &sppb.ExecuteSqlRequest{
     Session: session.Name,
     Sql:     `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`,
 }
 var md metadata.MD
 resultSet, err := client.ExecuteSql(ctx, req2, gax.WithGRPCOptions(grpc.Header(&md)))
 if err != nil {
     return err
 }
 for _, row := range resultSet.GetRows() {
     for _, value := range row.GetValues() {
         fmt.Fprintf(w, "%s ", value.GetStringValue())
     }
     fmt.Fprintf(w, "\n")
 }

 // The format is: "server-timing: gfet4t7; dur=[GFE latency in ms]"
 srvTiming := md.Get("server-timing")[0]
 gfeLtcy, err := strconv.Atoi(strings.TrimPrefix(srvTiming, "gfet4t7; dur="))
 if err != nil {
     return err
 }
 // Record GFE t4t7 latency with OpenCensus.
 ctx, err = tag.New(ctx, tag.Insert(KeyMethod, "ExecuteSql"))
 if err != nil {
     return err
 }
 stats.Record(ctx, GFELatencyMs.M(int64(gfeLtcy)))

 return nil
}
```

The code sample appends the string `  spanner/gfe_latency  ` to the metric name when it's exported to Monitoring. You can search for this metric in Monitoring using the appended string.

## Capture Cloud Spanner API request latency

Cloud Spanner API request latency is the time in seconds between the first byte of client request that the Cloud Spanner API frontend receives and the last byte of response that the Cloud Spanner API frontend sends.

This latency metric is available as part of [Spanner metrics](/monitoring/api/metrics_gcp_p_z#gcp-spanner) in Monitoring.

## Capture query latency

Query latency is the duration in milliseconds to run SQL queries in the Spanner database.

You can capture query latency using the following code:

### Java

``` java
private static final String MILLISECOND = "ms";
static final List<Double> RPC_MILLIS_BUCKET_BOUNDARIES =
    Collections.unmodifiableList(
        Arrays.asList(
            0.0, 0.01, 0.05, 0.1, 0.3, 0.6, 0.8, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 8.0, 10.0, 13.0,
            16.0, 20.0, 25.0, 30.0, 40.0, 50.0, 65.0, 80.0, 100.0, 130.0, 160.0, 200.0, 250.0,
            300.0, 400.0, 500.0, 650.0, 800.0, 1000.0, 2000.0, 5000.0, 10000.0, 20000.0, 50000.0,
            100000.0));
static final Aggregation AGGREGATION_WITH_MILLIS_HISTOGRAM =
    Distribution.create(BucketBoundaries.create(RPC_MILLIS_BUCKET_BOUNDARIES));

static MeasureDouble QUERY_STATS_ELAPSED =
    MeasureDouble.create(
        "cloud.google.com/java/spanner/query_stats_elapsed",
        "The execution of the query",
        MILLISECOND);

// Register the view. It is imperative that this step exists,
// otherwise recorded metrics will be dropped and never exported.
static View QUERY_STATS_LATENCY_VIEW = View
    .create(Name.create("cloud.google.com/java/spanner/query_stats_elapsed"),
        "The execution of the query",
        QUERY_STATS_ELAPSED,
        AGGREGATION_WITH_MILLIS_HISTOGRAM,
        Collections.emptyList());

static ViewManager manager = Stats.getViewManager();
private static final StatsRecorder STATS_RECORDER = Stats.getStatsRecorder();

static void captureQueryStatsMetric(DatabaseClient dbClient) {
  manager.registerView(QUERY_STATS_LATENCY_VIEW);

  // Enable OpenCensus exporters to export metrics to Cloud Monitoring.
  // Exporters use Application Default Credentials to authenticate.
  // See https://developers.google.com/identity/protocols/application-default-credentials
  // for more details.
  try {
    StackdriverStatsExporter.createAndRegister();
  } catch (IOException | IllegalStateException e) {
    System.out.println("Error during StackdriverStatsExporter");
  }

  try (ResultSet resultSet = dbClient.singleUse()
      .analyzeQuery(Statement.of("SELECT SingerId, AlbumId, AlbumTitle FROM Albums"),
          QueryAnalyzeMode.PROFILE)) {

    while (resultSet.next()) {
      System.out.printf(
          "%d %d %s", resultSet.getLong(0), resultSet.getLong(1), resultSet.getString(2));
    }
    Value value = resultSet.getStats().getQueryStats()
        .getFieldsOrDefault("elapsed_time", Value.newBuilder().setStringValue("0 msecs").build());
    double elapasedTime = Double.parseDouble(value.getStringValue().replaceAll(" msecs", ""));
    STATS_RECORDER.newMeasureMap()
        .put(QUERY_STATS_ELAPSED, elapasedTime)
        .record();
  }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "strconv"
 "strings"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"

 "contrib.go.opencensus.io/exporter/stackdriver"
 "go.opencensus.io/stats"
 "go.opencensus.io/stats/view"
 "go.opencensus.io/tag"
)

// OpenCensus Tag, Measure and View.
var (
 QueryStatsElapsed = stats.Float64("cloud.google.com/go/spanner/query_stats_elapsed",
     "The execution of the query", "ms")
 QueryStatsLatencyView = view.View{
     Name:        "cloud.google.com/go/spanner/query_stats_elapsed",
     Measure:     QueryStatsElapsed,
     Description: "The execution of the query",
     Aggregation: view.Distribution(0.0, 0.01, 0.05, 0.1, 0.3, 0.6, 0.8, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 8.0, 10.0, 13.0,
         16.0, 20.0, 25.0, 30.0, 40.0, 50.0, 65.0, 80.0, 100.0, 130.0, 160.0, 200.0, 250.0,
         300.0, 400.0, 500.0, 650.0, 800.0, 1000.0, 2000.0, 5000.0, 10000.0, 20000.0, 50000.0,
         100000.0),
     TagKeys: []tag.Key{}}
)

func queryWithQueryStats(w io.Writer, db string) error {
 projectID, _, _, err := parseDatabaseName(db)
 if err != nil {
     return err
 }

 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Register OpenCensus views.
 err = view.Register(&QueryStatsLatencyView)
 if err != nil {
     return err
 }

 // Create OpenCensus Stackdriver exporter.
 sd, err := stackdriver.NewExporter(stackdriver.Options{
     ProjectID: projectID,
 })
 if err != nil {
     return err
 }
 // It is imperative to invoke flush before your main function exits
 defer sd.Flush()

 // Start the metrics exporter
 sd.StartMetricsExporter()
 defer sd.StopMetricsExporter()

 // Execute a SQL query and get the query stats.
 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := client.Single().QueryWithStats(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         // Record query execution time with OpenCensus.
         elapasedTime := iter.QueryStats["elapsed_time"].(string)
         elapasedTimeMs, err := strconv.ParseFloat(strings.TrimSuffix(elapasedTime, " msecs"), 64)
         if err != nil {
             return err
         }
         stats.Record(ctx, QueryStatsElapsed.M(elapasedTimeMs))
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
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
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

  - Learn more about [OpenCensus](https://opencensus.io/) .

  - Learn how to use [metrics](/spanner/docs/latency-metrics) to diagnose latency.
