  - [JSON representation](#SCHEMA_REPRESENTATION)

Results from `  sessions.read  ` or `  ExecuteSql  ` .

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
  &quot;metadata&quot;: {
    object (ResultSetMetadata)
  },
  &quot;rows&quot;: [
    array
  ],
  &quot;stats&quot;: {
    object (ResultSetStats)
  },
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  metadata  `

`  object ( ResultSetMetadata  ` )

Metadata about the result set, such as row type information.

`  rows[]  `

`  array ( ListValue  ` format)

Each element in `  rows  ` is a row whose format is defined by `  metadata.row_type  ` . The ith element in each row matches the ith field in `  metadata.row_type  ` . Elements are encoded based on type as described `  here  ` .

`  stats  `

`  object ( ResultSetStats  ` )

Query plan and execution statistics for the SQL statement that produced this result set. These can be requested by setting `  ExecuteSqlRequest.query_mode  ` . DML statements always produce stats containing the number of rows modified, unless executed using the `  ExecuteSqlRequest.QueryMode.PLAN  ` `  ExecuteSqlRequest.query_mode  ` . Other fields might or might not be populated, based on the `  ExecuteSqlRequest.query_mode  ` .

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt to the `  Commit  ` request for this transaction.
