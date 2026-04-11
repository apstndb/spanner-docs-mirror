**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This page describes the features for search and secondary indexes.

When deciding between search indexes and secondary indexes, keep in mind that search indexes should be the default choice for full-text use cases and secondary indexes should be the default option for everything else. The following table describes when to use each type of index.

| Feature               | Secondary index                                                                                                                          | Search index                                                                                                                                                                                           |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Sort order            | The index is sorted by all index key columns                                                                                             | Search index can only be sorted by at most two user-controlled INT64 columns                                                                                                                           |
| Arrays                | You can't use arrays data type values as keys in secondary indexes.                                                                      | Search indexes support array indexing. All tokens of a document are colocated in the same split. As a result, transactions that change 1 row only write to one index split.                            |
| JSON/JSONB            | You can't use JSON values as keys in secondary indexes.                                                                                  | Search indexes support indexing JSON values.                                                                                                                                                           |
| Lookups               | Lookup by index key only needs to access one split                                                                                       | Queries that use search index generally need to read from all splits of a given partition. The only exception is top-k pattern matching.                                                               |
| Multi-column indexing | Secondary indexes can include multiple key columns. Queries need to look up data by prefix of index key columns for efficient execution. | Search indexes can index multiple columns. Queries can specify complex logical expressions (conjunctions, disjunctions, negations) on any subset of the indexed columns, and still execute efficiently |
| Index intersection    | Users can rewrite their query to join multiple secondary indexes.                                                                        | Intersection of multiple indexed columns is implemented as an efficient local zig-zag join, followed by a distributed merge union (that combines results from all relevant splits).                    |
| Reading data          | SQL Query or Read API                                                                                                                    | SQL Query                                                                                                                                                                                              |

Besides semantics, the syntax of the DDL statement to create a search index is different from the DDL syntax to create a secondary index:

  - Indexed columns are defined separately from the sort order in the search index.
  - The order of `TOKENLIST` columns in the `ON` clause of the `CREATE SEARCH INDEX` statement is immaterial.

## What's next

  - Learn about [search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) .
  - Learn about [secondary indexes](https://docs.cloud.google.com/spanner/docs/secondary-indexes) .
