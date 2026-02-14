  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ListInstancePartitionsResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Lists all instance partitions for the given instance.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/instancePartitions  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The instance whose instance partitions should be listed. Values are of the form `  projects/<project>/instances/<instance>  ` . Use `  {instance} = '-'  ` to list instance partitions for all Instances in a project, e.g., `  projects/myproject/instances/-  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.instancePartitions.list  `

### Query parameters

Parameters

`  pageSize  `

`  integer  `

Number of instance partitions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  pageToken  ` should contain a `  nextPageToken  ` from a previous `  ListInstancePartitionsResponse  ` .

`  instancePartitionDeadline  `

`  string ( Timestamp  ` format)

Optional. Deadline used while retrieving metadata for instance partitions. Instance partitions whose metadata cannot be retrieved within this deadline will be added to `  unreachable  ` in `  ListInstancePartitionsResponse  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

### Request body

The request body must be empty.

### Response body

The response for `  instancePartitions.list  ` .

If successful, the response body contains data with the following structure:

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
  &quot;instancePartitions&quot;: [
    {
      object (InstancePartition)
    }
  ],
  &quot;nextPageToken&quot;: string,
  &quot;unreachable&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instancePartitions[]  `

`  object ( InstancePartition  ` )

The list of requested instancePartitions.

`  nextPageToken  `

`  string  `

`  nextPageToken  ` can be sent in a subsequent `  instancePartitions.list  ` call to fetch more of the matching instance partitions.

`  unreachable[]  `

`  string  `

The list of unreachable instances or instance partitions. It includes the names of instances or instance partitions whose metadata could not be retrieved within `  instancePartitionDeadline  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
