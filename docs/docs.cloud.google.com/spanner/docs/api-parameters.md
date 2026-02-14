The following query parameters can be used with all methods and all resources in the Cloud Spanner API.

Query parameters that apply to all Cloud Spanner API operations are documented at [System Parameters](https://cloud.google.com/apis/docs/system-parameters) .

## Performance tips

We strongly recommend using the [client libraries](/spanner/docs/reference/libraries) for workloads that involve relatively high queries per second, relatively small requests or responses, and where you are concerned about latency or throughput. The libraries have substantially lower overhead than REST or JSON because they are built on top of the [gRPC](https://grpc.io/) interfaces. There are cases where using the REST API makes sense, and you want to optimize for fewer bytes transferred or smaller JSON responses.
