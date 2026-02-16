  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
      - [JSON representation](#body.BatchWriteResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [MutationGroup](#MutationGroup)
      - [JSON representation](#MutationGroup.SCHEMA_REPRESENTATION)
  - [Try it\!](#try-it)

Batches the supplied mutation groups in a collection of efficient transactions. All mutations in a group are committed atomically. However, mutations across groups can be committed non-atomically in an unspecified order and thus, they must be independent of each other. Partial failure is possible, that is, some groups might have been committed successfully, while some might have failed. The results of individual batches are streamed into the response as the batches are applied.

`  sessions.batchWrite  ` requests are not replay protected, meaning that each mutation group can be applied more than once. Replays of non-idempotent mutations can have undesirable effects. For example, replays of an insert mutation can produce an already exists error or if you use generated or commit timestamp-based keys, it can result in additional rows being added to the mutation's table. We recommend structuring your mutation groups to be idempotent to avoid this issue.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:batchWrite  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the batch request is to be run.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.write  `

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
  &quot;requestOptions&quot;: {
    object (RequestOptions)
  },
  &quot;mutationGroups&quot;: [
    {
      object (MutationGroup)
    }
  ],
  &quot;excludeTxnFromChangeStreams&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  requestOptions  `

`  object ( RequestOptions  ` )

Common options for this request.

`  mutationGroups[]  `

`  object ( MutationGroup  ` )

Required. The groups of mutations to be applied.

`  excludeTxnFromChangeStreams  `

`  boolean  `

Optional. If you don't set the `  excludeTxnFromChangeStreams  ` option or if it's set to `  false  ` , then any change streams monitoring columns modified by transactions will capture the updates made within that transaction.

### Response body

The result of applying a batch of mutations.

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
  &quot;indexes&quot;: [
    integer
  ],
  &quot;status&quot;: {
    object (Status)
  },
  &quot;commitTimestamp&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  indexes[]  `

`  integer  `

The mutation groups applied in this batch. The values index into the `  mutationGroups  ` field in the corresponding `  BatchWriteRequest  ` .

`  status  `

`  object ( Status  ` )

An `  OK  ` status indicates success. Any other status indicates a failure.

`  commitTimestamp  `

`  string ( Timestamp  ` format)

The commit timestamp of the transaction that applied this batch. Present if status is OK and the mutation groups were applied, absent otherwise.

For mutation groups with conditions, a status=OK and missing commitTimestamp means that the mutation groups were not applied due to the condition not being satisfied after evaluation.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

## MutationGroup

A group of mutations to be committed together. Related mutations should be placed in a group. For example, two mutations inserting rows with the same primary key prefix in both parent and child tables are related.

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
  &quot;mutations&quot;: [
    {
      object (Mutation)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  mutations[]  `

`  object ( Mutation  ` )

Required. The mutations in this group.
