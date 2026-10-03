---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet
title: PartialResultSet
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet#SCHEMA_REPRESENTATION)

Partial results from a streaming read or SQL query. Streaming reads and SQL queries better tolerate large result sets, large rows, and large values, but are a little trickier to consume.

**JSON representation**

```
{
  "metadata": {
    object (ResultSetMetadata)
  },
  "values": [
    value
  ],
  "chunkedValue": boolean,
  "resumeToken": string,
  "stats": {
    object (ResultSetStats)
  },
  "precommitToken": {
    object (MultiplexedSessionPrecommitToken)
  },
  "last": boolean
}
```

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
<td><code>metadata</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata"><code>ResultSetMetadata</code></a><code> )</code></p>
<p>Metadata about the result set, such as row type information. Only present in the first response.</p></td>
</tr>
<tr class="even">
<td><code>values[]</code></td>
<td><p><code>value ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#value"><code>Value</code></a><code> format)</code></p>
<p>A streamed result set consists of a stream of values, which might be split into many <code>PartialResultSet</code> messages to accommodate large rows and/or large values. Every N complete values defines a row, where N is equal to the number of entries in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType#FIELDS.fields"><code>metadata.row_type.fields</code></a> .</p>
<p>Most values are encoded based on type as described <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode"><code>here</code></a> .</p>
<p>It's possible that the last value in values is "chunked", meaning that the rest of the value is sent in subsequent <code>PartialResultSet</code> (s). This is denoted by the <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet#FIELDS.chunked_value"><code>chunkedValue</code></a> field. Two or more chunked values can be merged to form a complete value as follows:</p>
<ul>
<li><code>bool/number/null</code> : can't be chunked</li>
<li><code>string</code> : concatenate the strings</li>
<li><code>list</code> : concatenate the lists. If the last element in a list is a <code>string</code> , <code>list</code> , or <code>object</code> , merge it with the first element in the next list by applying these rules recursively.</li>
<li><code>object</code> : concatenate the (field name, field value) pairs. If a field name is duplicated, then apply these rules recursively to merge the field values.</li>
</ul>
<p>Some examples of merging:</p>
<pre data-fenced=""><code>Strings are concatenated.
&quot;foo&quot;, &quot;bar&quot; =&gt; &quot;foobar&quot;

Lists of non-strings are concatenated.
[2, 3], [4] =&gt; [2, 3, 4]

Lists are concatenated, but the last and first elements are merged
because they are strings.
[&quot;a&quot;, &quot;b&quot;], [&quot;c&quot;, &quot;d&quot;] =&gt; [&quot;a&quot;, &quot;bc&quot;, &quot;d&quot;]

Lists are concatenated, but the last and first elements are merged
because they are lists. Recursively, the last and first elements
of the inner lists are merged because they are strings.
[&quot;a&quot;, [&quot;b&quot;, &quot;c&quot;]], [[&quot;d&quot;], &quot;e&quot;] =&gt; [&quot;a&quot;, [&quot;b&quot;, &quot;cd&quot;], &quot;e&quot;]

Non-overlapping object fields are combined.
{&quot;a&quot;: &quot;1&quot;}, {&quot;b&quot;: &quot;2&quot;} =&gt; {&quot;a&quot;: &quot;1&quot;, &quot;b&quot;: 2&quot;}

Overlapping object fields are merged.
{&quot;a&quot;: &quot;1&quot;}, {&quot;a&quot;: &quot;2&quot;} =&gt; {&quot;a&quot;: &quot;12&quot;}

Examples of merging objects containing lists of strings.
{&quot;a&quot;: [&quot;1&quot;]}, {&quot;a&quot;: [&quot;2&quot;]} =&gt; {&quot;a&quot;: [&quot;12&quot;]}</code></pre>
<p>For a more complete example, suppose a streaming SQL query is yielding a result set whose rows contain a single string field. The following <code>PartialResultSet</code> s might be yielded:</p>
<pre data-fenced=""><code>{
  &quot;metadata&quot;: { ... }
  &quot;values&quot;: [&quot;Hello&quot;, &quot;W&quot;]
  &quot;chunkedValue&quot;: true
  &quot;resumeToken&quot;: &quot;Af65...&quot;
}
{
  &quot;values&quot;: [&quot;orl&quot;]
  &quot;chunkedValue&quot;: true
}
{
  &quot;values&quot;: [&quot;d&quot;]
  &quot;resumeToken&quot;: &quot;Zx1B...&quot;
}</code></pre>
<p>This sequence of <code>PartialResultSet</code> s encodes two rows, one containing the field value <code>"Hello"</code> , and a second containing the field value <code>"World" = "W" + "orl" + "d"</code> .</p>
<p>Not all <code>PartialResultSet</code> s contain a <code>resumeToken</code> . Execution can only be resumed from a previously yielded <code>resumeToken</code> . For the above sequence of <code>PartialResultSet</code> s, resuming the query with <code>"resumeToken": "Af65..."</code> yields results from the <code>PartialResultSet</code> with value "orl".</p></td>
</tr>
<tr class="odd">
<td><code>chunkedValue</code></td>
<td><p><code>boolean</code></p>
<p>If true, then the final value in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet#FIELDS.values"><code>values</code></a> is chunked, and must be combined with more values from subsequent <code>PartialResultSet</code> s to obtain a complete field value.</p></td>
</tr>
<tr class="even">
<td><code>resumeToken</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>bytes</code></a><code> format)</code></p>
<p>Streaming calls might be interrupted for a variety of reasons, such as TCP connection loss. If this occurs, the stream of results can be resumed by re-sending the original request and including <code>resumeToken</code> . Note that executing any other transaction in the same session invalidates the token.</p>
<p>A base64-encoded string.</p></td>
</tr>
<tr class="odd">
<td><code>stats</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats"><code>ResultSetStats</code></a><code> )</code></p>
<p>Query plan and execution statistics for the statement that produced this streaming result set. These can be requested by setting <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.request_body.FIELDS.query_mode"><code>ExecuteSqlRequest.query_mode</code></a> and are sent only once with the last response in the stream. This field is also present in the last response for DML statements.</p></td>
</tr>
<tr class="even">
<td><code>precommitToken</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MultiplexedSessionPrecommitToken"><code>MultiplexedSessionPrecommitToken</code></a><code> )</code></p>
<p>Optional. A precommit token is included if the read-write transaction has multiplexed sessions enabled. Pass the precommit token with the highest sequence number from this transaction attempt to the <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#google.spanner.v1.Spanner.Commit"><code>Commit</code></a> request for this transaction.</p></td>
</tr>
<tr class="odd">
<td><code>last</code></td>
<td><p><code>boolean</code></p>
<p>Optional. Indicates whether this is the last <code>PartialResultSet</code> in the stream. The server might optionally set this field. Clients shouldn't rely on this field being set in all cases.</p></td>
</tr>
</tbody>
</table>
