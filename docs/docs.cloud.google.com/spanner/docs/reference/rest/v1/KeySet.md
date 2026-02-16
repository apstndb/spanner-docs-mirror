  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [KeyRange](#KeyRange)
      - [JSON representation](#KeyRange.SCHEMA_REPRESENTATION)

`  KeySet  ` defines a collection of Cloud Spanner keys and/or key ranges. All the keys are expected to be in the same table or index. The keys need not be sorted in any particular way.

If the same key is specified multiple times in the set (for example if two ranges, two keys, or a key and a range overlap), Cloud Spanner behaves as if the key were only specified once.

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
  &quot;keys&quot;: [
    array
  ],
  &quot;ranges&quot;: [
    {
      object (KeyRange)
    }
  ],
  &quot;all&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  keys[]  `

`  array ( ListValue  ` format)

A list of specific keys. Entries in `  keys  ` should have exactly as many elements as there are columns in the primary or index key with which this `  KeySet  ` is used. Individual key values are encoded as described `  here  ` .

`  ranges[]  `

`  object ( KeyRange  ` )

A list of key ranges. See `  KeyRange  ` for more information about key range specifications.

`  all  `

`  boolean  `

For convenience `  all  ` can be set to `  true  ` to indicate that this `  KeySet  ` matches all keys in the table or index. Note that any keys specified in `  keys  ` or `  ranges  ` are only yielded once.

## KeyRange

KeyRange represents a range of rows in a table or index.

A range has a start key and an end key. These keys can be open or closed, indicating if the range includes rows with that key.

Keys are represented by lists, where the ith value in the list corresponds to the ith component of the table or index primary key. Individual values are encoded as described `  here  ` .

For example, consider the following table definition:

``` text
CREATE TABLE UserEvents (
  UserName STRING(MAX),
  EventDate STRING(10)
) PRIMARY KEY(UserName, EventDate);
```

The following keys name rows in this table:

``` text
["Bob", "2014-09-23"]
["Alfred", "2015-06-12"]
```

Since the `  UserEvents  ` table's `  PRIMARY KEY  ` clause names two columns, each `  UserEvents  ` key has two elements; the first is the `  UserName  ` , and the second is the `  EventDate  ` .

Key ranges with multiple components are interpreted lexicographically by component using the table or index key's declared sort order. For example, the following range returns all events for user `  "Bob"  ` that occurred in the year 2015:

``` text
"startClosed": ["Bob", "2015-01-01"]
"endClosed": ["Bob", "2015-12-31"]
```

Start and end keys can omit trailing key components. This affects the inclusion and exclusion of rows that exactly match the provided key components: if the key is closed, then rows that exactly match the provided components are included; if the key is open, then rows that exactly match are not included.

For example, the following range includes all events for `  "Bob"  ` that occurred during and after the year 2000:

``` text
"startClosed": ["Bob", "2000-01-01"]
"endClosed": ["Bob"]
```

The next example retrieves all events for `  "Bob"  ` :

``` text
"startClosed": ["Bob"]
"endClosed": ["Bob"]
```

To retrieve events before the year 2000:

``` text
"startClosed": ["Bob"]
"endOpen": ["Bob", "2000-01-01"]
```

The following range includes all rows in the table:

``` text
"startClosed": []
"endClosed": []
```

This range returns all users whose `  UserName  ` begins with any character from A to C:

``` text
"startClosed": ["A"]
"endOpen": ["D"]
```

This range returns all users whose `  UserName  ` begins with B:

``` text
"startClosed": ["B"]
"endOpen": ["C"]
```

Key ranges honor column sort order. For example, suppose a table is defined as follows:

``` text
CREATE TABLE DescendingSortedTable {
  Key INT64,
  ...
) PRIMARY KEY(Key DESC);
```

The following range retrieves all rows with key values between 1 and 100 inclusive:

``` text
"startClosed": ["100"]
"endClosed": ["1"]
```

Note that 100 is passed as the start, and 1 is passed as the end, because `  Key  ` is a descending column in the schema.

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

  // Union field start_key_type can be only one of the following:
  &quot;startClosed&quot;: array,
  &quot;startOpen&quot;: array
  // End of list of possible types for union field start_key_type.

  // Union field end_key_type can be only one of the following:
  &quot;endClosed&quot;: array,
  &quot;endOpen&quot;: array
  // End of list of possible types for union field end_key_type.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  start_key_type  ` . The start key must be provided. It can be either closed or open. `  start_key_type  ` can be only one of the following:

`  startClosed  `

`  array ( ListValue  ` format)

If the start is closed, then the range includes all rows whose first `  len(startClosed)  ` key columns exactly match `  startClosed  ` .

`  startOpen  `

`  array ( ListValue  ` format)

If the start is open, then the range excludes rows whose first `  len(startOpen)  ` key columns exactly match `  startOpen  ` .

Union field `  end_key_type  ` . The end key must be provided. It can be either closed or open. `  end_key_type  ` can be only one of the following:

`  endClosed  `

`  array ( ListValue  ` format)

If the end is closed, then the range includes all rows whose first `  len(endClosed)  ` key columns exactly match `  endClosed  ` .

`  endOpen  `

`  array ( ListValue  ` format)

If the end is open, then the range excludes rows whose first `  len(endOpen)  ` key columns exactly match `  endOpen  ` .
