  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Updates an instance partition, and begins allocating or releasing resources as requested. The returned long-running operation can be used to track the progress of updating the instance partition. If the named instance partition does not exist, returns `  NOT_FOUND  ` .

Immediately upon completion of this request:

  - For resource types for which a decrease in the instance partition's allocation has been requested, billing is based on the newly-requested level.

Until completion of the returned operation:

  - Cancelling the operation sets its metadata's `  cancelTime  ` , and begins restoring resources to their pre-request values. The operation is guaranteed to succeed at undoing all resource changes, after which point it terminates with a `  CANCELLED  ` status.
  - All other attempts to modify the instance partition are rejected.
  - Reading the instance partition via the API continues to give the pre-request resource levels.

Upon completion of the returned operation:

  - Billing begins for all successfully-allocated resources (some types may have lower than the requested levels).
  - All newly-reserved resources are available for serving the instance partition's tables.
  - The instance partition's new resource levels are readable via the API.

The returned long-running operation will have a name of the format `  <instance_partition_name>/operations/<operationId>  ` and can be used to track the instance partition modification. The metadata field type is `  UpdateInstancePartitionMetadata  ` . The response field type is `  InstancePartition  ` , if successful.

Authorization requires `  spanner.instancePartitions.update  ` permission on the resource `  name  ` .

### HTTP request

Choose a location:

  

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  instancePartition.name  `

`  string  `

Required. A unique identifier for the instance partition. Values are of the form `  projects/<project>/instances/<instance>/instancePartitions/[a-z][-a-z0-9]*[a-z0-9]  ` . The final segment of the name must be between 2 and 64 characters in length. An instance partition's name cannot be changed after the instance partition is created.

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
  &quot;instancePartition&quot;: {
    &quot;name&quot;: string,
    &quot;config&quot;: string,
    &quot;displayName&quot;: string,
    &quot;nodeCount&quot;: integer,
    &quot;processingUnits&quot;: integer,
    &quot;state&quot;: enum (State),
    &quot;createTime&quot;: string,
    &quot;updateTime&quot;: string,
    &quot;referencingDatabases&quot;: [
      string
    ],
    &quot;referencingBackups&quot;: [
      string
    ],
    &quot;etag&quot;: string
  },
  &quot;fieldMask&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instancePartition.config  `

`  string  `

Required. The name of the instance partition's configuration. Values are of the form `  projects/<project>/instanceConfigs/<configuration>  ` . See also `  InstanceConfig  ` and `  instanceConfigs.list  ` .

`  instancePartition.displayName  `

`  string  `

Required. The descriptive name for this instance partition as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.

`  instancePartition.state  `

`  enum ( State  ` )

Output only. The current instance partition state.

`  instancePartition.createTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance partition was created.

Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  instancePartition.updateTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance partition was most recently updated.

Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  instancePartition.referencingDatabases[]  `

`  string  `

Output only. The names of the databases that reference this instance partition. Referencing databases should share the parent instance. The existence of any referencing database prevents the instance partition from being deleted.

`  instancePartition.referencingBackups[] (deprecated)  `

`  string  `

Output only. Deprecated: This field is not populated. Output only. The names of the backups that reference this instance partition. Referencing backups should share the parent instance. The existence of any referencing backup prevents the instance partition from being deleted.

`  instancePartition.etag  `

`  string  `

Used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance partition from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance partition updates in order to avoid race conditions: An etag is returned in the response which contains instance partitions, and systems are expected to put that etag in the request to update instance partitions to ensure that their change will be applied to the same version of the instance partition. If no etag is provided in the call to update instance partition, then the existing instance partition is overwritten blindly.

`  fieldMask  `

`  string ( FieldMask  ` format)

Required. A mask specifying which fields in `  InstancePartition  ` should be updated. The field mask must always be specified; this prevents any future fields in `  InstancePartition  ` from being erased accidentally by clients that do not know about them.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .

Union field `  compute_capacity  ` . Compute capacity defines amount of server and storage resources that are available to the databases in an instance partition. At most, one of either `  node_count  ` or `  processing_units  ` should be present in the message. For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) . `  compute_capacity  ` can be only one of the following:

`  instancePartition.nodeCount  `

`  integer  `

The number of nodes allocated to this instance partition.

Users can set the `  nodeCount  ` field to specify the target number of nodes allocated to the instance partition.

This may be zero in API responses for instance partitions that are not yet in state `  READY  ` .

`  instancePartition.processingUnits  `

`  integer  `

The number of processing units allocated to this instance partition.

Users can set the `  processingUnits  ` field to specify the target number of processing units allocated to the instance partition.

This might be zero in API responses for instance partitions that are not yet in the `  READY  ` state.

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
