  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [Partition](#Partition)
      - [JSON representation](#Partition.SCHEMA_REPRESENTATION)

The response for `  sessions.partitionQuery  ` or `  sessions.partitionRead  `

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
  &quot;partitions&quot;: [
    {
      object (Partition)
    }
  ],
  &quot;transaction&quot;: {
    object (Transaction)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  partitions[]  `

`  object ( Partition  ` )

Partitions created by this request.

`  transaction  `

`  object ( Transaction  ` )

Transaction created by this request.

## Partition

Information returned for each partition returned in a PartitionResponse.

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
  &quot;partitionToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  partitionToken  `

`  string ( bytes format)  `

This token can be passed to `  sessions.read  ` , `  sessions.streamingRead  ` , `  ExecuteSql  ` , or `  sessions.executeStreamingSql  ` requests to restrict the results to those identified by this partition token.

A base64-encoded string.
