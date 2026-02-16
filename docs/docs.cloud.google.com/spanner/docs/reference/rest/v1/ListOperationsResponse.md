  - [JSON representation](#SCHEMA_REPRESENTATION)

The response message for `  Operations.ListOperations  ` .

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
  &quot;operations&quot;: [
    {
      object (Operation)
    }
  ],
  &quot;nextPageToken&quot;: string,
  &quot;unreachable&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  operations[]  `

`  object ( Operation  ` )

A list of operations that matches the specified filter in the request.

`  nextPageToken  `

`  string  `

The standard List next-page token.

`  unreachable[]  `

`  string  `

Unordered list. Unreachable resources. Populated when the request sets `  ListOperationsRequest.return_partial_success  ` and reads across collections. For example, when attempting to list all resources across all supported locations.
