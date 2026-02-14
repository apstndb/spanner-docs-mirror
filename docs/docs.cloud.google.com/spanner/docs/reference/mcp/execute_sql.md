## Tool: `       execute_sql      `

Execute SQL statement using a given session. \* execute\_sql tool can be used to execute DQL as well as DML statements. \* Use commit tool to commit result of a DML statement. \* DDL statements are only supported using update\_database\_schema tool.

The following sample demonstrate how to use `  curl  ` to invoke the `  execute_sql  ` MCP tool.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Curl Request</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                  
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;execute_sql&quot;,
    &quot;arguments&quot;: {
      // provide these details according to the tool&#39;s MCP specification
    }
  },
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;id&quot;: 1
}&#39;
                </code></pre></td>
</tr>
</tbody>
</table>

## Input Schema

The request for `  ExecuteSql  ` .

### ExecuteSqlRequest

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
  &quot;session&quot;: string,
  &quot;sql&quot;: string,
  &quot;seqno&quot;: string,

  // Union field transaction can be only one of the following:
  &quot;singleUseTransaction&quot;: boolean,
  &quot;readOnlyTransaction&quot;: boolean,
  &quot;readWriteTransaction&quot;: boolean,
  &quot;existingTransactionId&quot;: string
  // End of list of possible types for union field transaction.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  session  `

`  string  `

Required. The session in which the SQL query is executed. Format: `  projects/{project}/instances/{instance}/databases/{database}/sessions/{session}  `

`  sql  `

`  string  `

Required. The SQL query to execute.

`  seqno  `

`  string ( int64 format)  `

Optional. Sequence number of the request within a transaction. The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction can be aborted.

Union field `  transaction  ` . The transaction in which the SQL query is executed. If not set, a single use transaction will be used. For DML, use read\_write\_transaction or specify existing\_transaction\_id for previously created read-write transaction when multiple statements are part of transaction. `  transaction  ` can be only one of the following:

`  singleUseTransaction  `

`  boolean  `

Use a single use transaction for query execution.

`  readOnlyTransaction  `

`  boolean  `

Begin a new read-only transaction.

`  readWriteTransaction  `

`  boolean  `

Begin a new read-write transaction.

`  existingTransactionId  `

`  string ( bytes format)  `

Use an existing transaction.

A base64-encoded string.

## Output Schema

Results for `  execute_sql  ` tool

### ResultSet

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

Each element in `  rows  ` is a row whose format is defined by \[metadata.row\_type\]\[ResultSetMetadata.row\_type\]. The ith element in each row matches the ith field in \[metadata.row\_type\]\[ResultSetMetadata.row\_type\]. Elements are encoded based on type as described `  here  ` .

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt to the `  Commit  ` request for this transaction.

### ResultSetMetadata

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
  &quot;rowType&quot;: {
    object (StructType)
  },
  &quot;transaction&quot;: {
    object (Transaction)
  },
  &quot;undeclaredParameters&quot;: {
    object (StructType)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  rowType  `

`  object ( StructType  ` )

Indicates the field names and types for the rows in the result set. For example, a SQL query like `  "SELECT UserId, UserName FROM Users"  ` could return a `  row_type  ` value like:

``` text
"fields": [
  { "name": "UserId", "type": { "code": "INT64" } },
  { "name": "UserName", "type": { "code": "STRING" } },
]
```

`  transaction  `

`  object ( Transaction  ` )

If the read or SQL query began a transaction as a side-effect, the information about the new transaction is yielded here.

`  undeclaredParameters  `

`  object ( StructType  ` )

A SQL query can be parameterized. In PLAN mode, these parameters can be undeclared. This indicates the field names and types for those undeclared parameters in the SQL query. For example, a SQL query like `  "SELECT * FROM Users where UserId = @userId and UserName = @userName "  ` could return a `  undeclared_parameters  ` value like:

``` text
"fields": [
  { "name": "UserId", "type": { "code": "INT64" } },
  { "name": "UserName", "type": { "code": "STRING" } },
]
```

### StructType

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
  &quot;fields&quot;: [
    {
      object (Field)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  fields[]  `

`  object ( Field  ` )

The list of fields that make up this struct. Order is significant, because values of this struct type are represented as lists, where the order of field values matches the order of fields in the `  StructType  ` . In turn, the order of fields matches the order of columns in a read request, or the order of fields in the `  SELECT  ` clause of a query.

### Field

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
  &quot;name&quot;: string,
  &quot;type&quot;: {
    object (Type)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

The name of the field. For reads, this is the column name. For SQL queries, it is the column alias (e.g., `  "Word"  ` in the query `  "SELECT 'hello' AS Word"  ` ), or the column name (e.g., `  "ColName"  ` in the query `  "SELECT ColName FROM Table"  ` ). Some columns might have an empty name (e.g., `  "SELECT UPPER(ColName)"  ` ). Note that a query result can contain multiple fields with the same name.

`  type  `

`  object ( Type  ` )

The type of the field.

### Type

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
  &quot;code&quot;: enum (TypeCode),
  &quot;arrayElementType&quot;: {
    object (Type)
  },
  &quot;structType&quot;: {
    object (StructType)
  },
  &quot;typeAnnotation&quot;: enum (TypeAnnotationCode),
  &quot;protoTypeFqn&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  code  `

`  enum ( TypeCode  ` )

Required. The `  TypeCode  ` for this type.

`  arrayElementType  `

`  object ( Type  ` )

If `  code  ` == `  ARRAY  ` , then `  array_element_type  ` is the type of the array elements.

`  structType  `

`  object ( StructType  ` )

If `  code  ` == `  STRUCT  ` , then `  struct_type  ` provides type information for the struct's fields.

`  typeAnnotation  `

`  enum ( TypeAnnotationCode  ` )

The `  TypeAnnotationCode  ` that disambiguates SQL type that Spanner will use to represent values of this type during query processing. This is necessary for some type codes because a single `  TypeCode  ` can be mapped to different SQL types depending on the SQL dialect. `  type_annotation  ` typically is not needed to process the content of a value (it doesn't affect serialization) and clients can ignore it on the read path.

`  protoTypeFqn  `

`  string  `

If `  code  ` == `  PROTO  ` or `  code  ` == `  ENUM  ` , then `  proto_type_fqn  ` is the fully qualified name of the proto type representing the proto/enum definition.

### Transaction

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
  &quot;id&quot;: string,
  &quot;readTimestamp&quot;: string,
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  id  `

`  string ( bytes format)  `

`  id  ` may be used to identify the transaction in subsequent `  Read  ` , `  ExecuteSql  ` , `  Commit  ` , or `  Rollback  ` calls.

Single-use read-only transactions do not have IDs, because single-use transactions do not support multiple requests.

A base64-encoded string.

`  readTimestamp  `

`  string ( Timestamp  ` format)

For snapshot read-only transactions, the read timestamp chosen for the transaction. Not returned by default: see `  TransactionOptions.ReadOnly.return_read_timestamp  ` .

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

A precommit token is included in the response of a BeginTransaction request if the read-write transaction is on a multiplexed session and a mutation\_key was specified in the `  BeginTransaction  ` . The precommit token with the highest sequence number from this transaction attempt should be passed to the `  Commit  ` request for this transaction.

### Timestamp

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
  &quot;seconds&quot;: string,
  &quot;nanos&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  seconds  `

`  string ( int64 format)  `

Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).

`  nanos  `

`  integer  `

Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive.

### MultiplexedSessionPrecommitToken

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
  &quot;precommitToken&quot;: string,
  &quot;seqNum&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  precommitToken  `

`  string ( bytes format)  `

Opaque precommit token.

A base64-encoded string.

`  seqNum  `

`  integer  `

An incrementing seq number is generated on every precommit token that is returned. Clients should remember the precommit token with the highest sequence number from the current transaction attempt.

### ListValue

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
  &quot;values&quot;: [
    value
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  values[]  `

`  value ( Value  ` format)

Repeated field of dynamically typed values.

### Value

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

  // Union field kind can be only one of the following:
  &quot;nullValue&quot;: null,
  &quot;numberValue&quot;: number,
  &quot;stringValue&quot;: string,
  &quot;boolValue&quot;: boolean,
  &quot;structValue&quot;: {
    object
  },
  &quot;listValue&quot;: array
  // End of list of possible types for union field kind.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  kind  ` . The kind of value. `  kind  ` can be only one of the following:

`  nullValue  `

`  null  `

Represents a null value.

`  numberValue  `

`  number  `

Represents a double value.

`  stringValue  `

`  string  `

Represents a string value.

`  boolValue  `

`  boolean  `

Represents a boolean value.

`  structValue  `

`  object ( Struct  ` format)

Represents a structured value.

`  listValue  `

`  array ( ListValue  ` format)

Represents a repeated `  Value  ` .

### Struct

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
  &quot;fields&quot;: {
    string: value,
    ...
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  fields  `

`  map (key: string, value: value ( Value  ` format))

Unordered map of dynamically typed values.

An object containing a list of `  "key": value  ` pairs. Example: `  { "name": "wrench", "mass": "1.3kg", "count": "3" }  ` .

### FieldsEntry

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
  &quot;key&quot;: string,
  &quot;value&quot;: value
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  key  `

`  string  `

`  value  `

`  value ( Value  ` format)

### Tool Annotations

Destructive Hint: ✅ | Idempotent Hint: ❌ | Read Only Hint: ❌ | Open World Hint: ❌
