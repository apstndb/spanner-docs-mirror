## Index

  - `  Operations  ` (interface)
  - `  CancelOperationRequest  ` (message)
  - `  DeleteOperationRequest  ` (message)
  - `  GetOperationRequest  ` (message)
  - `  ListOperationsRequest  ` (message)
  - `  ListOperationsResponse  ` (message)
  - `  Operation  ` (message)
  - `  WaitOperationRequest  ` (message)

## Operations

Manages long-running operations with an API service.

When an API method normally takes long time to complete, it can be designed to return `  Operation  ` to the client, and the client can use this interface to receive the real response asynchronously by polling the operation resource, or pass the operation resource to another API (such as Pub/Sub API) to receive the response. Any API service that returns long-running operations should implement the `  Operations  ` interface so developers can have a consistent client experience.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>CancelOperation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc CancelOperation(                         CancelOperationRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Starts asynchronous cancellation on a long-running operation. The server makes a best effort to cancel the operation, but success is not guaranteed. If the server doesn't support this method, it returns <code dir="ltr" translate="no">           google.rpc.Code.UNIMPLEMENTED          </code> . Clients can use <code dir="ltr" translate="no">             Operations.GetOperation           </code> or other methods to check whether the cancellation succeeded or whether the operation completed despite cancellation. On successful cancellation, the operation is not deleted; instead, it becomes an operation with an <code dir="ltr" translate="no">             Operation.error           </code> value with a <code dir="ltr" translate="no">             google.rpc.Status.code           </code> of <code dir="ltr" translate="no">           1          </code> , corresponding to <code dir="ltr" translate="no">           Code.CANCELLED          </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
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
<th>DeleteOperation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc DeleteOperation(                         DeleteOperationRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Deletes a long-running operation. This method indicates that the client is no longer interested in the operation result. It does not cancel the operation. If the server doesn't support this method, it returns <code dir="ltr" translate="no">           google.rpc.Code.UNIMPLEMENTED          </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
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
<th>GetOperation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc GetOperation(                         GetOperationRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Gets the latest state of a long-running operation. Clients can use this method to poll the operation result at intervals as recommended by the API service.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
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
<th>ListOperations</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListOperations(                         ListOperationsRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListOperationsResponse            </code> )</p>
<p>Lists operations that match the specified filter in the request. If the server doesn't support this method, it returns <code dir="ltr" translate="no">           UNIMPLEMENTED          </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
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
<th>WaitOperation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc WaitOperation(                         WaitOperationRequest            </code> ) returns ( <code dir="ltr" translate="no">              Operation            </code> )</p>
<p>Waits until the specified long-running operation is done or reaches at most a specified timeout, returning the latest state. If the operation is already done, the latest state is immediately returned. If the timeout specified is greater than the default HTTP/RPC timeout, the HTTP/RPC timeout is used. If the server does not support this method, it returns <code dir="ltr" translate="no">           google.rpc.Code.UNIMPLEMENTED          </code> . Note that this method is on a best-effort basis. It may return the latest state before the specified timeout (including immediately), meaning even an immediate response is no guarantee that the operation is done.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.admin             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

## CancelOperationRequest

The request message for `  Operations.CancelOperation  ` .

Fields

`  name  `

`  string  `

The name of the operation resource to be cancelled.

## DeleteOperationRequest

The request message for `  Operations.DeleteOperation  ` .

Fields

`  name  `

`  string  `

The name of the operation resource to be deleted.

## GetOperationRequest

The request message for `  Operations.GetOperation  ` .

Fields

`  name  `

`  string  `

The name of the operation resource.

## ListOperationsRequest

The request message for `  Operations.ListOperations  ` .

Fields

`  name  `

`  string  `

The name of the operation's parent resource.

`  filter  `

`  string  `

The standard list filter.

`  page_size  `

`  int32  `

The standard list page size.

`  page_token  `

`  string  `

The standard list page token.

`  return_partial_success  `

`  bool  `

When set to `  true  ` , operations that are reachable are returned as normal, and those that are unreachable are returned in the `  ListOperationsResponse.unreachable  ` field.

This can only be `  true  ` when reading across collections. For example, when `  parent  ` is set to `  "projects/example/locations/-"  ` .

This field is not supported by default and will result in an `  UNIMPLEMENTED  ` error if set unless explicitly documented otherwise in service or product specific documentation.

## ListOperationsResponse

The response message for `  Operations.ListOperations  ` .

Fields

`  operations[]  `

`  Operation  `

A list of operations that matches the specified filter in the request.

`  next_page_token  `

`  string  `

The standard List next-page token.

`  unreachable[]  `

`  string  `

Unordered list. Unreachable resources. Populated when the request sets `  ListOperationsRequest.return_partial_success  ` and reads across collections. For example, when attempting to list all resources across all supported locations.

## Operation

This resource represents a long-running operation that is the result of a network API call.

Fields

`  name  `

`  string  `

The server-assigned name, which is only unique within the same service that originally returns it. If you use the default HTTP mapping, the `  name  ` should be a resource name ending with `  operations/{unique_id}  ` .

`  metadata  `

`  Any  `

Service-specific metadata associated with the operation. It typically contains progress information and common metadata such as create time. Some services might not provide such metadata. Any method that returns a long-running operation should document the metadata type, if any.

`  done  `

`  bool  `

If the value is `  false  ` , it means the operation is still in progress. If `  true  ` , the operation is completed, and either `  error  ` or `  response  ` is available.

Union field `  result  ` . The operation result, which can be either an `  error  ` or a valid `  response  ` . If `  done  ` == `  false  ` , neither `  error  ` nor `  response  ` is set. If `  done  ` == `  true  ` , exactly one of `  error  ` or `  response  ` can be set. Some services might not provide the result. `  result  ` can be only one of the following:

`  error  `

`  Status  `

The error result of the operation in case of failure or cancellation.

`  response  `

`  Any  `

The normal, successful response of the operation. If the original method returns no data on success, such as `  Delete  ` , the response is `  google.protobuf.Empty  ` . If the original method is standard `  Get  ` / `  Create  ` / `  Update  ` , the response should be the resource. For other methods, the response should have the type `  XxxResponse  ` , where `  Xxx  ` is the original method name. For example, if the original method name is `  TakeSnapshot()  ` , the inferred response type is `  TakeSnapshotResponse  ` .

## WaitOperationRequest

The request message for `  Operations.WaitOperation  ` .

Fields

`  name  `

`  string  `

The name of the operation resource to wait on.

`  timeout  `

`  Duration  `

The maximum duration to wait before timing out. If left blank, the wait will be at most the time permitted by the underlying HTTP/RPC protocol. If RPC context deadline is also specified, the shorter one will be used.
