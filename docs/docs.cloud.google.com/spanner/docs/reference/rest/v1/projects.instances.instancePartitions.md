  - [Resource: InstancePartition](#InstancePartition)
      - [JSON representation](#InstancePartition.SCHEMA_REPRESENTATION)
  - [State](#State)
  - [Methods](#METHODS_SUMMARY)

## Resource: InstancePartition

An isolated set of Cloud Spanner resources that databases can define placements on.

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
  &quot;name&quot;: string,
  &quot;config&quot;: string,
  &quot;displayName&quot;: string,
  &quot;state&quot;: enum (State),
  &quot;createTime&quot;: string,
  &quot;updateTime&quot;: string,
  &quot;referencingDatabases&quot;: [
    string
  ],
  &quot;referencingBackups&quot;: [
    string
  ],
  &quot;etag&quot;: string,

  // Union field compute_capacity can be only one of the following:
  &quot;nodeCount&quot;: integer,
  &quot;processingUnits&quot;: integer
  // End of list of possible types for union field compute_capacity.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Required. A unique identifier for the instance partition. Values are of the form `  projects/<project>/instances/<instance>/instancePartitions/[a-z][-a-z0-9]*[a-z0-9]  ` . The final segment of the name must be between 2 and 64 characters in length. An instance partition's name cannot be changed after the instance partition is created.

`  config  `

`  string  `

Required. The name of the instance partition's configuration. Values are of the form `  projects/<project>/instanceConfigs/<configuration>  ` . See also `  InstanceConfig  ` and `  instanceConfigs.list  ` .

`  displayName  `

`  string  `

Required. The descriptive name for this instance partition as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.

`  state  `

`  enum ( State  ` )

Output only. The current instance partition state.

`  createTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance partition was created.

Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  updateTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance partition was most recently updated.

Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  referencingDatabases[]  `

`  string  `

Output only. The names of the databases that reference this instance partition. Referencing databases should share the parent instance. The existence of any referencing database prevents the instance partition from being deleted.

`  referencingBackups[] (deprecated)  `

`  string  `

This item is deprecated\!

Output only. Deprecated: This field is not populated. Output only. The names of the backups that reference this instance partition. Referencing backups should share the parent instance. The existence of any referencing backup prevents the instance partition from being deleted.

`  etag  `

`  string  `

Used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance partition from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance partition updates in order to avoid race conditions: An etag is returned in the response which contains instance partitions, and systems are expected to put that etag in the request to update instance partitions to ensure that their change will be applied to the same version of the instance partition. If no etag is provided in the call to update instance partition, then the existing instance partition is overwritten blindly.

Union field `  compute_capacity  ` . Compute capacity defines amount of server and storage resources that are available to the databases in an instance partition. At most, one of either `  node_count  ` or `  processing_units  ` should be present in the message. For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) . `  compute_capacity  ` can be only one of the following:

`  nodeCount  `

`  integer  `

The number of nodes allocated to this instance partition.

Users can set the `  nodeCount  ` field to specify the target number of nodes allocated to the instance partition.

This may be zero in API responses for instance partitions that are not yet in state `  READY  ` .

`  processingUnits  `

`  integer  `

The number of processing units allocated to this instance partition.

Users can set the `  processingUnits  ` field to specify the target number of processing units allocated to the instance partition.

This might be zero in API responses for instance partitions that are not yet in the `  READY  ` state.

## State

Indicates the current state of the instance partition.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The instance partition is still being created. Resources may not be available yet, and operations such as creating placements using this instance partition may not work.

`  READY  `

The instance partition is fully created and ready to do work such as creating placements and using in databases.

## Methods

### `             create           `

Creates an instance partition and begins preparing it to be used.

### `             delete           `

Deletes an existing instance partition.

### `             get           `

Gets information about a particular instance partition.

### `             list           `

Lists all instance partitions for the given instance.

### `             patch           `

Updates an instance partition, and begins allocating or releasing resources as requested.
