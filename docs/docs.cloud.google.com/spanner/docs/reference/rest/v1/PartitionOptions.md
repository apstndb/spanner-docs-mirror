  - [JSON representation](#SCHEMA_REPRESENTATION)

Options for a `  PartitionQueryRequest  ` and `  PartitionReadRequest  ` .

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
  &quot;partitionSizeBytes&quot;: string,
  &quot;maxPartitions&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  partitionSizeBytes  `

`  string ( int64 format)  `

**Note:** This hint is currently ignored by `  sessions.partitionQuery  ` and `  sessions.partitionRead  ` requests.

The desired data size for each partition generated. The default for this option is currently 1 GiB. This is only a hint. The actual size of each partition can be smaller or larger than this size request.

`  maxPartitions  `

`  string ( int64 format)  `

**Note:** This hint is currently ignored by `  sessions.partitionQuery  ` and `  sessions.partitionRead  ` requests.

The desired maximum number of partitions to return. For example, this might be set to the number of workers available. The default for this option is currently 10,000. The maximum value is currently 200,000. This is only a hint. The actual number of partitions returned can be smaller or larger than this maximum count request.
