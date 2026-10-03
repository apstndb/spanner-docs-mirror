---
name: documents/docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1
uri: https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1
title: Package google.spanner.adapter.v1
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Index

- [`Adapter`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.Adapter) (interface)
- [`AdaptMessageRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.AdaptMessageRequest) (message)
- [`AdaptMessageResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.AdaptMessageResponse) (message)
- [`CreateSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.CreateSessionRequest) (message)
- [`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.Session) (message)

## Adapter

Cloud Spanner Adapter API

The Cloud Spanner Adapter service allows native drivers of supported database dialects to interact directly with Cloud Spanner by wrapping the underlying wire protocol used by the driver in a gRPC stream.

**AdaptMessage**

`rpc AdaptMessage( `[`AdaptMessageRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.AdaptMessageRequest)` ) returns ( `[`AdaptMessageResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.AdaptMessageResponse)` )`

Handles a single message from the client and returns the result as a stream. The server will interpret the message frame and respond with message frames to the client.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**CreateSession**

`rpc CreateSession( `[`CreateSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.CreateSessionRequest)` ) returns ( `[`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.Session)` )`

Creates a new session to be used for requests made by the adapter. A session identifies a specific incarnation of a database resource and is meant to be reused across many `AdaptMessage` calls.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## AdaptMessageRequest

Message sent by the client to the adapter.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
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
<tr class="even">
<td><code>protocol</code></td>
<td><p><code>string</code></p>
<p>Required. Identifier for the underlying wire protocol.</p></td>
</tr>
<tr class="odd">
<td><code>payload</code></td>
<td><p><code>bytes</code></p>
<p>Optional. Uninterpreted bytes from the underlying wire protocol.</p></td>
</tr>
<tr class="even">
<td><code>attachments</code></td>
<td><p><code>map&lt;string, string&gt;</code></p>
<p>Optional. Opaque request state passed by the client to the server.</p></td>
</tr>
</tbody>
</table>

## AdaptMessageResponse

Message sent by the adapter to the client.

| Fields          |                                                                                                                                                                                                                                                                                                                                              |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `payload`       | `bytes` Optional. Uninterpreted bytes from the underlying wire protocol.                                                                                                                                                                                                                                                                     |
| `state_updates` | `map<string, string>` Optional. Opaque state updates to be applied by the client.                                                                                                                                                                                                                                                            |
| `last`          | `bool` Optional. Indicates whether this is the last [`AdaptMessageResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.AdaptMessageResponse) in the stream. This field may be optionally set by the server. Clients should not rely on this field being set in all cases. |

## CreateSessionRequest

The request for \[CreateSessionRequest\]\[Adapter.CreateSessionRequest\].

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>parent</code></td>
<td><p><code>string</code></p>
<p>Required. The database in which the new session is created.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.sessions.create</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>session</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.adapter.v1#google.spanner.adapter.v1.Session"><code>Session</code></a></p>
<p>Required. The session to create.</p></td>
</tr>
</tbody>
</table>

## Session

A session in the Cloud Spanner Adapter API.

| Fields |                                                                               |
|--------|-------------------------------------------------------------------------------|
| `name` | `string` Identifier. The name of the session. This is always system-assigned. |
