> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This page describes the `SEARCH` function and its various advanced capabilities, which are used to perform [full-text search](https://docs.cloud.google.com/spanner/docs/full-text-search) queries on Spanner tables.

## Query a search index

Spanner provides the [`SEARCH`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_fulltext) function to use for search index queries. An example use case would be an application where users enter text in a search box and the application sends the user input directly into the `SEARCH` function. The `SEARCH` function would then use a search index to find that text.

The `SEARCH` function requires two arguments:

  - A search index name
  - A search query

The `SEARCH` function only works when a search index is defined. The `SEARCH` function can be combined with any arbitrary SQL constructs, such as filters, aggregations, or joins.

The `SEARCH` function can't be used with transaction queries.

The following query uses the `SEARCH` function to return all albums that have either `friday` or `monday` in the title:

### GoogleSQL

    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, 'friday OR monday')

### PostgreSQL

This example uses [spanner.search](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions-and-operators#spannersearch) .

    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'friday OR monday')

### Search query

Search queries use the [raw search query](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#rquery-syntax) syntax by default. Alternative syntaxes can be specified using the `SEARCH` [`dialect`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_fulltext) argument.

#### rquery dialect

The default dialect is [raw search query](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#rquery-syntax) . Spanner uses a domain-specific language (DSL) called *rquery* .

The rquery language follows the same rules as the [plain-text tokenizer](https://docs.cloud.google.com/spanner/docs/full-text-search/tokenization#tokenize_plain_text_or_html_content) when splitting the input search query into distinct terms. This includes segmentation of Asian languages.

For information about using rquery, see [rquery syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#rquery-syntax) .

#### words dialect

The words dialect is like rquery, but simpler. It doesn't use any special operators. For example, `OR` is treated as a search term instead of a disjunction operator. The double quotes are handled as punctuations rather than a phrase search and they are ignored.

With the words dialect, `AND` is implicitly applied to all terms, and is required during matching. It follows the same rules as the plain-text tokenizer when splitting the input search query into terms.

For information about using the words dialect, see [words syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#words-syntax) .

#### words\_phrase dialect

The words\_phrase dialect doesn't use any special operators and all terms are treated as a phrase, meaning the terms are required to be adjacent and in the order specified.

Same as rquery, the words\_phrase dialect follows the same rules as the plain-text tokenizer when splitting the input search query into terms.

For information about using the words\_phrase dialect, see [words phrase syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#words-phrase-syntax) .

### Expand search queries to increase related results

You can increase the likelihood of finding relevant results with Spanner's advanced capabilities to expand search queries with related terms, synonyms, and spelling corrections. These capabilities include:

  - [Enhanced query](https://docs.cloud.google.com/spanner/docs/full-text-search/search-query-enhancement#enhanced-query)
  - [Custom dictionaries](https://docs.cloud.google.com/spanner/docs/full-text-search/search-query-enhancement#custom-dictionaries)

For more information, see [Search with query enhancement](https://docs.cloud.google.com/spanner/docs/full-text-search/search-query-enhancement) .

## SQL query requirements

There are several conditions that a SQL query must meet to use a search index. If these conditions aren't met, the query uses either an alternative query plan or fails if no alternative plan exists.

Queries must meet the following conditions:

  - [SEARCH function](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_fulltext) and [`SEARCH_SUBSTRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_substring) functions require a search index. Spanner doesn't support these functions in queries against the base table or secondary indexes.

  - [Partitioned indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/partition-search-index) must have all partition columns bound by an equality condition in the `WHERE` clause of the query.
    
    For example, if a search index is defined as `PARTITION BY x, y` , the query must have a conjunct in the `WHERE` clause of `x = <parameter or constant> AND y = <parameter or constant>` . That search index isn't considered by the query optimizer if such a condition is missing.

  - All `TOKENLIST` columns referenced by `SEARCH` and `SEARCH_SUBSTRING` operators must be indexed in the same search index.
    
    For example, consider the following table and index definition:
    
    ### GoogleSQL
    
        CREATE TABLE Albums (
            AlbumId STRING(MAX) NOT NULL,
            AlbumTitle STRING(MAX),
            AlbumStudio STRING(MAX),
            AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN,
            AlbumStudio_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumStudio)) HIDDEN
        ) PRIMARY KEY(AlbumId);
        
        CREATE SEARCH INDEX AlbumsTitleIndex ON Albums(AlbumTitle_Tokens);
        CREATE SEARCH INDEX AlbumsStudioIndex ON Albums(AlbumStudio_Tokens);
    
    ### PostgreSQL
    
        CREATE TABLE albums (
            albumid character varying NOT NULL,
            albumtitle character varying,
            albumstudio character varying,
            albumtitle_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
            albumstudio_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumstudio)) VIRTUAL HIDDEN,
        PRIMARY KEY(albumid));
        
        CREATE SEARCH INDEX albumstitleindex ON albums(albumtitle_tokens);
        CREATE SEARCH INDEX albumsstudioindex ON albums(albumstudio_tokens);
    
    The following query fails because there's no single search index that indexes both `AlbumTitle_Tokens` and `AlbumStudio_Tokens` :
    
    ### GoogleSQL
    
        SELECT AlbumId
        FROM Albums
        WHERE SEARCH(AlbumTitle_Tokens, @p1)
            AND SEARCH(AlbumStudio_Tokens, @p2)
    
    ### PostgreSQL
    
    This example uses query parameters `$1` and `$2` which are bound to 'fast car' and 'blue note', respectively.
    
        SELECT albumid
        FROM albums
        WHERE spanner.search(albumtitle_tokens, $1)
            AND spanner.search(albumstudio_tokens, $2)

  - If the sort order column is nullable, both the schema and the query must exclude rows where the sort order column is NULL. For details, see [Search index sort order](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes#search-index-sort-order) .

  - If the search index is NULL filtered, the query must include the same NULL-filtering expression that's used in an index. See [NULL-filtered search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes#null-filtered-indexes) for details.

  - [Search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) and [search functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions) aren't supported in DML, partitioned DML, or partitioned queries.

  - [Search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) and [search functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions) are typically used in [read-only transactions](https://docs.cloud.google.com/spanner/docs/transactions#read-only_transactions) . If application requirements allow stale results, you might be able to improve latency by running search queries with a staleness duration of 10 seconds or longer. For more information, see [Read stale data](https://docs.cloud.google.com/spanner/docs/samples/spanner-read-stale-data) . This is particularly useful for search queries that fan out to many index splits.

[Search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) and [search functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions) are not recommended in [read-write transactions](https://docs.cloud.google.com/spanner/docs/transactions#read-write_transactions) . During execution, search queries lock an entire index partition; as a result, a high rate of search queries in read-write transactions might cause lock conflicts leading to latency spikes. By default, search indexes are not automatically selected in read-write transactions. If a query is forced to use a search index in a read-write transaction it fails by default. It also fails if the query contains any of the search functions. This behavior can be overridden with the GoogleSQL `@{ALLOW_SEARCH_INDEXES_IN_TRANSACTION=TRUE}` statement-level hint (but queries are still prone to lock conflicts).

Once index eligibility conditions are met, the query optimizer tries to accelerate non-text query conditions (like `Rating > 4` ). If the search index doesn't include the appropriate `TOKENLIST` column, the condition isn't accelerated and remains a [residual condition](https://docs.cloud.google.com/spanner/docs/query-execution-operators#filter_scan) .

## Query parameters

Search query arguments are specified as either a literal or a [query parameter](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/lexical#query_parameters) . We recommend using query parameters for full-text search rather than string literals when arguments allow query parameter value.

## Index selection

Spanner typically selects the most efficient index for a query using cost-based modeling. However, the `FORCE_INDEX` hint explicitly instructs Spanner to use a specific search index. For example, the following shows how to force Spanner to use the `AlbumsIndex` :

### GoogleSQL

    SELECT AlbumId
    FROM Albums @{FORCE_INDEX=AlbumsIndex}
    WHERE SEARCH(AlbumTitle_Tokens, "fifth symphony")

### PostgreSQL

    SELECT albumid
    FROM albums/*@force_index=albumsindex*/
    WHERE spanner.search(albumtitle_tokens, 'fifth symphony')

If the specified search index isn't [eligible](https://docs.cloud.google.com/spanner/docs/full-text-search/query-overview#sql-query-requirements) , the query fails, even if there are other eligible search indexes.

## Snippets in search results

A snippet is a piece of text extracted from a given string that gives users a sense of what a search result contains, and the reason why the result is relevant to their query.

For example, Gmail uses snippets to indicate the portion of an email that matches the search query:

![List of snippets](https://docs.cloud.google.com/static/spanner/docs/images/snippet.png)

Having the database generate a snippet has several benefits:

1.  **Convenience** : You don't need to implement logic to generate snippets from a search query.
2.  **Efficiency** : Snippets reduce the output size from the server.

The [`SNIPPET`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#snippet) function creates the snippet. It returns the relevant portion of the original string value along with positions of characters to highlight. The client can then choose how to display the snippet to the end user (for example, using highlighted or bold text).

For example, the following uses `SNIPPET` to retrieve text from `AlbumTitle` :

### GoogleSQL

    SELECT AlbumId, SNIPPET(AlbumTitle, "Fast Car")
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, "Fast Car")

### PostgreSQL

This example uses [spanner.snippet](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions-and-operators#spannersnippet) .

    SELECT albumid, spanner.snippet(albumtitle, 'Fast Car')
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'Fast Car')

## What's next

  - Learn how to [rank search results](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [perform a substring search](https://docs.cloud.google.com/spanner/docs/full-text-search/substring-search) .
  - Learn how to [paginate search results](https://docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [mix full-text and non-text queries](https://docs.cloud.google.com/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
  - Learn how to [search multiple columns](https://docs.cloud.google.com/spanner/docs/full-text-search/search-multiple-columns) .
