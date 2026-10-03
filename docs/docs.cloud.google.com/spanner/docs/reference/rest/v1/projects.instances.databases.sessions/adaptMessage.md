---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage
title: 'Method: projects.instances.databases.sessions.adaptMessage'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.AdaptMessageResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#try-it)

Handles a single message from the client and returns the result as a stream. The server will interpret the message frame and respond with message frames to the client.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/sessions/*}:adaptMessage`

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
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>Required. The database session in which the adapter request is processed.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.databases.adapt</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "protocol": string,
  "payload": string,
  "attachments": {
    string: string,
    ...
  }
}
```

| Fields        |                                                                                                                                                                                  |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `protocol`    | `string` Required. Identifier for the underlying wire protocol.                                                                                                                  |
| `payload`     | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Optional. Uninterpreted bytes from the underlying wire protocol. A base64-encoded string. |
| `attachments` | `map (key: string, value: string)` Optional. Opaque request state passed by the client to the server.                                                                            |

### Response body

Message sent by the adapter to the client.

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "payload": string,
  "stateUpdates": {
    string: string,
    ...
  },
  "last": boolean
}
```

| Fields         |                                                                                                                                                                                                                                                                                                                                                         |
|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `payload`      | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Optional. Uninterpreted bytes from the underlying wire protocol. A base64-encoded string.                                                                                                                                                                        |
| `stateUpdates` | `map (key: string, value: string)` Optional. Opaque state updates to be applied by the client.                                                                                                                                                                                                                                                          |
| `last`         | `boolean` Optional. Indicates whether this is the last [`AdaptMessageResponse`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage#body.AdaptMessageResponse) in the stream. This field may be optionally set by the server. Clients should not rely on this field being set in all cases. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
