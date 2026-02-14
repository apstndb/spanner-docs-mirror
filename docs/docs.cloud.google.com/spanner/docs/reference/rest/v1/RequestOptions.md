  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [Priority](#Priority)
  - [ClientContext](#ClientContext)
      - [JSON representation](#ClientContext.SCHEMA_REPRESENTATION)

Common request options for various APIs.

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
  &quot;priority&quot;: enum (Priority),
  &quot;requestTag&quot;: string,
  &quot;transactionTag&quot;: string,
  &quot;clientContext&quot;: {
    object (ClientContext)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  priority  `

`  enum ( Priority  ` )

Priority for the request.

`  requestTag  `

`  string  `

A per-request tag which can be applied to queries or reads, used for statistics collection. Both `  requestTag  ` and `  transactionTag  ` can be specified for a read or query that belongs to a transaction. This field is ignored for requests where it's not applicable (for example, `  CommitRequest  ` ). Legal characters for `  requestTag  ` values are all printable characters (ASCII 32 - 126) and the length of a requestTag is limited to 50 characters. Values that exceed this limit are truncated. Any leading underscore (\_) characters are removed from the string.

`  transactionTag  `

`  string  `

A tag used for statistics collection about this transaction. Both `  requestTag  ` and `  transactionTag  ` can be specified for a read or query that belongs to a transaction. To enable tagging on a transaction, `  transactionTag  ` must be set to the same value for all requests belonging to the same transaction, including `  sessions.beginTransaction  ` . If this request doesn't belong to any transaction, `  transactionTag  ` is ignored. Legal characters for `  transactionTag  ` values are all printable characters (ASCII 32 - 126) and the length of a `  transactionTag  ` is limited to 50 characters. Values that exceed this limit are truncated. Any leading underscore (\_) characters are removed from the string.

`  clientContext  `

`  object ( ClientContext  ` )

Optional. Optional context that may be needed for some requests.

## Priority

The relative priority for requests. Note that priority isn't applicable for `  sessions.beginTransaction  ` .

The priority acts as a hint to the Cloud Spanner scheduler and doesn't guarantee priority or order of execution. For example:

  - Some parts of a write operation always execute at `  PRIORITY_HIGH  ` , regardless of the specified priority. This can cause you to see an increase in high priority workload even when executing a low priority request. This can also potentially cause a priority inversion where a lower priority request is fulfilled ahead of a higher priority request.
  - If a transaction contains multiple operations with different priorities, Cloud Spanner doesn't guarantee to process the higher priority operations first. There might be other constraints to satisfy, such as the order of operations.

Enums

`  PRIORITY_UNSPECIFIED  `

`  PRIORITY_UNSPECIFIED  ` is equivalent to `  PRIORITY_HIGH  ` .

`  PRIORITY_LOW  `

This specifies that the request is low priority.

`  PRIORITY_MEDIUM  `

This specifies that the request is medium priority.

`  PRIORITY_HIGH  `

This specifies that the request is high priority.

## ClientContext

Container for various pieces of client-owned context attached to a request.

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
  &quot;secureContext&quot;: {
    string: value,
    ...
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  secureContext  `

`  map (key: string, value: value ( Value  ` format))

Optional. Map of parameter name to value for this request. These values will be returned by any SECURE\_CONTEXT() calls invoked by this request (e.g., by queries against Parameterized Secure Views).
