  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the long-running operation used to track the progress of optimizations performed on a newly restored database. This long-running operation is automatically created by the system after the successful completion of a database restore, and cannot be cancelled.

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
  &quot;progress&quot;: {
    object (OperationProgress)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Name of the restored database being optimized.

`  progress  `

`  object ( OperationProgress  ` )

The progress of the post-restore optimizations.
