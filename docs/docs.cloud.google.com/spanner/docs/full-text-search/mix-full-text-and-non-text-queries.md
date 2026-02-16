**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to perform a search that mixes full-text and non-text data.

## Perform a mixed full-text and non-text search

[Search indexes](/spanner/docs/full-text-search/search-indexes) support full-text, exact match, numeric columns, and JSON/JSONB columns. You can combine text and non-text conditions in the `  WHERE  ` clause similarly to multi-column search queries. The query optimizer tries to optimize non-text predicates with a search index. If that's not possible, Spanner evaluates the condition for every row that matches the search index. Referenced columns not stored in the search index are fetched from the base table.

Consider the following example:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  Title STRING(MAX),
  Rating FLOAT64,
  Genres ARRAY<STRING(MAX)>,
  Likes INT64,
  Cover BYTES(MAX),
  Title_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Title)) HIDDEN,
  Rating_Tokens TOKENLIST AS (TOKENIZE_NUMBER(Rating)) HIDDEN,
  Genres_Tokens TOKENLIST AS (TOKEN(Genres)) HIDDEN
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex
ON Albums(Title_Tokens, Rating_Tokens, Genres_Tokens)
STORING (Likes);
```

### PostgreSQL

Spanner PostgreSQL support has the following limitations:

  - `  spanner.tokenize_number  ` function only supports the `  bigint  ` type.
  - `  spanner.token  ` doesn't support tokenizing arrays.

<!-- end list -->

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  title character varying,
  rating bigint,
  genres character varying NOT NULL,
  likes bigint,
  cover bytea,
  title_tokens spanner.tokenlist AS (spanner.tokenize_fulltext(title)) VIRTUAL HIDDEN,
  rating_tokens spanner.tokenlist AS (spanner.tokenize_number(rating)) VIRTUAL HIDDEN,
  genres_tokens spanner.tokenlist AS (spanner.token(genres)) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsindex
ON albums(title_tokens, rating_tokens, genres_tokens)
INCLUDE (likes);
```

The behavior of queries on this table include the following:

  - `  Rating  ` and `  Genres  ` are included in the search index. Spanner accelerates conditions using search index posting lists. `  ARRAY_INCLUDES_ANY  ` , `  ARRAY_INCLUDES_ALL  ` are GoogleSQL functions and are not supported for PostgreSQL dialect.
    
    ``` text
    SELECT Album
    FROM Albums
    WHERE Rating > 4
      AND ARRAY_INCLUDES_ANY(Genres, ['jazz'])
    ```

  - The query can combine conjunctions, disjunctions, and negations in any way, including mixing full-text and non-text predicates. This query is fully accelerated by the search index.
    
    ``` text
    SELECT Album
    FROM Albums
    WHERE (SEARCH(Title_Tokens, 'car')
          OR Rating > 4)
      AND NOT ARRAY_INCLUDES_ANY(Genres, ['jazz'])
    ```

  - `  Likes  ` is stored in the index, but the schema doesn't request Spanner to build a token index for its possible values. Therefore, the full-text predicate on `  Title  ` and non-text predicate on `  Rating  ` is accelerated, but the predicate on `  Likes  ` isn't. In Spanner, the query fetches all documents with the term "car" in the `  Title  ` and a rating more than 4, then it filters documents that don't have at least 1000 likes. This query uses a lot of resources if almost all albums have the term "car" in their title and almost all of them have a rating of 5, but few albums have 1000 likes. In such cases, indexing `  Likes  ` similarly to `  Rating  ` saves resources.
    
    ### GoogleSQL
    
    ``` text
    SELECT Album
    FROM Albums
    WHERE SEARCH(Title_Tokens, 'car')
      AND Rating > 4
      AND Likes >= 1000
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT album
    FROM albums
    WHERE spanner.search(title_tokens, 'car')
      AND rating > 4
      AND likes >= 1000
    ```

  - `  Cover  ` isn't stored in the index. The following query does a [back join](/spanner/docs/query-execution-plans#index_and_back_join_queries) between `  AlbumsIndex  ` and `  Albums  ` to fetch `  Cover  ` for all matching albums.
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId, Cover
    FROM Albums
    WHERE SEARCH(Title_Tokens, 'car')
      AND Rating > 4
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid, cover
    FROM albums
    WHERE spanner.search(title_tokens, 'car')
      AND rating > 4
    ```

## What's next

  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
  - Learn how to [rank search results](/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [perform a substring search](/spanner/docs/full-text-search/substring-search) .
  - Learn how to [paginate search results](/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [search multiple columns](/spanner/docs/full-text-search/search-multiple-columns) .
