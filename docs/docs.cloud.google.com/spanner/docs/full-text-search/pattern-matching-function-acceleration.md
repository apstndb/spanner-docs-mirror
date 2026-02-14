**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Spanner [search indexes](/spanner/docs/full-text-search/search-indexes) can accelerate pattern matching expressions such as [`  LIKE  `](/spanner/docs/reference/standard-sql/operators#like_operator) , [`  STARTS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#starts_with) , [`  ENDS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#ends_with) , and regular expression matching predicate [`  REGEXP_CONTAINS  `](/spanner/docs/reference/standard-sql/string_functions#regexp_contains) . This page describes how to create and configure a search index using [`  TOKENIZE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_ngrams) to accelerate pattern matching predicates.

## Configure an n-gram `     TOKENLIST    ` for pattern-matching acceleration

To enable pattern-matching expressions acceleration, tokenize a lower-cased `  STRING  ` column with [`  TOKENIZE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_ngrams) and store the `  STRING  ` column using the `  STORING  ` clause in GoogleSQL, or `  INCLUDE  ` clause in PostgreSQL.

### GoogleSQL

``` text
CREATE TABLE Albums (
AlbumId INT64 NOT NULL,
AlbumTitle STRING(MAX),
AlbumTitle_Ngram_Tokens TOKENLIST AS (
  TOKENIZE_NGRAMS(LOWER(AlbumTitle), ngram_size_min=>3, ngram_size_max=>4)) HIDDEN,
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex
ON Albums(AlbumTitle_Ngram_Tokens) STORING (AlbumTitle);
```

### PostgreSQL

``` text
CREATE TABLE albums (
albumid bigint NOT NULL,
album_title varchar,
album_title_ngrams_tokens spanner.tokenlist GENERATED ALWAYS AS (
  spanner.tokenize_ngrams(
    lower(album_title),
    ngram_size_min => 3,
    ngram_size_max => 4
  )
) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsidx ON
albums(album_title_ngrams_tokens) INCLUDE (album_title);
```

## Automatic acceleration of queries with pattern-matching predicates

The query optimizer might choose to accelerate the following queries using `  AlbumsIndex  ` with `  AlbumTitle_Ngram_Tokens  ` . Optionally, the query can provide `  @{force_index = AlbumsIndex}  ` to force the optimizer to use `  AlbumsIndex  ` .

### GoogleSQL

In GoogleSQL, we accelerate [`  LIKE  `](/spanner/docs/reference/standard-sql/operators#like_operator) , [`  STARTS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#starts_with) , [`  ENDS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#ends_with) , and [`  REGEXP_CONTAINS  `](/spanner/docs/reference/standard-sql/string_functions#regexp_contains) .

  - `  LIKE  ` predicate:
    
    ``` text
    SELECT AlbumId
    FROM Albums @{FORCE_INDEX=AlbumsIndex}
    WHERE AlbumTitle LIKE "%999%";
    ```

  - `  STARTS_WITH  ` predicate:
    
    ``` text
    SELECT AlbumId
    FROM Albums @{FORCE_INDEX=AlbumsIndex}
    WHERE STARTS_WITH(AlbumTitle, "apple")
    ```

  - `  ENDS_WITH  ` predicate:
    
    ``` text
    SELECT AlbumId
    FROM Albums @{FORCE_INDEX=AlbumsIndex}
    WHERE ENDS_WITH(AlbumTitle, "apple")
    ```

  - `  REGEXP_CONTAINS  ` predicate:
    
    ``` text
    SELECT AlbumId
    FROM Albums @{FORCE_INDEX=AlbumsIndex}
    WHERE REGEXP_CONTAINS(AlbumTitle, r"(good|great)[ ]+morning")
    ```

### PostgreSQL

In PostgreSQL, we accelerate [`  LIKE  `](/spanner/docs/reference/postgresql/functions#pattern-matching) and [`  STARTS_WITH  `](/spanner/docs/reference/postgresql/functions#string_functions) .

  - `  LIKE  ` predicate:
    
    ``` text
    SELECT albumid
    FROM albums /*@ FORCE_INDEX = albumsidx */
    WHERE album_title like '%999%';
    ```

  - `  STARTS_WITH  ` predicate:
    
    ``` text
    SELECT albumid
    FROM albums /*@ FORCE_INDEX = albumsidx */
    WHERE starts_with(album_title, 'apple')
    ```

## Prerequisites on acceleration

For Spanner to enable this acceleration, the following rules must be met:

  - The index must store the `  STRING  ` column using the `  STORING  ` clause in GoogleSQL, or `  INCLUDE  ` clause in PostgreSQL. This prevents costly back-joins to the base table during post-filtering, which is critical for performance when the search over-retrieves documents.
  - The `  STRING  ` column must be tokenized using [`  TOKENIZE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_ngrams) .
  - The tokenization must apply to `  LOWER(column_name)  ` rather than `  column_name  ` .
  - The [`  LIKE  `](/spanner/docs/reference/standard-sql/operators#like_operator) pattern, [`  STARTS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#starts_with) prefix, [`  ENDS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#ends_with) suffix, or [`  REGEXP_CONTAINS  `](/spanner/docs/reference/standard-sql/string_functions#regexp_contains) regular expression must be specified as a constant literal. [Query parameters](/spanner/docs/reference/standard-sql/lexical#query_parameters) are not supported to avoid acceleration on patterns that are too short.
  - The [`  LIKE  `](/spanner/docs/reference/standard-sql/operators#like_operator) pattern, [`  STARTS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#starts_with) prefix, [`  ENDS_WITH  `](/spanner/docs/reference/standard-sql/string_functions#ends_with) suffix, or [`  REGEXP_CONTAINS  `](/spanner/docs/reference/standard-sql/string_functions#regexp_contains) regular expression must contain enough text for at least one n-gram. For example `  r".*"  ` doesn't qualify because there's no sequence of characters to match. Similarly, if the ngram minimum size is set to 3, the [`  LIKE  `](/spanner/docs/reference/standard-sql/operators#like_operator) predicate `  "%ab%"  ` doesn't qualify because `  "ab"  ` (size 2) is too short.

## What's next

  - Learn about [finding approximate matches with fuzzy search](/spanner/docs/full-text-search/fuzzy-search) .
