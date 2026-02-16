This document shows you how to set up client-side and end-to-end tracing using OpenTelemetry. You need to set up client-side tracing before you can opt in for end-to-end tracing. For more information, see [Trace collection overview](/spanner/docs/tracing-overview) .

## Before you begin

  - To ensure that the service account your application uses has the necessary permissions to set up trace collection, ask your administrator to grant the service account your application uses the [Cloud Trace Agent](/iam/docs/roles-permissions/cloudtrace#cloudtrace.agent) ( `  roles/cloudtrace.agent  ` ) IAM role on your project.
    
    **Important:** You must grant this role to the service account your application uses, *not* to your user account. Failure to grant the role to the correct principal might result in permission errors.

  - Verify that the Cloud Trace API is enabled on your project. For more information on enabling APIs, see [Enabling APIs](/apis/docs/getting-started#enabling_apis) .

## Configure client-side tracing

To configure client-side tracing, you need to export traces. You can export traces to a collector or directly to an observability backend. You can configure tracing using OpenTelemetry APIs.

### Export traces to a collector using OpenTelemetry APIs

To export traces to a collector using OpenTelemetry APIs, configure the OpenTelemetry SDK and OLTP exporter:

1.  Add the necessary dependencies to your application using the following code:
    
    ### Java
    
    ``` xml
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
      <artifactId>opentelemetry-sdk-trace</artifactId>
    </dependency>
    <dependency>
      <groupId>io.opentelemetry</groupId>
      <artifactId>opentelemetry-exporter-otlp</artifactId>
    </dependency>
    ```
    
    ### Go
    
    ``` text
    go.opentelemetry.io/otel v1.28.0
    go.opentelemetry.io/otel/sdk v1.28.0
    go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc v1.28.0
    ```
    
    ### Node.js
    
    ``` text
    "@opentelemetry/exporter-trace-otlp-grpc": "^0.57.0",
    "@opentelemetry/sdk-trace-base": "^1.26.0",
    "@opentelemetry/sdk-trace-node": "^1.26.0",
    ```
    
    ### Python
    
    ``` text
    pip install opentelemetry-api opentelemetry-sdk
    pip install opentelemetry-exporter-otlp
    ```

2.  Configure the OpenTelemetry object and enable tracing.
    
    ### Java
    
    ``` java
    Resource resource = Resource
        .getDefault().merge(Resource.builder().put("service.name", "My App").build());
    
    OtlpGrpcSpanExporter otlpGrpcSpanExporter =
        OtlpGrpcSpanExporter
            .builder()
            .setEndpoint(otlpEndpoint) // Replace with your OTLP endpoint
            .build();
    
    // Using a batch span processor
    // You can use `.setScheduleDelay()`, `.setExporterTimeout()`,
    // `.setMaxQueueSize`(), and `.setMaxExportBatchSize()` to further customize.
    BatchSpanProcessor otlpGrpcSpanProcessor =
        BatchSpanProcessor.builder(otlpGrpcSpanExporter).build();
    
    // Create a new tracer provider
    sdkTracerProvider = SdkTracerProvider.builder()
        // Use Otlp exporter or any other exporter of your choice.
        .addSpanProcessor(otlpGrpcSpanProcessor)
        .setResource(resource)
        .setSampler(Sampler.traceIdRatioBased(0.1))
        .build();
    
    // Export to a collector that is expecting OTLP using gRPC.
    OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
        .setTracerProvider(sdkTracerProvider).build();
    
    // Enable OpenTelemetry traces before Injecting OpenTelemetry
    SpannerOptions.enableOpenTelemetryTraces();
    
    // Inject OpenTelemetry object via Spanner options or register as GlobalOpenTelemetry.
    SpannerOptions options = SpannerOptions.newBuilder()
        .setOpenTelemetry(openTelemetry)
        .build();
    Spanner spanner = options.getService();
    ```
    
    ### Go
    
    ``` go
    // Ensure that your Go runtime version is supported by the OpenTelemetry-Go
    // compatibility policy before enabling OpenTelemetry instrumentation.
    
    // Enable OpenTelemetry traces by setting environment variable GOOGLE_API_GO_EXPERIMENTAL_TELEMETRY_PLATFORM_TRACING
    // to the case-insensitive value "opentelemetry" before loading the client library.
    
    ctx := context.Background()
    
    // Create a new resource to uniquely identify the application
    res, err := resource.Merge(resource.Default(),
     resource.NewWithAttributes(semconv.SchemaURL,
         semconv.ServiceName("My App"),
         semconv.ServiceVersion("0.1.0"),
     ))
    if err != nil {
     log.Fatal(err)
    }
    
    // Create a new OTLP exporter.
    defaultOtlpEndpoint := "http://localhost:4317" // Replace with the endpoint on which OTLP collector is running
    traceExporter, err := otlptracegrpc.New(ctx, otlptracegrpc.WithEndpoint(defaultOtlpEndpoint))
    if err != nil {
     log.Fatal(err)
    }
    
    // Create a new tracer provider
    tracerProvider := sdktrace.NewTracerProvider(
     sdktrace.WithResource(res),
     sdktrace.WithBatcher(traceExporter),
     sdktrace.WithSampler(sdktrace.TraceIDRatioBased(0.1)),
    )
    
    // Register tracer provider as global
    otel.SetTracerProvider(tracerProvider)
    ```
    
    ### Node.js
    
    ``` javascript
    const {NodeTracerProvider} = require('@opentelemetry/sdk-trace-node');
    const {
      OTLPTraceExporter,
    } = require('@opentelemetry/exporter-trace-otlp-grpc');
    const {
      BatchSpanProcessor,
      TraceIdRatioBasedSampler,
    } = require('@opentelemetry/sdk-trace-base');
    const {Resource} = require('@opentelemetry/resources');
    const {Spanner} = require('@google-cloud/spanner');
    
    // Define a Resource with service metadata
    const resource = new Resource({
      'service.name': 'my-service',
      'service.version': '1.0.0',
    });
    
    // Create an OTLP gRPC trace exporter
    const traceExporter = new OTLPTraceExporter({
      url: 'http://localhost:4317', // Default OTLP gRPC endpoint
    });
    
    // Create a provider with a custom sampler
    const provider = new NodeTracerProvider({
      sampler: new TraceIdRatioBasedSampler(1.0), // Sample 100% of traces
      resource,
      spanProcessors: [new BatchSpanProcessor(traceExporter)],
    });
    
    // Uncomment following line to register tracerProvider globally or pass it in Spanner object
    // provider.register();
    
    // Create the Cloud Spanner Client.
    const spanner = new Spanner({
      projectId: projectId,
      observabilityOptions: {
        tracerProvider: provider,
        enableExtendedTracing: true,
        enableEndToEndTracing: true,
      },
    });
    ```
    
    ### Python
    
    ``` python
    # Setup OpenTelemetry, trace and OTLP exporter.
    tracer_provider = TracerProvider(sampler=ALWAYS_ON)
    otlp_exporter = OTLPSpanExporter(endpoint="http://localhost:4317")
    tracer_provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
    
    # Setup the Cloud Spanner Client.
    spanner_client = spanner.Client(
        project_id,
        observability_options=dict(tracer_provider=tracer_provider, enable_extended_tracing=True, enable_end_to_end_tracing=True),
    )
    ```

### Export directly to an observability backend using OpenTelemetry APIs

To configure Spanner client libraries to directly export trace spans to Cloud Trace or another observability service provider backend, follow these steps:

1.  Add the necessary dependencies to your application using the following code:
    
    ### Java
    
    ``` text
    <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner</artifactId>
    </dependency>
    <dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-api</artifactId>
    </dependency>
    <dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk</artifactId>
    </dependency>
    <dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk-common</artifactId>
    </dependency>
    <dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-sdk-trace</artifactId>
    </dependency>
    <dependency>
    <groupId>com.google.cloud.opentelemetry</groupId>
    <artifactId>exporter-trace</artifactId>
    <version>0.30.0</version>
    </dependency>
    ```
    
    ### Go
    
    ``` text
    go.opentelemetry.io/otel v1.28.0
    go.opentelemetry.io/otel/sdk v1.28.0
    github.com/GoogleCloudPlatform/opentelemetry-operations-go/exporter/trace v1.24.1
    ```
    
    ### Node.js
    
    ``` text
    "@google-cloud/opentelemetry-cloud-trace-exporter": "^2.4.1",
    "@opentelemetry/sdk-trace-base": "^1.26.0",
    "@opentelemetry/sdk-trace-node": "^1.26.0",
    ```
    
    ### Python
    
    ``` text
    pip install opentelemetry-api opentelemetry-sdk
    pip install opentelemetry-exporter-gcp-trace
    ```

2.  Configure the OpenTelemetry object and enable tracing.
    
    ### Java
    
    ``` java
    Resource resource = Resource
        .getDefault().merge(Resource.builder().put("service.name", "My App").build());
    
    SpanExporter traceExporter = TraceExporter.createWithConfiguration(
        TraceConfiguration.builder().setProjectId(projectId).build()
    );
    
    // Using a batch span processor
    // You can use `.setScheduleDelay()`, `.setExporterTimeout()`,
    // `.setMaxQueueSize`(), and `.setMaxExportBatchSize()` to further customize.
    BatchSpanProcessor otlpGrpcSpanProcessor =
        BatchSpanProcessor.builder(traceExporter).build();
    
    // Create a new tracer provider
    sdkTracerProvider = SdkTracerProvider.builder()
        // Use Otlp exporter or any other exporter of your choice.
        .addSpanProcessor(otlpGrpcSpanProcessor)
        .setResource(resource)
        .setSampler(Sampler.traceIdRatioBased(0.1))
        .build();
    
    // Export to a collector that is expecting OTLP using gRPC.
    OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
        .setTracerProvider(sdkTracerProvider).build();
    
    // Enable OpenTelemetry traces before Injecting OpenTelemetry
    SpannerOptions.enableOpenTelemetryTraces();
    
    // Inject OpenTelemetry object via Spanner options or register it as global object.
    // To register as the global OpenTelemetry object,
    // use "OpenTelemetrySdk.builder()....buildAndRegisterGlobal()".
    SpannerOptions options = SpannerOptions.newBuilder()
        .setOpenTelemetry(openTelemetry)
        .build();
    Spanner spanner = options.getService();
    ```
    
    ### Go
    
    ``` go
    // Ensure that your Go runtime version is supported by the OpenTelemetry-Go
    // compatibility policy before enabling OpenTelemetry instrumentation.
    
    // Enable OpenTelemetry traces by setting environment variable GOOGLE_API_GO_EXPERIMENTAL_TELEMETRY_PLATFORM_TRACING
    // to the case-insensitive value "opentelemetry" before loading the client library.
    
    // Create a new resource to uniquely identify the application
    res, err := resource.Merge(resource.Default(),
     resource.NewWithAttributes(semconv.SchemaURL,
         semconv.ServiceName("My App"),
         semconv.ServiceVersion("0.1.0"),
     ))
    if err != nil {
     log.Fatal(err)
    }
    
    // Create a new cloud trace exporter
    exporter, err := traceExporter.New(traceExporter.WithProjectID(projectID))
    if err != nil {
     log.Fatal(err)
    }
    
    // Create a new tracer provider
    tracerProvider := sdktrace.NewTracerProvider(
     sdktrace.WithResource(res),
     sdktrace.WithBatcher(exporter),
     sdktrace.WithSampler(sdktrace.TraceIDRatioBased(0.1)),
    )
    
    // Register tracer provider as global
    otel.SetTracerProvider(tracerProvider)
    ```
    
    ### Node.js
    
    ``` javascript
    const {NodeTracerProvider} = require('@opentelemetry/sdk-trace-node');
    const {
      TraceExporter,
    } = require('@google-cloud/opentelemetry-cloud-trace-exporter');
    const {
      BatchSpanProcessor,
      TraceIdRatioBasedSampler,
    } = require('@opentelemetry/sdk-trace-base');
    const {Spanner} = require('@google-cloud/spanner');
    
    const traceExporter = new TraceExporter({projectId: projectId});
    
    // Create a provider with a custom sampler
    const provider = new NodeTracerProvider({
      sampler: new TraceIdRatioBasedSampler(1.0), // Sample 100% of traces
      spanProcessors: [new BatchSpanProcessor(traceExporter)],
    });
    
    // Uncomment following line to register tracerProvider globally or pass it in Spanner object
    // provider.register();
    
    // Set global propagator to propogate the trace context for end to end tracing.
    const {propagation} = require('@opentelemetry/api');
    const {W3CTraceContextPropagator} = require('@opentelemetry/core');
    propagation.setGlobalPropagator(new W3CTraceContextPropagator());
    
    const spanner = new Spanner({
      projectId: projectId,
      observabilityOptions: {
        tracerProvider: provider,
        // Enable extended tracing to allow your SQL statements to be annotated.
        enableExtendedTracing: true,
        // Enable end to end tracing.
        enableEndToEndTracing: true,
      },
    });
    ```
    
    ### Python
    
    ``` python
    # Setup OpenTelemetry, trace and Cloud Trace exporter.
    tracer_provider = TracerProvider(sampler=ALWAYS_ON)
    trace_exporter = CloudTraceSpanExporter(project_id=project_id)
    tracer_provider.add_span_processor(BatchSpanProcessor(trace_exporter))
    
    # Setup the Cloud Spanner Client.
    spanner_client = spanner.Client(
        project_id,
        observability_options=dict(tracer_provider=tracer_provider, enable_extended_tracing=True, enable_end_to_end_tracing=True),
    )
    ```

## Configure end-to-end tracing

This section provides instructions for configuring end-to-end tracing on Spanner client libraries:

1.  Add the necessary dependencies to your application using the following code:
    
    ### Java
    
    The existing client-side tracing dependencies are sufficient for configuring end-to-end tracing. You don't need any additional dependencies.
    
    ### Go
    
    In addition to the dependencies you need for client-side tracing, you also need the following dependency:
    
    `  go.opentelemetry.io/otel/propagation v1.28.0  `
    
    ### Node.js
    
    The existing client-side tracing dependencies are sufficient for configuring end-to-end tracing. You don't need any additional dependencies.
    
    ### Python
    
    The existing client-side tracing dependencies are sufficient for configuring end-to-end tracing. You don't need any additional dependencies.

2.  Opt in for end-to-end tracing.
    
    ### Java
    
    ``` text
    SpannerOptions options = SpannerOptions.newBuilder()
      .setOpenTelemetry(openTelemetry)
      .setEnableEndToEndTracing(/* enableEndtoEndTracing= */ true)
      .build();
    ```
    
    ### Go
    
    Use the `  EnableEndToEndTracing  ` option in the client configuration to opt in.
    
    ``` text
    client, _ := spanner.NewClientWithConfig(ctx, "projects/test-project/instances/test-instance/databases/test-db", spanner.ClientConfig{
    SessionPoolConfig: spanner.DefaultSessionPoolConfig,
    EnableEndToEndTracing:      true,
    }, clientOptions...)
    ```
    
    ### Node.js
    
    ``` text
    const spanner = new Spanner({
    projectId: projectId,
    observabilityOptions: {
    tracerProvider: openTelemetryTracerProvider,
    enableEndToEndTracing: true,
    }
    })
    ```
    
    ### Python
    
    ``` text
    observability_options = dict(
    tracer_provider=tracer_provider,
    enable_end_to_end_tracing=True,
    )
    spanner = spanner.Client(project_id, observability_options=observability_options)
    ```

3.  Set the trace context propagation in OpenTelemetry.
    
    ### Java
    
    ``` text
    OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
      .setTracerProvider(sdkTracerProvider)
      .setPropagators(ContextPropagators.create(W3CTraceContextPropagator.getInstance()))
      .buildAndRegisterGlobal();
    ```
    
    ### Go
    
    ``` text
    // Register the TraceContext propagator globally.
    otel.SetTextMapPropagator(propagation.TraceContext{})
    ```
    
    ### Node.js
    
    ``` text
    const {propagation} = require('@opentelemetry/api');
    const {W3CTraceContextPropagator} = require('@opentelemetry/core');
    propagation.setGlobalPropagator(new W3CTraceContextPropagator());
    ```
    
    ### Python
    
    ``` text
    from opentelemetry.propagate import set_global_textmap
    from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator
    set_global_textmap(TraceContextTextMapPropagator())
    ```

### End-to-end tracing attributes

End-to-end traces can include the following information:

<table>
<thead>
<tr class="header">
<th><strong>Attribute name</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>service.name</td>
<td>The attribute value is always <code dir="ltr" translate="no">       spanner_api_frontend      </code> .</td>
</tr>
<tr class="even">
<td>cloud.region</td>
<td>The Google Cloud cloud region of the Spanner API frontend that serves your application request.</td>
</tr>
<tr class="odd">
<td>gcp.spanner.server.query.fingerprint</td>
<td>The attribute value is the query fingerprint. To debug this query further, see the <code dir="ltr" translate="no">         TEXT_FINGERPRINT       </code> column in the Query statistics tables.</td>
</tr>
<tr class="even">
<td>gcp.spanner.server.paxos.participantcount</td>
<td>The number of participants involved in the transaction. For more information, see <a href="/spanner/docs/whitepapers/life-of-reads-and-writes#multi-split_write">Life of Spanner Reads &amp; Writes</a> .</td>
</tr>
<tr class="odd">
<td>gcp.spanner.isolation_level</td>
<td>The attribute value is the isolation level of the transaction. Possible values are <code dir="ltr" translate="no">       SERIALIZABLE      </code> and <code dir="ltr" translate="no">       REPEATABLE_READ      </code> . For more information, see <a href="/spanner/docs/isolation-levels">Isolation levels overview</a> .</td>
</tr>
</tbody>
</table>

### Sample trace

An end-to-end trace lets you view the following details:

  - The latency between your application and Spanner. You can calculate network latency to see if you have any network issues.
  - The Spanner API frontend cloud region where your application requests are being served from. You can use this to check for cross-region calls between your application and Spanner.

In the following example, your application request is being served by the Spanner API frontend in the `  us-west1  ` region and the network latency is 8.542ms (55.47ms - 46.928ms).

## What's next

  - For more information about OpenTelemetry, see [OpenTelemetry documentation](https://opentelemetry.io/docs/what-is-opentelemetry/) .
