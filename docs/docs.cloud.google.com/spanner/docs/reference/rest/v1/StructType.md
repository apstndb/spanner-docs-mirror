  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [Field](#Field)
      - [JSON representation](#Field.SCHEMA_REPRESENTATION)

`  StructType  ` defines the fields of a `  STRUCT  ` type.

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

## Field

Message representing a single field of a struct.

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
