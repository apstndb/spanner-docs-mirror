  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [TypeCode](#TypeCode)
  - [TypeAnnotationCode](#TypeAnnotationCode)

`  Type  ` indicates the type of a Cloud Spanner value, as might be stored in a table cell or returned from an SQL query.

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

If `  code  ` == `  ARRAY  ` , then `  arrayElementType  ` is the type of the array elements.

`  structType  `

`  object ( StructType  ` )

If `  code  ` == `  STRUCT  ` , then `  structType  ` provides type information for the struct's fields.

`  typeAnnotation  `

`  enum ( TypeAnnotationCode  ` )

The `  TypeAnnotationCode  ` that disambiguates SQL type that Spanner will use to represent values of this type during query processing. This is necessary for some type codes because a single `  TypeCode  ` can be mapped to different SQL types depending on the SQL dialect. `  typeAnnotation  ` typically is not needed to process the content of a value (it doesn't affect serialization) and clients can ignore it on the read path.

`  protoTypeFqn  `

`  string  `

If `  code  ` == `  PROTO  ` or `  code  ` == `  ENUM  ` , then `  protoTypeFqn  ` is the fully qualified name of the proto type representing the proto/enum definition.

## TypeCode

`  TypeCode  ` is used as part of `  Type  ` to indicate the type of a Cloud Spanner value.

Each legal value of a type can be encoded to or decoded from a JSON value, using the encodings described below. All Cloud Spanner values can be `  null  ` , regardless of type; `  null  ` s are always encoded as a JSON `  null  ` .

Enums

`  TYPE_CODE_UNSPECIFIED  `

Not specified.

`  BOOL  `

Encoded as JSON `  true  ` or `  false  ` .

`  INT64  `

Encoded as `  string  ` , in decimal format.

`  FLOAT64  `

Encoded as `  number  ` , or the strings `  "NaN"  ` , `  "Infinity"  ` , or `  "-Infinity"  ` .

`  FLOAT32  `

Encoded as `  number  ` , or the strings `  "NaN"  ` , `  "Infinity"  ` , or `  "-Infinity"  ` .

`  TIMESTAMP  `

Encoded as `  string  ` in RFC 3339 timestamp format. The time zone must be present, and must be `  "Z"  ` .

If the schema has the column option `  allow_commit_timestamp=true  ` , the placeholder string `  "spanner.commit_timestamp()"  ` can be used to instruct the system to insert the commit timestamp associated with the transaction commit.

`  DATE  `

Encoded as `  string  ` in RFC 3339 date format.

`  STRING  `

Encoded as `  string  ` .

`  BYTES  `

Encoded as a base64-encoded `  string  ` , as described in RFC 4648, section 4.

`  ARRAY  `

Encoded as `  list  ` , where the list elements are represented according to `  arrayElementType  ` .

`  STRUCT  `

Encoded as `  list  ` , where list element `  i  ` is represented according to `  structType.fields[i]  ` .

`  NUMERIC  `

Encoded as `  string  ` , in decimal format or scientific notation format. Decimal format: `  [+-]Digits[.[Digits]]  ` or `  [+-][Digits].Digits  `

Scientific notation: `  [+-]Digits[.[Digits]][ExponentIndicator[+-]Digits]  ` or `  [+-][Digits].Digits[ExponentIndicator[+-]Digits]  ` (ExponentIndicator is `  "e"  ` or `  "E"  ` )

`  JSON  `

Encoded as a JSON-formatted `  string  ` as described in RFC 7159. The following rules are applied when parsing JSON input:

  - Whitespace characters are not preserved.
  - If a JSON object has duplicate keys, only the first key is preserved.
  - Members of a JSON object are not guaranteed to have their order preserved.
  - JSON array elements will have their order preserved.

`  PROTO  `

Encoded as a base64-encoded `  string  ` , as described in RFC 4648, section 4.

`  ENUM  `

Encoded as `  string  ` , in decimal format.

`  UUID  `

Encoded as `  string  ` , in lower-case hexa-decimal format, as described in RFC 9562, section 4.

## TypeAnnotationCode

`  TypeAnnotationCode  ` is used as a part of `  Type  ` to disambiguate SQL types that should be used for a given Cloud Spanner value. Disambiguation is needed because the same Cloud Spanner type can be mapped to different SQL types depending on SQL dialect. TypeAnnotationCode doesn't affect the way value is serialized.

Enums

`  TYPE_ANNOTATION_CODE_UNSPECIFIED  `

Not specified.

`  PG_NUMERIC  `

PostgreSQL compatible NUMERIC type. This annotation needs to be applied to `  Type  ` instances having `  NUMERIC  ` type code to specify that values of this type should be treated as PostgreSQL NUMERIC values. Currently this annotation is always needed for `  NUMERIC  ` when a client interacts with PostgreSQL-enabled Spanner databases.

`  PG_JSONB  `

PostgreSQL compatible JSONB type. This annotation needs to be applied to `  Type  ` instances having `  JSON  ` type code to specify that values of this type should be treated as PostgreSQL JSONB values. Currently this annotation is always needed for `  JSON  ` when a client interacts with PostgreSQL-enabled Spanner databases.

`  PG_OID  `

PostgreSQL compatible OID type. This annotation can be used by a client interacting with PostgreSQL-enabled Spanner database to specify that a value should be treated using the semantics of the OID type.
