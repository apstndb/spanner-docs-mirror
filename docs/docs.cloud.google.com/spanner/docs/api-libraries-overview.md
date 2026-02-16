## Available interfaces

You can use one of several programmatic interfaces when interacting with Spanner. These are the available interfaces, in the order that we recommend using them:

  - [Client libraries](/spanner/docs/reference/libraries) : The Spanner client libraries are available in multiple languages and are built on [gRPC](https://grpc.io/) . These client libraries provide a layer of abstraction on top of gRPC and handle the details of session management, transaction execution, retries, and more.
  - [ORM and framework drivers](/spanner/docs/drivers-overview) : Google supports open-source Spanner drivers for several popular object-relational mapping libraries (ORMs) and frameworks, such as JDBC. These drivers allow the use of Spanner databases through APIs defined by those frameworks.
  - [RPC API](/spanner/docs/reference/rpc) : If a client library or ORM driver is not available for your programming language of choice, use the RPC API, which is built on [gRPC](https://grpc.io/) . gRPC offers a number of performance benefits compared with using the REST API, including representing objects in protocol buffer format (which are faster to produce and consume compared with JSON) and persistent connections (which result in less per-request overhead). Read more about these and other benefits in [gRPC Concepts](https://grpc.io/docs/guides/concepts/) .
  - [REST API](/spanner/docs/reference/rest) : If you're unable to use Spanner's client libraries or the RPC API, use the REST API. Note that some features that are available in the RPC API are not supported in the REST API, as documented below.

## RPC versus REST API

This table compares Spanner features available through its RPC and REST API interfaces.

<table>
<thead>
<tr class="header">
<th>Feature</th>
<th>Supported in the <a href="/spanner/docs/reference/rpc">RPC API</a> ?</th>
<th>Supported in the <a href="/spanner/docs/reference/rest">REST API?</a></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Cancelling a request</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr class="even">
<td>Setting a deadline or timeout on a request</td>
<td>Yes</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Sending a streaming request</td>
<td>Yes. see <a href="/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql"><code dir="ltr" translate="no">        ExecuteStreamingSQL       </code></a> and <a href="/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.StreamingRead"><code dir="ltr" translate="no">        StreamingRead       </code></a> .</td>
<td><em></em> Partial. <a href="/apis/docs/http#streaming">HTTP Streaming</a> is supported but application-level <a href="/apis/docs/http#flow_control">flow control</a> is not.</td>
</tr>
</tbody>
</table>

## Client libraries features support

The following table lists the client libraries, noting the major Spanner features that each one supports.

Client

[Go](https://pkg.go.dev/cloud.google.com/go/spanner)

[Java](/java/docs/reference/google-cloud-spanner/latest/overview)

[Node.js](https://googleapis.dev/nodejs/spanner/latest/)

[Python](/python/docs/reference/spanner/latest)

[Ruby](https://googleapis.dev/ruby/google-cloud-spanner/latest/Google/Cloud/Spanner.html)

[C++](/cpp/docs/reference/spanner/latest)

[PHP](https://googleapis.github.io/google-cloud-php/#/docs/google-cloud/v0.189.0/spanner/readme)

[C\#](/dotnet/docs/reference/Google.Cloud.Spanner.Data/latest)

[Batch DDL](/spanner/docs/schema-updates#order_of_execution_of_statements_in_batches)

[Batch DML](/spanner/docs/samples/spanner-dml-batch-update)

[Configurable leader option](/spanner/docs/modifying-leader-region)

[Graph queries](/spanner/docs/graph/queries-overview)

[Interleaved tables](/spanner/docs/schema-and-data-model#parent-child)

[JSON type](/spanner/docs/working-with-json)

[Mutations](/spanner/docs/modify-mutation-api)

[Partitioned DML](/spanner/docs/dml-partitioned)

[Partitioned read](/spanner/docs/reads#read_data_in_parallel)

[PostgreSQL interface](/spanner/docs/postgresql-interface)

[Request priority](/spanner/docs/reference/rest/v1/RequestOptions#Priority)

[Request tagging](/spanner/docs/introspection/troubleshooting-with-tags)

[Session labeling](/spanner/docs/sessions)

[Stale reads](/spanner/docs/reads)

[Statement hints](/spanner/docs/reference/standard-sql/query-syntax#statement-hints)

[Client metrics](/spanner/docs/view-manage-client-side-metrics)
