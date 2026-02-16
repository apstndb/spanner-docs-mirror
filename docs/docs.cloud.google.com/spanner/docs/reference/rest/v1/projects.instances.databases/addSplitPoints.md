  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [SplitPoints](#SplitPoints)
      - [JSON representation](#SplitPoints.SCHEMA_REPRESENTATION)
  - [Key](#Key)
      - [JSON representation](#Key.SCHEMA_REPRESENTATION)
  - [Try it\!](#try-it)

Adds split points to specified tables and indexes of a database.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}:addSplitPoints  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  database  `

`  string  `

Required. The database on whose tables or indexes the split points are to be added. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.databases.addSplitPoints  `

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
  &quot;splitPoints&quot;: [
    {
      object (SplitPoints)
    }
  ],
  &quot;initiator&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  splitPoints[]  `

`  object ( SplitPoints  ` )

Required. The split points to add.

`  initiator  `

`  string  `

Optional. A user-supplied tag associated with the split points. For example, "initial\_data\_load", "special\_event\_1". Defaults to "CloudAddSplitPointsAPI" if not specified. The length of the tag must not exceed 50 characters, or else it is trimmed. Only valid UTF8 characters are allowed.

### Response body

If successful, the response body is empty.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

## SplitPoints

The split points of a table or an index.

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
  &quot;table&quot;: string,
  &quot;index&quot;: string,
  &quot;keys&quot;: [
    {
      object (Key)
    }
  ],
  &quot;expireTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  table  `

`  string  `

The table to split.

`  index  `

`  string  `

The index to split. If specified, the `  table  ` field must refer to the index's base table.

`  keys[]  `

`  object ( Key  ` )

Required. The list of split keys. In essence, the split boundaries.

`  expireTime  `

`  string ( Timestamp  ` format)

Optional. The expiration timestamp of the split points. A timestamp in the past means immediate expiration. The maximum value can be 30 days in the future. Defaults to 10 days in the future if not specified.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

## Key

A split key.

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
  &quot;keyParts&quot;: array
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  keyParts  `

`  array ( ListValue  ` format)

Required. The column values making up the split key.
