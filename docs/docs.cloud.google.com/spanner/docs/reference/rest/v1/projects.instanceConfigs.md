  - [Resource: InstanceConfig](#InstanceConfig)
      - [JSON representation](#InstanceConfig.SCHEMA_REPRESENTATION)
  - [Type](#Type)
  - [ReplicaInfo](#ReplicaInfo)
      - [JSON representation](#ReplicaInfo.SCHEMA_REPRESENTATION)
  - [ReplicaType](#ReplicaType)
  - [State](#State)
  - [FreeInstanceAvailability](#FreeInstanceAvailability)
  - [QuorumType](#QuorumType)
  - [Methods](#METHODS_SUMMARY)

## Resource: InstanceConfig

A possible configuration for a Cloud Spanner instance. Configurations define the geographic placement of nodes and their replication.

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
  &quot;displayName&quot;: string,
  &quot;configType&quot;: enum (Type),
  &quot;replicas&quot;: [
    {
      object (ReplicaInfo)
    }
  ],
  &quot;optionalReplicas&quot;: [
    {
      object (ReplicaInfo)
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
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

A unique identifier for the instance configuration. Values are of the form `  projects/<project>/instanceConfigs/[a-z][-a-z0-9]*  ` .

User instance configuration must start with `  custom-  ` .

`  displayName  `

`  string  `

The name of this instance configuration as it appears in UIs.

`  configType  `

`  enum ( Type  ` )

Output only. Whether this instance configuration is a Google-managed or user-managed configuration.

`  replicas[]  `

`  object ( ReplicaInfo  ` )

The geographic placement of nodes in this instance configuration and their replication properties.

To create user-managed configurations, input `  replicas  ` must include all replicas in `  replicas  ` of the `  baseConfig  ` and include one or more replicas in the `  optionalReplicas  ` of the `  baseConfig  ` .

`  optionalReplicas[]  `

`  object ( ReplicaInfo  ` )

Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.

`  baseConfig  `

`  string  `

Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. `  baseConfig  ` must refer to a configuration of type `  GOOGLE_MANAGED  ` in the same project as this configuration.

`  labels  `

`  map (key: string, value: string)  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. Therefore, you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

`  etag  `

`  string  `

etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.

`  leaderOptions[]  `

`  string  `

Allowed values of the "defaultLeader" schema option for databases in instances that use this instance configuration.

`  reconciling  `

`  boolean  `

Output only. If true, the instance configuration is being created or updated. If false, there are no ongoing operations for the instance configuration.

`  state  `

`  enum ( State  ` )

Output only. The current instance configuration state. Applicable only for `  USER_MANAGED  ` configurations.

`  freeInstanceAvailability  `

`  enum ( FreeInstanceAvailability  ` )

Output only. Describes whether free instances are available to be created in this instance configuration.

`  quorumType  `

`  enum ( QuorumType  ` )

Output only. The `  QuorumType  ` of the instance configuration.

`  storageLimitPerProcessingUnit  `

`  string ( int64 format)  `

Output only. The storage limit in bytes per processing unit.

## Type

The type of this configuration.

Enums

`  TYPE_UNSPECIFIED  `

Unspecified.

`  GOOGLE_MANAGED  `

Google-managed configuration.

`  USER_MANAGED  `

User-managed configuration.

## ReplicaInfo

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
  &quot;location&quot;: string,
  &quot;type&quot;: enum (ReplicaType),
  &quot;defaultLeaderLocation&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  location  `

`  string  `

The location of the serving resources, e.g., "us-central1".

`  type  `

`  enum ( ReplicaType  ` )

The type of replica.

`  defaultLeaderLocation  `

`  boolean  `

If true, this location is designated as the default leader location where leader replicas are placed. See the [region types documentation](https://cloud.google.com/spanner/docs/instances#region_types) for more details.

## ReplicaType

Indicates the type of replica. See the [replica types documentation](https://cloud.google.com/spanner/docs/replication#replica_types) for more details.

Enums

`  TYPE_UNSPECIFIED  `

Not specified.

`  READ_WRITE  `

sessions.read-write replicas support both reads and writes. These replicas:

  - Maintain a full copy of your data.
  - Serve reads.
  - Can vote whether to commit a write.
  - Participate in leadership election.
  - Are eligible to become a leader.

`  READ_ONLY  `

sessions.read-only replicas only support reads (not writes). sessions.read-only replicas:

  - Maintain a full copy of your data.
  - Serve reads.
  - Do not participate in voting to commit writes.
  - Are not eligible to become a leader.

`  WITNESS  `

Witness replicas don't support reads but do participate in voting to commit writes. Witness replicas:

  - Do not maintain a full copy of data.
  - Do not serve reads.
  - Vote whether to commit writes.
  - Participate in leader election but are not eligible to become leader.

## State

Indicates the current state of the instance configuration.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The instance configuration is still being created.

`  READY  `

The instance configuration is fully created and ready to be used to create instances.

## FreeInstanceAvailability

Describes the availability for free instances to be created in an instance configuration.

Enums

`  FREE_INSTANCE_AVAILABILITY_UNSPECIFIED  `

Not specified.

`  AVAILABLE  `

Indicates that free instances are available to be created in this instance configuration.

`  UNSUPPORTED  `

Indicates that free instances are not supported in this instance configuration.

`  DISABLED  `

Indicates that free instances are currently not available to be created in this instance configuration.

`  QUOTA_EXCEEDED  `

Indicates that additional free instances cannot be created in this instance configuration because the project has reached its limit of free instances.

## QuorumType

Indicates the quorum type of this instance configuration.

Enums

`  QUORUM_TYPE_UNSPECIFIED  `

Quorum type not specified.

`  REGION  `

An instance configuration tagged with `  REGION  ` quorum type forms a write quorum in a single region.

`  DUAL_REGION  `

An instance configuration tagged with the `  DUAL_REGION  ` quorum type forms a write quorum with exactly two read-write regions in a multi-region configuration.

This instance configuration requires failover in the event of regional failures.

`  MULTI_REGION  `

An instance configuration tagged with the `  MULTI_REGION  ` quorum type forms a write quorum from replicas that are spread across more than one region in a multi-region configuration.

## Methods

### `             create           `

Creates an instance configuration and begins preparing it to be used.

### `             delete           `

Deletes the instance configuration.

### `             get           `

Gets information about a particular instance configuration.

### `             list           `

Lists the supported instance configurations for a given project.

### `             patch           `

Updates an instance configuration.
