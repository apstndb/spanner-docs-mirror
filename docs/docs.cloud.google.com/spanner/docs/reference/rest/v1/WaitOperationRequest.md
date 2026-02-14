  - [JSON representation](#SCHEMA_REPRESENTATION)

The request message for `  Operations.WaitOperation  ` .

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
  &quot;timeout&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

The name of the operation resource to wait on.

`  timeout  `

`  string ( Duration  ` format)

The maximum duration to wait before timing out. If left blank, the wait will be at most the time permitted by the underlying HTTP/RPC protocol. If RPC context deadline is also specified, the shorter one will be used.

A duration in seconds with up to nine fractional digits, ending with ' `  s  ` '. Example: `  "3.5s"  ` .
