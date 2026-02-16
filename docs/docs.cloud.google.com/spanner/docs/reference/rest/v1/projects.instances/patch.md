  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
          - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION.instance.SCHEMA_REPRESENTATION)
          - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION.instance.SCHEMA_REPRESENTATION_1)
          - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION.instance.SCHEMA_REPRESENTATION_2)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Updates an instance, and begins allocating or releasing resources as requested. The returned long-running operation can be used to track the progress of updating the instance. If the named instance does not exist, returns `  NOT_FOUND  ` .

Immediately upon completion of this request:

  - For resource types for which a decrease in the instance's allocation has been requested, billing is based on the newly-requested level.

Until completion of the returned operation:

  - Cancelling the operation sets its metadata's `  cancelTime  ` , and begins restoring resources to their pre-request values. The operation is guaranteed to succeed at undoing all resource changes, after which point it terminates with a `  CANCELLED  ` status.
  - All other attempts to modify the instance are rejected.
  - Reading the instance via the API continues to give the pre-request resource levels.

Upon completion of the returned operation:

  - Billing begins for all successfully-allocated resources (some types may have lower than the requested levels).
  - All newly-reserved resources are available for serving the instance's tables.
  - The instance's new resource levels are readable via the API.

The returned long-running operation will have a name of the format `  <instance_name>/operations/<operationId>  ` and can be used to track the instance modification. The metadata field type is `  UpdateInstanceMetadata  ` . The response field type is `  Instance  ` , if successful.

Authorization requires `  spanner.instances.update  ` permission on the resource `  name  ` .

### HTTP request

Choose a location:

  

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  instance.name  `

`  string  `

Required. A unique identifier for the instance, which cannot be changed after the instance is created. Values are of the form `  projects/<project>/instances/[a-z][-a-z0-9]*[a-z0-9]  ` . The final segment of the name must be between 2 and 64 characters in length.

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
  &quot;instance&quot;: {
    &quot;name&quot;: string,
    &quot;config&quot;: string,
    &quot;displayName&quot;: string,
    &quot;nodeCount&quot;: integer,
    &quot;processingUnits&quot;: integer,
    &quot;replicaComputeCapacity&quot;: [
      {
        &quot;replicaSelection&quot;: {
          object (ReplicaSelection)
        },

        // Union field compute_capacity can be only one of the following:
        &quot;nodeCount&quot;: integer,
        &quot;processingUnits&quot;: integer
        // End of list of possible types for union field compute_capacity.
      }
    ],
    &quot;autoscalingConfig&quot;: {
      &quot;autoscalingLimits&quot;: {
        object (AutoscalingLimits)
      },
      &quot;autoscalingTargets&quot;: {
        object (AutoscalingTargets)
      },
      &quot;asymmetricAutoscalingOptions&quot;: [
        {
          object (AsymmetricAutoscalingOption)
        }
      ]
    },
    &quot;state&quot;: enum (State),
    &quot;labels&quot;: {
      string: string,
      ...
    },
    &quot;instanceType&quot;: enum (InstanceType),
    &quot;endpointUris&quot;: [
      string
    ],
    &quot;createTime&quot;: string,
    &quot;updateTime&quot;: string,
    &quot;freeInstanceMetadata&quot;: {
      &quot;expireTime&quot;: string,
      &quot;upgradeTime&quot;: string,
      &quot;expireBehavior&quot;: enum (ExpireBehavior)
    },
    &quot;edition&quot;: enum (Edition),
    &quot;defaultBackupScheduleType&quot;: enum (DefaultBackupScheduleType)
  },
  &quot;fieldMask&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instance.config  `

`  string  `

Required. The name of the instance's configuration. Values are of the form `  projects/<project>/instanceConfigs/<configuration>  ` . See also `  InstanceConfig  ` and `  instanceConfigs.list  ` .

`  instance.displayName  `

`  string  `

Required. The descriptive name for this instance as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.

`  instance.nodeCount  `

`  integer  `

The number of nodes allocated to this instance. At most, one of either `  nodeCount  ` or `  processingUnits  ` should be present in the message.

Users can set the `  nodeCount  ` field to specify the target number of nodes allocated to the instance.

If autoscaling is enabled, `  nodeCount  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of nodes allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying node count across replicas (achieved by setting `  asymmetricAutoscalingOptions  ` in the autoscaling configuration), the `  nodeCount  ` set here is the maximum node count across all replicas.

For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  instance.processingUnits  `

`  integer  `

The number of processing units allocated to this instance. At most, one of either `  processingUnits  ` or `  nodeCount  ` should be present in the message.

Users can set the `  processingUnits  ` field to specify the target number of processing units allocated to the instance.

If autoscaling is enabled, `  processingUnits  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of processing units allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying processing units per replica (achieved by setting `  asymmetricAutoscalingOptions  ` in the autoscaling configuration), the `  processingUnits  ` set here is the maximum processing units across all replicas.

For more information, see [Compute capacity, nodes and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  instance.replicaComputeCapacity[]  `

`  object ( ReplicaComputeCapacity  ` )

Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.

`  instance.autoscalingConfig  `

`  object ( AutoscalingConfig  ` )

Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, nodeCount and processingUnits are treated as OUTPUT\_ONLY fields and reflect the current compute capacity allocated to the instance.

`  instance.state  `

`  enum ( State  ` )

Output only. The current instance state. For `  instances.create  ` , the state must be either omitted or set to `  CREATING  ` . For `  instances.patch  ` , the state must be either omitted or set to `  READY  ` .

`  instance.labels  `

`  map (key: string, value: string)  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. And so you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

`  instance.instanceType  `

`  enum ( InstanceType  ` )

The `  InstanceType  ` of the current instance.

`  instance.endpointUris[]  `

`  string  `

Deprecated. This field is not populated.

`  instance.createTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance was created.

Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  instance.updateTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance was most recently updated.

Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  instance.freeInstanceMetadata  `

`  object ( FreeInstanceMetadata  ` )

Free instance metadata. Only populated for free instances.

`  instance.edition  `

`  enum ( Edition  ` )

Optional. The `  Edition  ` of the current instance.

`  instance.defaultBackupScheduleType  `

`  enum ( DefaultBackupScheduleType  ` )

Optional. Controls the default backup schedule behavior for new databases within the instance. By default, a backup schedule is created automatically when a new database is created in a new instance.

Note that the `  AUTOMATIC  ` value isn't permitted for free instances, as backups and backup schedules aren't supported for free instances.

In the `  instances.get  ` or `  instances.list  ` response, if the value of `  defaultBackupScheduleType  ` isn't set, or set to `  NONE  ` , Spanner doesn't create a default backup schedule for new databases in the instance.

`  fieldMask  `

`  string ( FieldMask  ` format)

Required. A mask specifying which fields in `  Instance  ` should be updated. The field mask must always be specified; this prevents any future fields in `  Instance  ` from being erased accidentally by clients that do not know about them.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
