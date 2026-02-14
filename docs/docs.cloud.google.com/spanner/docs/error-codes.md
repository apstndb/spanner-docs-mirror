This page describes Spanner error codes and recommended actions to handle these errors. Google APIs, including Spanner, use the canonical error codes defined by [`  google.rpc.Code  `](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto) .

When a Spanner request is successful, the API returns an HTTP `  200 OK  ` status code along with the requested data in the body of the response.

When a request fails, the Spanner API returns an HTTP `  4xx  ` or `  5xx  ` status code that generically identifies the failure as well as a response that provides more specific information about the error(s) that caused the failure.

The response object contains a single field `  error  ` whose value contains the following elements:

<table>
<thead>
<tr class="header">
<th>Element</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       code      </code></td>
<td>An <a href="https://tools.ietf.org/html/rfc7231#section-6">HTTP status code</a> that generically identifies the request failure.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       message      </code></td>
<td>Specific information about the request failure.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       status      </code></td>
<td>The <a href="https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto">canonical error code</a> ( <code dir="ltr" translate="no">       google.rpc.Code      </code> ) for Google APIs. Codes that may be returned by the Spanner API are listed in <a href="#error-codes">Error codes</a> .</td>
</tr>
</tbody>
</table>

If a request made with a content type of `  application/x-protobuf  ` results in an error, it will return a serialized [`  google.rpc.Status  `](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto) message as the payload.

**Note:** The text provided in the message might change at any time so applications shouldn't depend on the actual text.

## Error codes

The recommended way to classify errors is to inspect the value of the [canonical error code](https://github.com/googleapis/googleapis/blob/master/google/rpc/code.proto) ( `  google.rpc.Code  ` ). In JSON errors, this code appears in the `  status  ` field. In `  application/x-protobuf  ` errors, it's in the `  code  ` field.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Error code</th>
<th>Description</th>
<th>Recommended action</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       ABORTED      </code></td>
<td>The operation was aborted, typically due to concurrency issue such as a sequencer check failure or <a href="/spanner/docs/transactions">transaction</a> abort. Indicates that the request conflicted with another request.</td>
<td>For a non-transactional commit:<br />
Retry the request or structure your entities to reduce contention.<br />
<br />
For requests that are part of a transactional commit:<br />
Retry the entire transaction or structure your entities to reduce contention.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ALREADY_EXISTS      </code></td>
<td>The entity that a client attempted to create already exists (for example, inserting a row with an existing primary key).</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       CANCELLED      </code></td>
<td>The operation was cancelled, typically by the caller.</td>
<td>Retry the operation.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       DEADLINE_EXCEEDED      </code></td>
<td>The deadline expired before the operation could complete.</td>
<td>Investigate if the deadline is sufficient. Use a deadline corresponding to the actual time in which a response is useful. Note that for operations that change the state of the system, an error might be returned even if the operation has completed successfully.<br />
<br />
For tips, see <a href="/spanner/docs/deadline-exceeded">Troubleshoot deadline exceeded errors</a> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       FAILED_PRECONDITION      </code></td>
<td>The operation was rejected because a precondition for the request was not met. The message field in the error response provides information about the precondition that failed. For example, reading or querying from a timestamp that has exceeded the maximum timestamp staleness.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       INTERNAL      </code></td>
<td>The server returned an error. Some invariants expected by the underlying system have been broken.</td>
<td>Don't retry unless you understand the specific circumstance and cause of the error.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       INVALID_ARGUMENT      </code></td>
<td>The client specified an invalid value. The message field in the error response provides information as to which value was invalid.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       NOT_FOUND      </code></td>
<td>Indicates that some requested entity, such as updating an entity or querying a table or column, doesn't exist.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       OUT_OF_RANGE      </code></td>
<td>The operation was attempted past the valid range.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       PERMISSION_DENIED      </code></td>
<td>The user was not authorized to make the request.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       RESOURCE_EXHAUSTED      </code></td>
<td>Some resource has been exhausted.<br />
<br />
For the administrator plane, it's possible that the project exceeded its <a href="/spanner/quotas">quota</a> or the entire file system is out of space.<br />
<br />
For the data plane, this can happen if your Spanner nodes are overloaded.<br />
<br />
For more information, also see <a href="#sessions">Session-related error codes</a> .</td>
<td>For the administrator plane, verify that you didn't exceed your Spanner or project quota. If you've exceeded a quota, request a quota increase or wait for the quota to reset before trying again. Configure your retries to use exponential backoff.<br />
<br />
For the data plane, verify that your Spanner nodes haven't exceeded their capacity. Spanner retries these errors in the client library. If all retries fail, see <a href="#flow-control-mechanism-errors">Flow control mechanism errors</a> .<br />
<br />
In general, if your application is experiencing <code dir="ltr" translate="no">       RESOURCE_EXHAUSTED      </code> errors, treat the situation like an <code dir="ltr" translate="no">       UNAVAILABLE      </code> error, and retry with exponential backoff.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       UNAUTHENTICATED      </code></td>
<td>The request doesn't have valid authentication credentials for the operation.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       UNAVAILABLE      </code></td>
<td>The server is unavailable.</td>
<td>Retry using exponential backoff. Note that it is not always safe to retry non-idempotent operations.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       UNIMPLEMENTED      </code></td>
<td>The operation is not implemented or is not supported/enabled in this service.</td>
<td>Don't retry without fixing the problem.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       UNKNOWN      </code></td>
<td>Server returned an unknown error. Errors raised by APIs that don't return enough error information may be converted to this error.</td>
<td>Check that your request is safe. Then, retry with exponential backoff.</td>
</tr>
</tbody>
</table>

### Session errors

Spanner uses [sessions](/spanner/docs/sessions) to manage interactions between your application and the database. Sessions represent a connection to the database and facilitate operations like reads and writes.

Common session-related errors that your application might encounter include:

  - [`  Session not found  `](#session-not-found)
  - [`  RESOURCE_EXHAUSTED  `](#session-resource-exhausted)

#### Session not found

The `  Session not found  ` error occurs when your application attempts to use a session that no longer exists. This can happen for several reasons.

  - Your client application might explicitly delete a session. For example, closing a database client in your code or calling the `  deleteSessions  ` API directly removes the session. If you don't use one of the Spanner client libraries, create a new session when this error occurs. Add the new session to your session pool and remove the deleted session from the pool.

  - Spanner also automatically deletes sessions under certain conditions.
    
      - It deletes a session if it remains idle for more than one hour. This can occur in data stream jobs where downstream processing takes longer than the session idle timeout. If you're using a Dataflow job, add a `  Reshuffle  ` transform operation after the Spanner read in the Dataflow pipeline. This can help decouple the Spanner read operation from the subsequent long-running processing steps.
    
      - Spanner also deletes a session if it is older than 28 days. If you're using the client library, it handles these cases automatically. If you don't use one of the Spanner client libraries, create a new session when this error occurs. Add the new session to your session pool and remove the deleted session from the pool.

  - If you use one of the [Spanner client libraries](/spanner/docs/reference/libraries) , then the library manages sessions automatically. If you encounter this error, verify that your code doesn't explicitly delete sessions, such as by closing the database client. Occasionally, this might also be caused by an issue in the client library's session management.

#### Resource exhausted

The `  RESOURCE_EXHAUSTED: No session available in the pool  ` or `  RESOURCE_EXHAUSTED: Timed out after waiting X ms for acquiring session  ` errors indicate that your application can't acquire a session from the session pool. This happens when no sessions are available to process new read or write requests.

The following table describes some reasons that might cause these errors, and corresponding recommended actions.

<table>
<thead>
<tr class="header">
<th>Reason</th>
<th>Recommended action</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>All sessions in the pool are in use.</strong> Your application might receive more concurrent requests than the configured maximum number of sessions. All sessions in the pool are occupied, with no sessions available for new requests.</td>
<td><a href="/spanner/docs/sessions#multiplexed_sessions">Enable multiplexed sessions</a> . Multiplexed sessions allow multiple transactions and reads to share a single session, which can reduce the total number of sessions required by your application. You can also increase the <code dir="ltr" translate="no">       maxSession      </code> or <code dir="ltr" translate="no">       minSession      </code> configuration in your <a href="/spanner/docs/sessions#configure_the_number_of_sessions_and_grpc_channels_in_the_pools">session pool settings</a> .</td>
</tr>
<tr class="even">
<td><strong>Requests take a long time to complete.</strong> Long-running read or write requests can occupy all available sessions for extended periods. If these requests take longer than the session acquire timeout setting, new requests can't obtain a session from the session pool.</td>
<td>Investigate why your requests take a long time to complete. Optimize your queries or application logic to reduce execution time. You can increase the <a href="/spanner/docs/custom-timeout-and-retry">session acquire timeout setting</a> . You can also <a href="/spanner/docs/sessions#multiplexed_sessions">enable multiplexed sessions</a> for eligible client libraries to improve session utilization.</td>
</tr>
<tr class="odd">
<td><strong>There are session leaks.</strong> A session leak occurs when your application checks out a session from the pool but doesn't return it after completing the request. This typically happens when iterators or result sets aren't closed properly in your code. If all sessions leak, no sessions are available for new requests.</td>
<td>Debug your application code to identify and fix the session leaks. Ensure that your code properly closes all iterators and result sets. For more information, see <a href="https://cloud.google.com/blog/products/databases/debug-and-fix-session-leaks-in-spanner">Session leak detection solutions</a> . You can also use the <a href="/spanner/docs/sessions#automatic_cleanup_of_session_leaks">automatic clean up of session leaks</a> feature to set your session pool to automatically resolve inactive transactions.</td>
</tr>
<tr class="even">
<td><strong>Session creation is slow.</strong> Session creation is an computationally expensive operation. Client libraries send <code dir="ltr" translate="no">       BatchCreateSessions      </code> APIs to create initial sessions (based on the <code dir="ltr" translate="no">       minSession      </code> configuration) and <code dir="ltr" translate="no">       CreateSessions      </code> APIs for additional sessions (up to the <code dir="ltr" translate="no">       maxSession      </code> ). If session creation takes longer than the session acquire timeout setting, new requests might time out while waiting for sessions.</td>
<td>Verify the latency of <code dir="ltr" translate="no">       BatchCreateSessions      </code> and <code dir="ltr" translate="no">       CreateSessions      </code> API calls. Slow session creation might result from resource issues on the Spanner side or a large number of concurrent session creation operations.</td>
</tr>
</tbody>
</table>

### Flow control mechanism errors

Spanner might activate its flow control mechanism to protect itself from overload under the following conditions:

  - There is high CPU usage on the Spanner node. If you suspect that your request is causing high CPU usage, then you can use the [CPU utilization metrics](/spanner/docs/cpu-utilization) to investigate the issue.
  - There might be hotspots, which increase the processing time of the request. If you suspect that your request is causing hotspots, refer to [Find hotspots in your database](/spanner/docs/find-hotspots-in-database) to investigate the issue. For more information, see [Key Visualizer](/spanner/docs/key-visualizer) .

The flow control mechanism is supported by the following client libraries:

  - [Go Client v1.65 or later](https://github.com/googleapis/google-cloud-go/releases/tag/spanner%2Fv1.65.0)
  - [Java Client v6.72 or later](https://github.com/googleapis/java-spanner/releases/tag/v6.72.0)

The overall time for the request to complete won't increase due to the use of the flow control mechanism. Without this mechanism, Spanner waits before processing the request and eventually returns a `  DEADLINE_EXCEEDED  ` error.

When the flow control mechanism is active, Spanner pushes requests back to the client to retry. If the retry consumes the entire user-provided deadline, then the client receives a `  RESOURCE_EXHAUSTED  ` error. This error is returned if Spanner estimates that the processing time of the request is too long. The error propagates flow control and Spanner retries the request to the client, instead of accumulating retries internally. This allows Spanner to avoid accumulating additional resource consumption.
