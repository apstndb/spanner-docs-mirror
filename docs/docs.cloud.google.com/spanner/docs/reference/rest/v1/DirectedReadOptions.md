  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [IncludeReplicas](#IncludeReplicas)
      - [JSON representation](#IncludeReplicas.SCHEMA_REPRESENTATION)
  - [ReplicaSelection](#ReplicaSelection)
      - [JSON representation](#ReplicaSelection.SCHEMA_REPRESENTATION)
  - [Type](#Type)
  - [ExcludeReplicas](#ExcludeReplicas)
      - [JSON representation](#ExcludeReplicas.SCHEMA_REPRESENTATION)

The `  DirectedReadOptions  ` can be used to indicate which replicas or regions should be used for non-transactional reads or queries.

`  DirectedReadOptions  ` can only be specified for a read-only transaction, otherwise the API returns an `  INVALID_ARGUMENT  ` error.

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

  // Union field replicas can be only one of the following:
  &quot;includeReplicas&quot;: {
    object (IncludeReplicas)
  },
  &quot;excludeReplicas&quot;: {
    object (ExcludeReplicas)
  }
  // End of list of possible types for union field replicas.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  replicas  ` . Required. At most one of either `  include_replicas  ` or `  exclude_replicas  ` should be present in the message. `  replicas  ` can be only one of the following:

`  includeReplicas  `

`  object ( IncludeReplicas  ` )

`  Include_replicas  ` indicates the order of replicas (as they appear in this list) to process the request. If `  autoFailoverDisabled  ` is set to `  true  ` and all replicas are exhausted without finding a healthy replica, Spanner waits for a replica in the list to become available, requests might fail due to `  DEADLINE_EXCEEDED  ` errors.

`  excludeReplicas  `

`  object ( ExcludeReplicas  ` )

`  Exclude_replicas  ` indicates that specified replicas should be excluded from serving requests. Spanner doesn't route requests to the replicas in this list.

## IncludeReplicas

An `  IncludeReplicas  ` contains a repeated set of `  ReplicaSelection  ` which indicates the order in which replicas should be considered.

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
  &quot;replicaSelections&quot;: [
    {
      object (ReplicaSelection)
    }
  ],
  &quot;autoFailoverDisabled&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  replicaSelections[]  `

`  object ( ReplicaSelection  ` )

The directed read replica selector.

`  autoFailoverDisabled  `

`  boolean  `

If `  true  ` , Spanner doesn't route requests to a replica outside the \< `  includeReplicas  ` list when all of the specified replicas are unavailable or unhealthy. Default value is `  false  ` .

## ReplicaSelection

The directed read replica selector. Callers must provide one or more of the following fields for replica selection:

  - `  location  ` - The location must be one of the regions within the multi-region configuration of your database.
  - `  type  ` - The type of the replica.

Some examples of using replica\_selectors are:

  - `  location:us-east1  ` --\> The "us-east1" replica(s) of any available type is used to process the request.
  - `  type:READ_ONLY  ` --\> The "READ\_ONLY" type replica(s) in the nearest available location are used to process the request.
  - `  location:us-east1 type:READ_ONLY  ` --\> The "READ\_ONLY" type replica(s) in location "us-east1" is used to process the request.

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
  &quot;type&quot;: enum (Type)
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  location  `

`  string  `

The location or region of the serving requests, for example, "us-east1".

`  type  `

`  enum ( Type  ` )

The type of replica.

## Type

Indicates the type of replica.

Enums

`  TYPE_UNSPECIFIED  `

Not specified.

`  READ_WRITE  `

sessions.read-write replicas support both reads and writes.

`  READ_ONLY  `

sessions.read-only replicas only support reads (not writes).

## ExcludeReplicas

An ExcludeReplicas contains a repeated set of ReplicaSelection that should be excluded from serving requests.

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
  &quot;replicaSelections&quot;: [
    {
      object (ReplicaSelection)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  replicaSelections[]  `

`  object ( ReplicaSelection  ` )

The directed read replica selector.
