  - [JSON representation](#SCHEMA_REPRESENTATION)

Partial results from a streaming read or SQL query. Streaming reads and SQL queries better tolerate large result sets, large rows, and large values, but are a little trickier to consume.

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
  &quot;values&quot;: [
    value
  ],
  &quot;chunkedValue&quot;: boolean,
  &quot;resumeToken&quot;: string,
  &quot;stats&quot;: {
    object (ResultSetStats)
  },
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  },
  &quot;last&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  metadata  `

`  object ( ResultSetMetadata  ` )

Metadata about the result set, such as row type information. Only present in the first response.

`  values[]  `

`  value ( Value  ` format)

A streamed result set consists of a stream of values, which might be split into many `  PartialResultSet  ` messages to accommodate large rows and/or large values. Every N complete values defines a row, where N is equal to the number of entries in `  metadata.row_type.fields  ` .

Most values are encoded based on type as described `  here  ` .

It's possible that the last value in values is "chunked", meaning that the rest of the value is sent in subsequent `  PartialResultSet  ` (s). This is denoted by the `  chunkedValue  ` field. Two or more chunked values can be merged to form a complete value as follows:

  - `  bool/number/null  ` : can't be chunked
  - `  string  ` : concatenate the strings
  - `  list  ` : concatenate the lists. If the last element in a list is a `  string  ` , `  list  ` , or `  object  ` , merge it with the first element in the next list by applying these rules recursively.
  - `  object  ` : concatenate the (field name, field value) pairs. If a field name is duplicated, then apply these rules recursively to merge the field values.

Some examples of merging:

``` text
Strings are concatenated.
"foo", "bar" => "foobar"

Lists of non-strings are concatenated.
[2, 3], [4] => [2, 3, 4]

Lists are concatenated, but the last and first elements are merged
because they are strings.
["a", "b"], ["c", "d"] => ["a", "bc", "d"]

Lists are concatenated, but the last and first elements are merged
because they are lists. Recursively, the last and first elements
of the inner lists are merged because they are strings.
["a", ["b", "c"]], [["d"], "e"] => ["a", ["b", "cd"], "e"]

Non-overlapping object fields are combined.
{"a": "1"}, {"b": "2"} => {"a": "1", "b": 2"}

Overlapping object fields are merged.
{"a": "1"}, {"a": "2"} => {"a": "12"}

Examples of merging objects containing lists of strings.
{"a": ["1"]}, {"a": ["2"]} => {"a": ["12"]}
```

For a more complete example, suppose a streaming SQL query is yielding a result set whose rows contain a single string field. The following `  PartialResultSet  ` s might be yielded:

``` text
{
  "metadata": { ... }
  "values": ["Hello", "W"]
  "chunkedValue": true
  "resumeToken": "Af65..."
}
{
  "values": ["orl"]
  "chunkedValue": true
}
{
  "values": ["d"]
  "resumeToken": "Zx1B..."
}
```

This sequence of `  PartialResultSet  ` s encodes two rows, one containing the field value `  "Hello"  ` , and a second containing the field value `  "World" = "W" + "orl" + "d"  ` .

Not all `  PartialResultSet  ` s contain a `  resumeToken  ` . Execution can only be resumed from a previously yielded `  resumeToken  ` . For the above sequence of `  PartialResultSet  ` s, resuming the query with `  "resumeToken": "Af65..."  ` yields results from the `  PartialResultSet  ` with value "orl".

`  chunkedValue  `

`  boolean  `

If true, then the final value in `  values  ` is chunked, and must be combined with more values from subsequent `  PartialResultSet  ` s to obtain a complete field value.

`  resumeToken  `

`  string ( bytes format)  `

Streaming calls might be interrupted for a variety of reasons, such as TCP connection loss. If this occurs, the stream of results can be resumed by re-sending the original request and including `  resumeToken  ` . Note that executing any other transaction in the same session invalidates the token.

A base64-encoded string.

`  stats  `

`  object ( ResultSetStats  ` )

Query plan and execution statistics for the statement that produced this streaming result set. These can be requested by setting `  ExecuteSqlRequest.query_mode  ` and are sent only once with the last response in the stream. This field is also present in the last response for DML statements.

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

Optional. A precommit token is included if the read-write transaction has multiplexed sessions enabled. Pass the precommit token with the highest sequence number from this transaction attempt to the `  Commit  ` request for this transaction.

`  last  `

`  boolean  `

Optional. Indicates whether this is the last `  PartialResultSet  ` in the stream. The server might optionally set this field. Clients shouldn't rely on this field being set in all cases.
