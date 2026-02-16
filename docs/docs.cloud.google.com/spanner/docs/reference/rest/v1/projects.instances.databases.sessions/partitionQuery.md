  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates a set of partition tokens that can be used to execute a query operation in parallel. Each of the returned partition tokens can be used by `  sessions.executeStreamingSql  ` to specify a subset of the query result to read. The same session and read-only transaction must be used by the `  PartitionQueryRequest  ` used to create the partition tokens and the `  ExecuteSqlRequests  ` that use the partition tokens.

Partition tokens become invalid when the session used to create them is deleted, is idle for too long, begins a new transaction, or becomes too old. When any of these happen, it isn't possible to resume the query, and the whole operation must be restarted from the beginning.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:partitionQuery  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session used to create the partitions.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.partitionQuery  `

### Request body

The request body contains data with the following structure:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;transaction&quot;: {
    object (TransactionSelector)
  },
  &quot;sql&quot;: string,
  &quot;params&quot;: {
    object
  },
  &quot;paramTypes&quot;: {
    string: {
      object (Type)
    },
    ...
  },
  &quot;partitionOptions&quot;: {
    object (PartitionOptions)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  transaction  `

`  object ( TransactionSelector  ` )

sessions.read-only snapshot transactions are supported, read and write and single-use transactions are not.

`  sql  `

`  string  `

Required. The query request to generate partitions for. The request fails if the query isn't root partitionable. For a query to be root partitionable, it needs to satisfy a few conditions. For example, if the query execution plan contains a distributed union operator, then it must be the first operator in the plan. For more information about other conditions, see [sessions.read data in parallel](https://cloud.google.com/spanner/docs/reads#read_data_in_parallel) .

The query request must not contain DML commands, such as `  INSERT  ` , `  UPDATE  ` , or `  DELETE  ` . Use `  sessions.executeStreamingSql  ` with a `  PartitionedDml  ` transaction for large, partition-friendly DML operations.

`  params  `

`  object ( Struct  ` format)

Optional. Parameter names and values that bind to placeholders in the SQL string.

A parameter placeholder consists of the `  @  ` character followed by the parameter name (for example, `  @firstName  ` ). Parameter names can contain letters, numbers, and underscores.

Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:

`  "WHERE id > @msg_id AND id < @msg_id + 100"  `

It's an error to execute a SQL statement with unbound parameters.

`  paramTypes  `

`  map (key: string, value: object ( Type  ` ))

Optional. It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `  BYTES  ` and values of type `  STRING  ` both appear in `  params  ` as JSON strings.

In these cases, `  paramTypes  ` can be used to specify the exact SQL type for some or all of the SQL query parameters. See the definition of `  Type  ` for more information about SQL types.

`  partitionOptions  `

`  object ( PartitionOptions  ` )

Additional options that affect how many partitions are created.

### Response body

If successful, the response body contains an instance of `  PartitionResponse  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
