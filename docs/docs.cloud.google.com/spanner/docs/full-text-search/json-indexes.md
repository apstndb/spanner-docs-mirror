**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

In addition to indexing text, the Spanner [search index](/spanner/docs/full-text-search/search-indexes) provides an efficient way to index and query JSON and JSONB documents. Use search indexes for standalone JSON and JSONB queries, or to augment other [full-text search](/spanner/docs/full-text-search) queries.

For more information, see [Index JSON data](/spanner/docs/working-with-json#index) and [Index JSONB data](/spanner/docs/working-with-jsonb#index) .

## Tokenize JSON and JSONB

You can use the `  TOKENIZE_JSON  ` function to create a JSON index in GoogleSQL, or the `  TOKENIZE_JSONB  ` function to create a JSONB index in PostgreSQL. For details, see [`  TOKENIZE_JSON  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_json) and [`  TOKENIZE_JSONB  `](/spanner/docs/reference/postgresql/functions-and-operators#indexing) .

## JSON and JSONB queries

You can use a search index to accelerate queries that include *JSON containment* and *key existence* conditions. JSON containment determines if one JSON document is contained within another. Key existence determines if a key exists in the database schema.

  - In GoogleSQL:
    
      - Express JSON containment in your schema by using the [`  JSON_CONTAINS  `](/spanner/docs/reference/standard-sql/json_functions#json_contains) function.
      - Construct key existence conditions using the field access, array subscript operators, and `  IS NOT NULL  ` . The field access and array subscript operators describe a JSON document path. `  IS NOT NULL  ` checks for the existence of this path (for example, `  doc.sub.path[@index].key IS NOT NULL  ` ).

  - In PostgreSQL:
    
      - Express JSONB containment using the `  @>  ` and `  <@  ` operators. For more information, see [JSONB operators](/spanner/docs/reference/postgresql/functions-and-operators#jsonb_operators) .
      - Construct key existence conditions using the `  ?  ` , `  ?|  ` , and `  ?&  ` operators. For more information, see [JSONB operators](/spanner/docs/reference/postgresql/functions-and-operators#jsonb_operators) .

In your queries, you can include multiple JSON conditions of any type in the search index. You can also include the JSON conditions in a logical combination using `  AND  ` , `  OR  ` , and `  NOT  ` .

### Check search index usage

To check that that your query uses a search index, look for a *Search index scan* node in the [query execution plan](/spanner/docs/query-execution-plans) .

## Restrictions

  - Search indexes, including JSON and JSONB search indexes, are used only in read-only transactions. Spanner might use relevant secondary indexes in a read-write transaction. If you attempt to force the use of a search index in a read-write transaction, the following error occurs: `  ERROR: spanner: code = "InvalidArgument", desc = "The search index AlbumsIndex cannot be used in transactional queries by default."  `
  - Attempts to store certain large or very complex JSON documents in a search index might return a `  too many search token bytes  ` error. The output token size from this JSON document must be smaller than 10 MB. If you don't need the entire document to be searchable, consider extracting a smaller subset of the document (for example, by using a generated column) and searching over the column instead.

## What's next

  - Learn about [tokenization and tokenizers](/spanner/docs/full-text-search/tokenization) .
  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
  - Learn about [indexing JSON data](/spanner/docs/working-with-json#index) .
  - Learn about [indexing JSONB data](/spanner/docs/working-with-jsonb#index) .
