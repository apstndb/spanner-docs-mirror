---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/Type
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type
title: Type
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#SCHEMA_REPRESENTATION)
- [TypeCode](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode)
- [TypeAnnotationCode](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeAnnotationCode)

`Type` indicates the type of a Cloud Spanner value, as might be stored in a table cell or returned from an SQL query.

**JSON representation**

```
{
  "code": enum (TypeCode),
  "arrayElementType": {
    object (Type)
  },
  "structType": {
    object (StructType)
  },
  "typeAnnotation": enum (TypeAnnotationCode),
  "protoTypeFqn": string
}
```

| Fields             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `code`             | `enum ( `[`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode)` )` Required. The [`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode) for this type.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `arrayElementType` | `object ( `[`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type)` )` If [`code`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#FIELDS.code) == [`ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.ARRAY) , then `arrayElementType` is the type of the array elements.                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `structType`       | `object ( `[`StructType`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType)` )` If [`code`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#FIELDS.code) == [`STRUCT`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.STRUCT) , then `structType` provides type information for the struct's fields.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `typeAnnotation`   | `enum ( `[`TypeAnnotationCode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeAnnotationCode)` )` The [`TypeAnnotationCode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeAnnotationCode) that disambiguates SQL type that Spanner will use to represent values of this type during query processing. This is necessary for some type codes because a single [`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode) can be mapped to different SQL types depending on the SQL dialect. [`typeAnnotation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#FIELDS.type_annotation) typically is not needed to process the content of a value (it doesn't affect serialization) and clients can ignore it on the read path. |
| `protoTypeFqn`     | `string` If [`code`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#FIELDS.code) == [`PROTO`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.PROTO) or [`code`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#FIELDS.code) == [`ENUM`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.ENUM) , then `protoTypeFqn` is the fully qualified name of the proto type representing the proto/enum definition.                                                                                                                                                                                                                                                                                                 |

## TypeCode

`TypeCode` is used as part of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type) to indicate the type of a Cloud Spanner value.

Each legal value of a type can be encoded to or decoded from a JSON value, using the encodings described below. All Cloud Spanner values can be `null` , regardless of type; `null` s are always encoded as a JSON `null` .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Enums</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>TYPE_CODE_UNSPECIFIED</code></td>
<td>Not specified.</td>
</tr>
<tr class="even">
<td><code>BOOL</code></td>
<td>Encoded as JSON <code>true</code> or <code>false</code> .</td>
</tr>
<tr class="odd">
<td><code>INT64</code></td>
<td>Encoded as <code>string</code> , in decimal format.</td>
</tr>
<tr class="even">
<td><code>FLOAT64</code></td>
<td>Encoded as <code>number</code> , or the strings <code>"NaN"</code> , <code>"Infinity"</code> , or <code>"-Infinity"</code> .</td>
</tr>
<tr class="odd">
<td><code>FLOAT32</code></td>
<td>Encoded as <code>number</code> , or the strings <code>"NaN"</code> , <code>"Infinity"</code> , or <code>"-Infinity"</code> .</td>
</tr>
<tr class="even">
<td><code>TIMESTAMP</code></td>
<td><p>Encoded as <code>string</code> in RFC 3339 timestamp format. The time zone must be present, and must be <code>"Z"</code> .</p>
<p>If the schema has the column option <code>allow_commit_timestamp=true</code> , the placeholder string <code>"spanner.commit_timestamp()"</code> can be used to instruct the system to insert the commit timestamp associated with the transaction commit.</p></td>
</tr>
<tr class="odd">
<td><code>DATE</code></td>
<td>Encoded as <code>string</code> in RFC 3339 date format.</td>
</tr>
<tr class="even">
<td><code>STRING</code></td>
<td>Encoded as <code>string</code> .</td>
</tr>
<tr class="odd">
<td><code>BYTES</code></td>
<td>Encoded as a base64-encoded <code>string</code> , as described in RFC 4648, section 4.</td>
</tr>
<tr class="even">
<td><code>ARRAY</code></td>
<td>Encoded as <code>list</code> , where the list elements are represented according to <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#FIELDS.array_element_type"><code>arrayElementType</code></a> .</td>
</tr>
<tr class="odd">
<td><code>STRUCT</code></td>
<td>Encoded as <code>list</code> , where list element <code>i</code> is represented according to <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType#FIELDS.fields"><code>structType.fields[i]</code></a> .</td>
</tr>
<tr class="even">
<td><code>NUMERIC</code></td>
<td><p>Encoded as <code>string</code> , in decimal format or scientific notation format. Decimal format: <code>[+-]Digits[.[Digits]]</code> or <code>[+-][Digits].Digits</code></p>
<p>Scientific notation: <code>[+-]Digits[.[Digits]][ExponentIndicator[+-]Digits]</code> or <code>[+-][Digits].Digits[ExponentIndicator[+-]Digits]</code> (ExponentIndicator is <code>"e"</code> or <code>"E"</code> )</p></td>
</tr>
<tr class="odd">
<td><code>JSON</code></td>
<td><p>Encoded as a JSON-formatted <code>string</code> as described in RFC 7159. The following rules are applied when parsing JSON input:</p>
<ul>
<li>Whitespace characters are not preserved.</li>
<li>If a JSON object has duplicate keys, only the first key is preserved.</li>
<li>Members of a JSON object are not guaranteed to have their order preserved.</li>
<li>JSON array elements will have their order preserved.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>PROTO</code></td>
<td>Encoded as a base64-encoded <code>string</code> , as described in RFC 4648, section 4.</td>
</tr>
<tr class="odd">
<td><code>ENUM</code></td>
<td>Encoded as <code>string</code> , in decimal format.</td>
</tr>
<tr class="even">
<td><code>UUID</code></td>
<td>Encoded as <code>string</code> , in lower-case hexa-decimal format, as described in RFC 9562, section 4.</td>
</tr>
</tbody>
</table>

## TypeAnnotationCode

`TypeAnnotationCode` is used as a part of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type) to disambiguate SQL types that should be used for a given Cloud Spanner value. Disambiguation is needed because the same Cloud Spanner type can be mapped to different SQL types depending on SQL dialect. TypeAnnotationCode doesn't affect the way value is serialized.

| Enums                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `TYPE_ANNOTATION_CODE_UNSPECIFIED` | Not specified.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `PG_NUMERIC`                       | PostgreSQL compatible NUMERIC type. This annotation needs to be applied to [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type) instances having [`NUMERIC`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.NUMERIC) type code to specify that values of this type should be treated as PostgreSQL NUMERIC values. Currently this annotation is always needed for [`NUMERIC`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.NUMERIC) when a client interacts with PostgreSQL-enabled Spanner databases. |
| `PG_JSONB`                         | PostgreSQL compatible JSONB type. This annotation needs to be applied to [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type) instances having [`JSON`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.JSON) type code to specify that values of this type should be treated as PostgreSQL JSONB values. Currently this annotation is always needed for [`JSON`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.JSON) when a client interacts with PostgreSQL-enabled Spanner databases.                 |
| `PG_OID`                           | PostgreSQL compatible OID type. This annotation can be used by a client interacting with PostgreSQL-enabled Spanner database to specify that a value should be treated using the semantics of the OID type.                                                                                                                                                                                                                                                                                                                                                                                                     |
