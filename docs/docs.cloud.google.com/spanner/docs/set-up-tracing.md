This document shows you how to set up client-side and end-to-end tracing using OpenTelemetry. You need to set up client-side tracing before you can opt in for end-to-end tracing. For more information, see [Trace collection overview](https://docs.cloud.google.com/spanner/docs/tracing-overview) .

## Before you begin

  - To ensure that the service account your application uses has the necessary permissions to set up trace collection, ask your administrator to grant the [Cloud Trace Agent](https://docs.cloud.google.com/iam/docs/roles-permissions/cloudtrace#cloudtrace.agent) ( `  roles/cloudtrace.agent  ` ) IAM role to the service account your application uses on your project.
    
    **Important:** You must grant this role to the service account your application uses, *not* to your user account. Failure to grant the role to the correct principal might result in permission errors.

  - Verify that the Cloud Trace API is enabled on your project. For more information on enabling APIs, see [Enabling APIs](https://docs.cloud.google.com/apis/docs/getting-started#enabling_apis) .

## Configure client-side tracing

To configure client-side tracing, you need to export traces. You can export traces to a collector or directly to an observability backend. You can configure tracing using OpenTelemetry APIs.

### Export traces to a collector using OpenTelemetry APIs

To export traces to a collector using OpenTelemetry APIs, configure the OpenTelemetry SDK and OLTP exporter:

1.  Add the necessary dependencies to your application using the following code:
    
    ### Java
    
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
    
    ### Go
    
        go.opentelemetry.io/otel v1.28.0
        go.opentelemetry.io/otel/sdk v1.28.0
        go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc v1.28.0
    
    ### Node.js
    
        "@opentelemetry/exporter-trace-otlp-grpc": "^0.57.0",
        "@opentelemetry/sdk-trace-base": "^1.26.0",
        "@opentelemetry/sdk-trace-node": "^1.26.0",
    
    ### Python
    
        pip install opentelemetry-api opentelemetry-sdk
        pip install opentelemetry-exporter-otlp

2.  Configure the OpenTelemetry object and enable tracing.
    
    ### Java
    
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
    
    ### Go
    
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
    
    ### Node.js
    
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
    
    ### Python
    
        # Setup OpenTelemetry, trace and OTLP exporter.
        tracer_provider = TracerProvider(sampler=ALWAYS_ON)
        otlp_exporter = OTLPSpanExporter(endpoint="http://localhost:4317")
        tracer_provider.add_span_processor(BatchSpanProcessor(otlp_exporter))
        
        # Setup the Cloud Spanner Client.
        spanner_client = spanner.Client(
            project_id,
            observability_options=dict(tracer_provider=tracer_provider, enable_extended_tracing=True, enable_end_to_end_tracing=True),
        )

### Export directly to an observability backend using OpenTelemetry APIs

To configure Spanner client libraries to directly export trace spans to Cloud Trace or another observability service provider backend, follow these steps:

1.  Add the necessary dependencies to your application using the following code:
    
    ### Java
    
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
    
    ### Go
    
        go.opentelemetry.io/otel v1.28.0
        go.opentelemetry.io/otel/sdk v1.28.0
        github.com/GoogleCloudPlatform/opentelemetry-operations-go/exporter/trace v1.24.1
    
    ### Node.js
    
        "@google-cloud/opentelemetry-cloud-trace-exporter": "^2.4.1",
        "@opentelemetry/sdk-trace-base": "^1.26.0",
        "@opentelemetry/sdk-trace-node": "^1.26.0",
    
    ### Python
    
        pip install opentelemetry-api opentelemetry-sdk
        pip install opentelemetry-exporter-gcp-trace

2.  Configure the OpenTelemetry object and enable tracing.
    
    ### Java
    
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
    
    ### Go
    
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
    
    ### Node.js
    
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
    
    ### Python
    
        # Setup OpenTelemetry, trace and Cloud Trace exporter.
        tracer_provider = TracerProvider(sampler=ALWAYS_ON)
        trace_exporter = CloudTraceSpanExporter(project_id=project_id)
        tracer_provider.add_span_processor(BatchSpanProcessor(trace_exporter))
        
        # Setup the Cloud Spanner Client.
        spanner_client = spanner.Client(
            project_id,
            observability_options=dict(tracer_provider=tracer_provider, enable_extended_tracing=True, enable_end_to_end_tracing=True),
        )

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
    
        SpannerOptions options = SpannerOptions.newBuilder()
          .setOpenTelemetry(openTelemetry)
          .setEnableEndToEndTracing(/* enableEndtoEndTracing= */ true)
          .build();
    
    ### Go
    
    Use the `  EnableEndToEndTracing  ` option in the client configuration to opt in.
    
        client, _ := spanner.NewClientWithConfig(ctx, "projects/test-project/instances/test-instance/databases/test-db", spanner.ClientConfig{
        SessionPoolConfig: spanner.DefaultSessionPoolConfig,
        EnableEndToEndTracing:      true,
        }, clientOptions...)
    
    ### Node.js
    
        const spanner = new Spanner({
        projectId: projectId,
        observabilityOptions: {
        tracerProvider: openTelemetryTracerProvider,
        enableEndToEndTracing: true,
        }
        })
    
    ### Python
    
        observability_options = dict(
        tracer_provider=tracer_provider,
        enable_end_to_end_tracing=True,
        )
        spanner = spanner.Client(project_id, observability_options=observability_options)

3.  Set the trace context propagation in OpenTelemetry.
    
    ### Java
    
        OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
          .setTracerProvider(sdkTracerProvider)
          .setPropagators(ContextPropagators.create(W3CTraceContextPropagator.getInstance()))
          .buildAndRegisterGlobal();
    
    ### Go
    
        // Register the TraceContext propagator globally.
        otel.SetTextMapPropagator(propagation.TraceContext{})
    
    ### Node.js
    
        const {propagation} = require('@opentelemetry/api');
        const {W3CTraceContextPropagator} = require('@opentelemetry/core');
        propagation.setGlobalPropagator(new W3CTraceContextPropagator());
    
    ### Python
    
        from opentelemetry.propagate import set_global_textmap
        from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator
        set_global_textmap(TraceContextTextMapPropagator())

### End-to-end tracing attributes

End-to-end traces can include the following information:

| **Attribute name**                        | **Description**                                                                                                                                                                                                                                                              |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| service.name                              | The attribute value is always `        spanner_api_frontend       ` .                                                                                                                                                                                                        |
| cloud.region                              | The Google Cloud cloud region of the Spanner API frontend that serves your application request.                                                                                                                                                                              |
| gcp.spanner.server.query.fingerprint      | The attribute value is the query fingerprint. To debug this query further, see the `          TEXT_FINGERPRINT        ` column in the Query statistics tables.                                                                                                               |
| gcp.spanner.server.paxos.participantcount | The number of participants involved in the transaction. For more information, see [Life of Spanner Reads & Writes](https://docs.cloud.google.com/spanner/docs/whitepapers/life-of-reads-and-writes#multi-split_write) .                                                      |
| gcp.spanner.isolation\_level              | The attribute value is the isolation level of the transaction. Possible values are `        SERIALIZABLE       ` and `        REPEATABLE_READ       ` . For more information, see [Isolation levels overview](https://docs.cloud.google.com/spanner/docs/isolation-levels) . |

### Sample trace

An end-to-end trace lets you view the following details:

  - The latency between your application and Spanner. You can calculate network latency to see if you have any network issues.
  - The Spanner API frontend cloud region where your application requests are being served from. You can use this to check for cross-region calls between your application and Spanner.

In the following example, your application request is being served by the Spanner API frontend in the `  us-west1  ` region and the network latency is 8.542ms (55.47ms - 46.928ms).

![View a end-to-end trace.](https://docs.cloud.google.com/static/spanner/docs/images/server-side-trace.png)

## What's next

  - For more information about OpenTelemetry, see [OpenTelemetry documentation](https://opentelemetry.io/docs/what-is-opentelemetry/) .
