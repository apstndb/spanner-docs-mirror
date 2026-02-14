**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes the `  SEARCH  ` function and the enhanced query mode, which are used to perform [full-text search](/spanner/docs/full-text-search) queries on Spanner tables.

## Query a search index

Spanner provides the [`  SEARCH  `](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) function to use for search index queries. An example use case would be an application where users enter text in a search box and the application sends the user input directly into the `  SEARCH  ` function. The `  SEARCH  ` function would then use a search index to find that text.

The `  SEARCH  ` function requires two arguments:

  - A search index name
  - A search query

The `  SEARCH  ` function only works when a search index is defined. The `  SEARCH  ` function can be combined with any arbitrary SQL constructs, such as filters, aggregations, or joins.

The `  SEARCH  ` function can't be used with transaction queries.

The following query uses the `  SEARCH  ` function to return all albums that have either `  friday  ` or `  monday  ` in the title:

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, 'friday OR monday')
```

### PostgreSQL

This example uses [spanner.search](/spanner/docs/reference/postgresql/functions-and-operators#spannersearch) .

``` text
SELECT albumid
FROM albums
WHERE spanner.search(albumtitle_tokens, 'friday OR monday')
```

### Search query

Search queries use the [raw search query](/spanner/docs/reference/standard-sql/search_functions#rquery-syntax) syntax by default. Alternative syntaxes can be specified using the `  SEARCH  ` [`  dialect  `](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) argument.

#### rquery dialect

The default dialect is [raw search query](/spanner/docs/reference/standard-sql/search_functions#rquery-syntax) . Spanner uses a domain-specific language (DSL) called *rquery* .

The rquery language follows the same rules as the [plain-text tokenizer](/spanner/docs/full-text-search/tokenization#tokenize_plain_text_or_html_content) when splitting the input search query into distinct terms. This includes segmentation of Asian languages.

For information about using rquery, see [rquery syntax](/spanner/docs/reference/standard-sql/search_functions#rquery-syntax) .

#### words dialect

The words dialect is like rquery, but simpler. It doesn't use any special operators. For example, `  OR  ` is treated as a search term instead of a disjunction operator. The double quotes are handled as punctuations rather than a phrase search and they are ignored.

With the words dialect, `  AND  ` is implicitly applied to all terms, and is required during matching. It follows the same rules as the plain-text tokenizer when splitting the input search query into terms.

For information about using the words dialect, see [words syntax](/spanner/docs/reference/standard-sql/search_functions#words-syntax) .

#### words\_phrase dialect

The words\_phrase dialect doesn't use any special operators and all terms are treated as a phrase, meaning the terms are required to be adjacent and in the order specified.

Same as rquery, the words\_phrase dialect follows the same rules as the plain-text tokenizer when splitting the input search query into terms.

For information about using the words\_phrase dialect, see [words phrase syntax](/spanner/docs/reference/standard-sql/search_functions#words-phrase-syntax) .

## Enhanced query mode

Spanner offers two full-text search modes: a basic token-based search and a more advanced mode called *`  enhance_query  `* . When enabled, `  enhance_query  ` expands the search query to include related terms and synonyms, increasing the likelihood of finding relevant results.

To enable this option, set the optional argument `  enhance_query=>true  ` in the `  SEARCH  ` function. For example, the search query `  hotl cal  ` matches the album `  Hotel California  ` .

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, 'hotl cal', enhance_query=>true)
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE spanner.search(albumtitle_tokens, 'hotl cal', enhance_query=>true)
```

The `  enhance_query  ` mode is a query-time option. It doesn't affect tokenization. You can use the same search index with or without `  enhance_query  ` .

Google is continuously improving the query enhancement algorithms. As a result, a query with `  enhance_query == true  ` might yield slightly different results over time.

When the `  enhance_query  ` mode is enabled, it might increase the number of terms that the `  SEARCH  ` function is looking for which could slightly elevate latency.

## SQL query requirements

There are several conditions that a SQL query must meet to use a search index. If these conditions aren't met, the query uses either an alternative query plan or fails if no alternative plan exists.

Queries must meet the following conditions:

  - [SEARCH function](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) and [`  SEARCH_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#search_substring) functions require a search index. Spanner doesn't support these functions in queries against the base table or secondary indexes.

  - [Partitioned indexes](/spanner/docs/full-text-search/partition-search-index) must have all partition columns bound by an equality condition in the `  WHERE  ` clause of the query.
    
    For example, if a search index is defined as `  PARTITION BY x, y  ` , the query must have a conjunct in the `  WHERE  ` clause of `  x = <parameter or constant> AND y = <parameter or constant>  ` . That search index isn't considered by the query optimizer if such a condition is missing.

  - All `  TOKENLIST  ` columns referenced by `  SEARCH  ` and `  SEARCH_SUBSTRING  ` operators must be indexed in the same search index.
    
    For example, consider the following table and index definition:
    
    ### GoogleSQL
    
    ``` text
    CREATE TABLE Albums (
        AlbumId STRING(MAX) NOT NULL,
        AlbumTitle STRING(MAX),
        AlbumStudio STRING(MAX),
        AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN,
        AlbumStudio_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumStudio)) HIDDEN
    ) PRIMARY KEY(AlbumId);
    
    CREATE SEARCH INDEX AlbumsTitleIndex ON Albums(AlbumTitle_Tokens);
    CREATE SEARCH INDEX AlbumsStudioIndex ON Albums(AlbumStudio_Tokens);
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE TABLE albums (
        albumid character varying NOT NULL,
        albumtitle character varying,
        albumstudio character varying,
        albumtitle_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
        albumstudio_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumstudio)) VIRTUAL HIDDEN,
    PRIMARY KEY(albumid));
    
    CREATE SEARCH INDEX albumstitleindex ON albums(albumtitle_tokens);
    CREATE SEARCH INDEX albumsstudioindex ON albums(albumstudio_tokens);
    ```
    
    The following query fails because there's no single search index that indexes both `  AlbumTitle_Tokens  ` and `  AlbumStudio_Tokens  ` :
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, @p1)
        AND SEARCH(AlbumStudio_Tokens, @p2)
    ```
    
    ### PostgreSQL
    
    This example uses query parameters `  $1  ` and `  $2  ` which are bound to 'fast car' and 'blue note', respectively.
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, $1)
        AND spanner.search(albumstudio_tokens, $2)
    ```

  - If the sort order column is nullable, both the schema and the query must exclude rows where the sort order column is NULL. For details, see [Search index sort order](/spanner/docs/full-text-search/search-indexes#search-index-sort-order) .

  - If the search index is NULL filtered, the query must include the same NULL-filtering expression that's used in an index. See [NULL-filtered search indexes](/spanner/docs/full-text-search/search-indexes#null-filtered-indexes) for details.

  - [Search indexes](/spanner/docs/full-text-search/search-indexes) and [search functions](/spanner/docs/reference/standard-sql/search_functions) aren't supported in DML, partitioned DML, or partitioned queries.

  - [Search indexes](/spanner/docs/full-text-search/search-indexes) and [search functions](/spanner/docs/reference/standard-sql/search_functions) are typically used in [read-only transactions](/spanner/docs/transactions#read-only_transactions) . If application requirements allow stale results, you might be able to improve latency by running search queries with a staleness duration of 10 seconds or longer. For more information, see [Read stale data](/spanner/docs/samples/spanner-read-stale-data) . This is particularly useful for search queries that fan out to many index splits.

[Search indexes](/spanner/docs/full-text-search/search-indexes) and [search functions](/spanner/docs/reference/standard-sql/search_functions) are not recommended in [read-write transactions](/spanner/docs/transactions#read-write_transactions) . During execution, search queries lock an entire index partition; as a result, a high rate of search queries in read-write transactions might cause lock conflicts leading to latency spikes. By default, search indexes are not automatically selected in read-write transactions. If a query is forced to use a search index in a read-write transaction it fails by default. It also fails if the query contains any of the search functions. This behavior can be overridden with the GoogleSQL `  @{ALLOW_SEARCH_INDEXES_IN_TRANSACTION=TRUE}  ` statement-level hint (but queries are still prone to lock conflicts).

Once index eligibility conditions are met, the query optimizer tries to accelerate non-text query conditions (like `  Rating > 4  ` ). If the search index doesn't include the appropriate `  TOKENLIST  ` column, the condition isn't accelerated and remains a [residual condition](/spanner/docs/query-execution-operators#filter_scan) .

## Query parameters

Search query arguments are specified as either a literal or a [query parameter](/spanner/docs/reference/standard-sql/lexical#query_parameters) . We recommend using query parameters for full-text search rather than string literals when arguments allow query parameter value.

## Index selection

Spanner typically selects the most efficient index for a query using cost-based modeling. However, the `  FORCE_INDEX  ` hint explicitly instructs Spanner to use a specific search index. For example, the following shows how to force Spanner to use the `  AlbumsIndex  ` :

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums @{FORCE_INDEX=AlbumsIndex}
WHERE SEARCH(AlbumTitle_Tokens, "fifth symphony")
```

### PostgreSQL

``` text
SELECT albumid
FROM albums/*@force_index=albumsindex*/
WHERE spanner.search(albumtitle_tokens, 'fifth symphony')
```

If the specified search index isn't [eligible](#sql-query-requirements) , the query fails, even if there are other eligible search indexes.

## Snippets in search results

A snippet is a piece of text extracted from a given string that gives users a sense of what a search result contains, and the reason why the result is relevant to their query.

For example, Gmail uses snippets to indicate the portion of an email that matches the search query:

Having the database generate a snippet has several benefits:

1.  **Convenience** : You don't need to implement logic to generate snippets from a search query.
2.  **Efficiency** : Snippets reduce the output size from the server.

The [`  SNIPPET  `](/spanner/docs/reference/standard-sql/search_functions#snippet) function creates the snippet. It returns the relevant portion of the original string value along with positions of characters to highlight. The client can then choose how to display the snippet to the end user (for example, using highlighted or bold text).

For example, the following uses `  SNIPPET  ` to retrieve text from `  AlbumTitle  ` :

### GoogleSQL

``` text
SELECT AlbumId, SNIPPET(AlbumTitle, "Fast Car")
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, "Fast Car")
```

### PostgreSQL

This example uses [spanner.snippet](/spanner/docs/reference/postgresql/functions-and-operators#spannersnippet) .

``` text
SELECT albumid, spanner.snippet(albumtitle, 'Fast Car')
FROM albums
WHERE spanner.search(albumtitle_tokens, 'Fast Car')
```

## What's next

  - Learn how to [rank search results](/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [perform a substring search](/spanner/docs/full-text-search/substring-search) .
  - Learn how to [paginate search results](/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [mix full-text and non-text queries](/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
  - Learn how to [search multiple columns](/spanner/docs/full-text-search/search-multiple-columns) .
