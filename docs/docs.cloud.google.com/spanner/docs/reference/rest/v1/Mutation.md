  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [Write](#Write)
      - [JSON representation](#Write.SCHEMA_REPRESENTATION)
  - [Delete](#Delete)
      - [JSON representation](#Delete.SCHEMA_REPRESENTATION)

A modification to one or more Cloud Spanner rows. Mutations can be applied to a Cloud Spanner database by sending them in a `  Commit  ` call.

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

  // Union field operation can be only one of the following:
  &quot;insert&quot;: {
    object (Write)
  },
  &quot;update&quot;: {
    object (Write)
  },
  &quot;insertOrUpdate&quot;: {
    object (Write)
  },
  &quot;replace&quot;: {
    object (Write)
  },
  &quot;delete&quot;: {
    object (Delete)
  }
  // End of list of possible types for union field operation.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  operation  ` . Required. The operation to perform. `  operation  ` can be only one of the following:

`  insert  `

`  object ( Write  ` )

Insert new rows in a table. If any of the rows already exist, the write or transaction fails with error `  ALREADY_EXISTS  ` .

`  update  `

`  object ( Write  ` )

Update existing rows in a table. If any of the rows does not already exist, the transaction fails with error `  NOT_FOUND  ` .

`  insertOrUpdate  `

`  object ( Write  ` )

Like `  insert  ` , except that if the row already exists, then its column values are overwritten with the ones provided. Any column values not explicitly written are preserved.

When using `  insertOrUpdate  ` , just as when using `  insert  ` , all `  NOT NULL  ` columns in the table must be given a value. This holds true even when the row already exists and will therefore actually be updated.

`  replace  `

`  object ( Write  ` )

Like `  insert  ` , except that if the row already exists, it is deleted, and the column values provided are inserted instead. Unlike `  insertOrUpdate  ` , this means any values not explicitly written become `  NULL  ` .

In an interleaved table, if you create the child table with the `  ON DELETE CASCADE  ` annotation, then replacing a parent row also deletes the child rows. Otherwise, you must delete the child rows before you replace the parent row.

`  delete  `

`  object ( Delete  ` )

Delete rows from a table. Succeeds whether or not the named rows were present.

## Write

Arguments to `  insert  ` , `  update  ` , `  insertOrUpdate  ` , and `  replace  ` operations.

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
  &quot;table&quot;: string,
  &quot;columns&quot;: [
    string
  ],
  &quot;values&quot;: [
    array
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  table  `

`  string  `

Required. The table whose rows will be written.

`  columns[]  `

`  string  `

The names of the columns in `  table  ` to be written.

The list of columns must contain enough columns to allow Cloud Spanner to derive values for all primary key columns in the row(s) to be modified.

`  values[]  `

`  array ( ListValue  ` format)

The values to be written. `  values  ` can contain more than one list of values. If it does, then multiple rows are written, one for each entry in `  values  ` . Each list in `  values  ` must have exactly as many entries as there are entries in `  columns  ` above. Sending multiple lists is equivalent to sending multiple `  Mutation  ` s, each containing one `  values  ` entry and repeating `  table  ` and `  columns  ` . Individual values in each list are encoded as described `  here  ` .

## Delete

Arguments to `  delete  ` operations.

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
  &quot;table&quot;: string,
  &quot;keySet&quot;: {
    object (KeySet)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  table  `

`  string  `

Required. The table whose rows will be deleted.

`  keySet  `

`  object ( KeySet  ` )

Required. The primary keys of the rows within `  table  ` to delete. The primary keys must be specified in the order in which they appear in the `  PRIMARY KEY()  ` clause of the table's equivalent DDL statement (the DDL statement used to create the table). Delete is idempotent. The transaction will succeed even if some or all rows do not exist.
