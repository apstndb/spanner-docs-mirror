  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
          - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION.instance_config.SCHEMA_REPRESENTATION)
          - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION.instance_config.SCHEMA_REPRESENTATION_1)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Updates an instance configuration. The returned long-running operation can be used to track the progress of updating the instance. If the named instance configuration does not exist, returns `  NOT_FOUND  ` .

Only user-managed configurations can be updated.

Immediately after the request returns:

  - The instance configuration's `  reconciling  ` field is set to true.

While the operation is pending:

  - Cancelling the operation sets its metadata's `  cancelTime  ` . The operation is guaranteed to succeed at undoing all changes, after which point it terminates with a `  CANCELLED  ` status.
  - All other attempts to modify the instance configuration are rejected.
  - Reading the instance configuration via the API continues to give the pre-request values.

Upon completion of the returned operation:

  - Creating instances using the instance configuration uses the new values.
  - The new values of the instance configuration are readable via the API.
  - The instance configuration's `  reconciling  ` field becomes false.

The returned long-running operation will have a name of the format `  <instance_config_name>/operations/<operationId>  ` and can be used to track the instance configuration modification. The metadata field type is `  UpdateInstanceConfigMetadata  ` . The response field type is `  InstanceConfig  ` , if successful.

Authorization requires `  spanner.instanceConfigs.update  ` permission on the resource `  name  ` .

### HTTP request

Choose a location:

  
`  PATCH https://spanner.googleapis.com/v1/{instanceConfig.name=projects/*/instanceConfigs/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  instanceConfig.name  `

`  string  `

A unique identifier for the instance configuration. Values are of the form `  projects/<project>/instanceConfigs/[a-z][-a-z0-9]*  ` .

User instance configuration must start with `  custom-  ` .

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
  &quot;instanceConfig&quot;: {
    &quot;name&quot;: string,
    &quot;displayName&quot;: string,
    &quot;configType&quot;: enum (Type),
    &quot;replicas&quot;: [
      {
        &quot;location&quot;: string,
        &quot;type&quot;: enum (ReplicaType),
        &quot;defaultLeaderLocation&quot;: boolean
      }
    ],
    &quot;optionalReplicas&quot;: [
      {
        &quot;location&quot;: string,
        &quot;type&quot;: enum (ReplicaType),
        &quot;defaultLeaderLocation&quot;: boolean
      }
    ],
    &quot;baseConfig&quot;: string,
    &quot;labels&quot;: {
      string: string,
      ...
    },
    &quot;etag&quot;: string,
    &quot;leaderOptions&quot;: [
      string
    ],
    &quot;reconciling&quot;: boolean,
    &quot;state&quot;: enum (State),
    &quot;freeInstanceAvailability&quot;: enum (FreeInstanceAvailability),
    &quot;quorumType&quot;: enum (QuorumType),
    &quot;storageLimitPerProcessingUnit&quot;: string
  },
  &quot;updateMask&quot;: string,
  &quot;validateOnly&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instanceConfig.displayName  `

`  string  `

The name of this instance configuration as it appears in UIs.

`  instanceConfig.configType  `

`  enum ( Type  ` )

Output only. Whether this instance configuration is a Google-managed or user-managed configuration.

`  instanceConfig.replicas[]  `

`  object ( ReplicaInfo  ` )

The geographic placement of nodes in this instance configuration and their replication properties.

To create user-managed configurations, input `  replicas  ` must include all replicas in `  replicas  ` of the `  baseConfig  ` and include one or more replicas in the `  optionalReplicas  ` of the `  baseConfig  ` .

`  instanceConfig.optionalReplicas[]  `

`  object ( ReplicaInfo  ` )

Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.

`  instanceConfig.baseConfig  `

`  string  `

Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. `  baseConfig  ` must refer to a configuration of type `  GOOGLE_MANAGED  ` in the same project as this configuration.

`  instanceConfig.labels  `

`  map (key: string, value: string)  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. Therefore, you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

`  instanceConfig.etag  `

`  string  `

etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.

`  instanceConfig.leaderOptions[]  `

`  string  `

Allowed values of the "defaultLeader" schema option for databases in instances that use this instance configuration.

`  instanceConfig.reconciling  `

`  boolean  `

Output only. If true, the instance configuration is being created or updated. If false, there are no ongoing operations for the instance configuration.

`  instanceConfig.state  `

`  enum ( State  ` )

Output only. The current instance configuration state. Applicable only for `  USER_MANAGED  ` configurations.

`  instanceConfig.freeInstanceAvailability  `

`  enum ( FreeInstanceAvailability  ` )

Output only. Describes whether free instances are available to be created in this instance configuration.

`  instanceConfig.quorumType  `

`  enum ( QuorumType  ` )

Output only. The `  QuorumType  ` of the instance configuration.

`  instanceConfig.storageLimitPerProcessingUnit  `

`  string ( int64 format)  `

Output only. The storage limit in bytes per processing unit.

`  updateMask  `

`  string ( FieldMask  ` format)

Required. A mask specifying which fields in `  InstanceConfig  ` should be updated. The field mask must always be specified; this prevents any future fields in `  InstanceConfig  ` from being erased accidentally by clients that do not know about them. Only displayName and labels can be updated.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .

`  validateOnly  `

`  boolean  `

An option to validate, but not actually execute, a request, and provide the same response.

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
