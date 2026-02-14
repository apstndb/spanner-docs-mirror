**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes the features for search and secondary indexes.

When deciding between search indexes and secondary indexes, keep in mind that search indexes should be the default choice for full-text use cases and secondary indexes should be the default option for everything else. The following table describes when to use each type of index.

<table>
<thead>
<tr class="header">
<th>Feature</th>
<th>Secondary index</th>
<th>Search index</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Sort order</td>
<td>The index is sorted by all index key columns</td>
<td>Search index can only be sorted by at most two user-controlled INT64 columns</td>
</tr>
<tr class="even">
<td>Arrays</td>
<td>You can't use arrays data type values as keys in secondary indexes.</td>
<td>Search indexes support array indexing. All tokens of a document are colocated in the same split. As a result, transactions that change 1 row only write to one index split.</td>
</tr>
<tr class="odd">
<td>JSON/JSONB</td>
<td>You can't use JSON values as keys in secondary indexes.</td>
<td>Search indexes support indexing JSON values.</td>
</tr>
<tr class="even">
<td>Lookups</td>
<td>Lookup by index key only needs to access one split</td>
<td>Queries that use search index generally need to read from all splits of a given partition. The only exception is top-k pattern matching.</td>
</tr>
<tr class="odd">
<td>Multi-column indexing</td>
<td>Secondary indexes can include multiple key columns. Queries need to look up data by prefix of index key columns for efficient execution.</td>
<td>Search indexes can index multiple columns. Queries can specify complex logical expressions (conjunctions, disjunctions, negations) on any subset of the indexed columns, and still execute efficiently</td>
</tr>
<tr class="even">
<td>Index intersection</td>
<td>Users can rewrite their query to join multiple secondary indexes.</td>
<td>Intersection of multiple indexed columns is implemented as an efficient local zig-zag join, followed by a distributed merge union (that combines results from all relevant splits).</td>
</tr>
<tr class="odd">
<td>Reading data</td>
<td>SQL Query or Read API</td>
<td>SQL Query</td>
</tr>
</tbody>
</table>

Besides semantics, the syntax of the DDL statement to create a search index is different from the DDL syntax to create a secondary index:

  - Indexed columns are defined separately from the sort order in the search index.
  - The order of `  TOKENLIST  ` columns in the `  ON  ` clause of the `  CREATE SEARCH INDEX  ` statement is immaterial.

## What's next

  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
  - Learn about [secondary indexes](/spanner/docs/secondary-indexes) .
