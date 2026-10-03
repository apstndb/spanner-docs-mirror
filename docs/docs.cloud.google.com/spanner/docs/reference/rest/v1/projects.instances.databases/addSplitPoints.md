---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints
title: 'Method: projects.instances.databases.addSplitPoints'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#body.aspect)
- [SplitPoints](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#SplitPoints)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#SplitPoints.SCHEMA_REPRESENTATION)
- [Key](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#Key)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#Key.SCHEMA_REPRESENTATION)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#try-it)

Adds split points to specified tables and indexes of a database.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}:addSplitPoints`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>database</code></td>
<td><p><code>string</code></p>
<p>Required. The database on whose tables or indexes the split points are to be added. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/databases/&lt;database&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.databases.addSplitPoints</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "splitPoints": [
    {
      object (SplitPoints)
    }
  ],
  "initiator": string
}
```

| Fields          |                                                                                                                                                                                                                                                                                                          |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `splitPoints[]` | `object ( `[`SplitPoints`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#SplitPoints)` )` Required. The split points to add.                                                                                                                  |
| `initiator`     | `string` Optional. A user-supplied tag associated with the split points. For example, "initial_data_load", "special_event_1". Defaults to "CloudAddSplitPointsAPI" if not specified. The length of the tag must not exceed 50 characters, or else it is trimmed. Only valid UTF8 characters are allowed. |

### Response body

If successful, the response body is empty.

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## SplitPoints

The split points of a table or an index.

**JSON representation**

```
{
  "table": string,
  "index": string,
  "keys": [
    {
      object (Key)
    }
  ],
  "expireTime": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`      | `string` The table to split.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `index`      | `string` The index to split. If specified, the `table` field must refer to the index's base table.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `keys[]`     | `object ( `[`Key`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/addSplitPoints#Key)` )` Required. The list of split keys. In essence, the split boundaries.                                                                                                                                                                                                                                                                                                                                                                              |
| `expireTime` | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Optional. The expiration timestamp of the split points. A timestamp in the past means immediate expiration. The maximum value can be 30 days in the future. Defaults to 10 days in the future if not specified. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |

## Key

A split key.

**JSON representation**

```
{
  "keyParts": array
}
```

| Fields     |                                                                                                                                                             |
|------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `keyParts` | `array ( `[`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value)` format)` Required. The column values making up the split key. |
