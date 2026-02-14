  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by `  instances.move  ` .

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
  &quot;targetConfig&quot;: string,
  &quot;progress&quot;: {
    object (OperationProgress)
  },
  &quot;cancelTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  targetConfig  `

`  string  `

The target instance configuration where to move the instance. Values are of the form `  projects/<project>/instanceConfigs/<config>  ` .

`  progress  `

`  object ( OperationProgress  ` )

The progress of the `  instances.move  ` operation. `  progressPercent  ` is reset when cancellation is requested.

`  cancelTime  `

`  string ( Timestamp  ` format)

The time at which this operation was cancelled.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .
