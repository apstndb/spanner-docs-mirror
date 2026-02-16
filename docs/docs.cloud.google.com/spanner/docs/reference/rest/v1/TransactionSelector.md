  - [JSON representation](#SCHEMA_REPRESENTATION)

This message is used to select the transaction in which a `  sessions.read  ` or `  ExecuteSql  ` call runs.

See `  TransactionOptions  ` for more information about transactions.

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

  // Union field selector can be only one of the following:
  &quot;singleUse&quot;: {
    object (TransactionOptions)
  },
  &quot;id&quot;: string,
  &quot;begin&quot;: {
    object (TransactionOptions)
  }
  // End of list of possible types for union field selector.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  selector  ` . If no fields are set, the default is a single use transaction with strong concurrency. `  selector  ` can be only one of the following:

`  singleUse  `

`  object ( TransactionOptions  ` )

Execute the read or SQL query in a temporary transaction. This is the most efficient way to execute a transaction that consists of a single SQL query.

`  id  `

`  string ( bytes format)  `

Execute the read or SQL query in a previously-started transaction.

A base64-encoded string.

`  begin  `

`  object ( TransactionOptions  ` )

Begin a new transaction and execute this read or SQL query in it. The transaction ID of the new transaction is returned in `  ResultSetMetadata.transaction  ` , which is a `  Transaction  ` .
