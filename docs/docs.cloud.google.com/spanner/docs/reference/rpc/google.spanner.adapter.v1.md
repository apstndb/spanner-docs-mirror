## Index

  - `  Adapter  ` (interface)
  - `  AdaptMessageRequest  ` (message)
  - `  AdaptMessageResponse  ` (message)
  - `  CreateSessionRequest  ` (message)
  - `  Session  ` (message)

## Adapter

Cloud Spanner Adapter API

The Cloud Spanner Adapter service allows native drivers of supported database dialects to interact directly with Cloud Spanner by wrapping the underlying wire protocol used by the driver in a gRPC stream.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>AdaptMessage</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc AdaptMessage(                         AdaptMessageRequest            </code> ) returns ( <code dir="ltr" translate="no">              AdaptMessageResponse            </code> )</p>
<p>Handles a single message from the client and returns the result as a stream. The server will interpret the message frame and respond with message frames to the client.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>CreateSession</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc CreateSession(                         CreateSessionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Session            </code> )</p>
<p>Creates a new session to be used for requests made by the adapter. A session identifies a specific incarnation of a database resource and is meant to be reused across many <code dir="ltr" translate="no">           AdaptMessage          </code> calls.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

## AdaptMessageRequest

Message sent by the client to the adapter.

Fields

`  name  `

`  string  `

Required. The database session in which the adapter request is processed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.databases.adapt  `

`  protocol  `

`  string  `

Required. Identifier for the underlying wire protocol.

`  payload  `

`  bytes  `

Optional. Uninterpreted bytes from the underlying wire protocol.

`  attachments  `

`  map<string, string>  `

Optional. Opaque request state passed by the client to the server.

## AdaptMessageResponse

Message sent by the adapter to the client.

Fields

`  payload  `

`  bytes  `

Optional. Uninterpreted bytes from the underlying wire protocol.

`  state_updates  `

`  map<string, string>  `

Optional. Opaque state updates to be applied by the client.

`  last  `

`  bool  `

Optional. Indicates whether this is the last `  AdaptMessageResponse  ` in the stream. This field may be optionally set by the server. Clients should not rely on this field being set in all cases.

## CreateSessionRequest

The request for \[CreateSessionRequest\]\[Adapter.CreateSessionRequest\].

Fields

`  parent  `

`  string  `

Required. The database in which the new session is created.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.sessions.create  `

`  session  `

`  Session  `

Required. The session to create.

## Session

A session in the Cloud Spanner Adapter API.

Fields

`  name  `

`  string  `

Identifier. The name of the session. This is always system-assigned.
