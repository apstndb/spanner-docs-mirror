## Tool: `       list_configs      `

List instance configs in a given project. \* Response may include next\_page\_token to fetch additional configs using list\_configs tool with page\_token set.

The following sample demonstrate how to use `  curl  ` to invoke the `  list_configs  ` MCP tool.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Curl Request</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                  
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;list_configs&quot;,
    &quot;arguments&quot;: {
      // provide these details according to the tool&#39;s MCP specification
    }
  },
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;id&quot;: 1
}&#39;
                </code></pre></td>
</tr>
</tbody>
</table>

## Input Schema

The request for `  ListInstanceConfigs  ` .

### ListInstanceConfigsRequest

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
  &quot;parent&quot;: string,
  &quot;pageSize&quot;: integer,
  &quot;pageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  parent  `

`  string  `

Required. The name of the project for which a list of supported instance configurations is requested. Values are of the form `  projects/<project>  ` .

`  pageSize  `

`  integer  `

Number of instance configurations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListInstanceConfigsResponse  ` .

## Output Schema

The response for `  ListInstanceConfigs  ` .

### ListInstanceConfigsResponse

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
  &quot;instanceConfigs&quot;: [
    {
      object (InstanceConfig)
    }
  ],
  &quot;nextPageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instanceConfigs[]  `

`  object ( InstanceConfig  ` )

The list of requested instance configurations.

`  nextPageToken  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListInstanceConfigs  ` call to fetch more of the matching instance configurations.

### InstanceConfig

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

To create user-managed configurations, input `  replicas  ` must include all replicas in `  replicas  ` of the `  base_config  ` and include one or more replicas in the `  optional_replicas  ` of the `  base_config  ` .

`  optionalReplicas[]  `

`  object ( ReplicaInfo  ` )

Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.

`  baseConfig  `

`  string  `

Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. `  base_config  ` must refer to a configuration of type `  GOOGLE_MANAGED  ` in the same project as this configuration.

`  labels  `

`  map (key: string, value: string)  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. Therefore, you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

An object containing a list of `  "key": value  ` pairs. Example: `  { "name": "wrench", "mass": "1.3kg", "count": "3" }  ` .

`  etag  `

`  string  `

etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.

`  leaderOptions[]  `

`  string  `

Allowed values of the "default\_leader" schema option for databases in instances that use this instance configuration.

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

### ReplicaInfo

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

### LabelsEntry

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
  &quot;key&quot;: string,
  &quot;value&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  key  `

`  string  `

`  value  `

`  string  `

### Tool Annotations

Destructive Hint: ❌ | Idempotent Hint: ❌ | Read Only Hint: ✅ | Open World Hint: ❌
