**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

[Search indexes](/spanner/docs/full-text-search/search-indexes) can index multiple tokenized columns, making queries on these columns more efficient. This page describes how to perform a search on multiple columns, which is a type of [full-text search](/spanner/docs/full-text-search) .

## Perform a multi-column search

The [structure of the search index](/spanner/docs/full-text-search/search-indexes#search-indexes) ensures that queries don't need a distributed join, ensuring predictable performance of the queries. The distributed join is avoided due to the colocation of all tokens that correspond to a base table row on the same split.

For example, consider the following schema:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  Title STRING(MAX),
  Studio STRING(MAX),
  Title_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Title)) HIDDEN,
  Studio_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Studio)) HIDDEN
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex ON Albums(Title_Tokens, Studio_Tokens);
```

### PostgreSQL

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  title character varying,
  studio character varying,
  title_tokens spanner.tokenlist
      GENERATED ALWAYS AS (TOKENIZE_FULLTEXT(title)) VIRTUAL HIDDEN,
  studio_tokens spanner.tokenlist
      GENERATED ALWAYS AS (TOKENIZE_FULLTEXT(studio)) VIRTUAL HIDDEN,
) PRIMARY KEY(albumid);

CREATE SEARCH INDEX albumsindex ON albums(title_tokens, studio_tokens);
```

A query can now search two fields: `  Title_Tokens  ` and `  Studio_Tokens  ` .

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(Title_Tokens, "fifth symphony")
  AND SEARCH(Studio_Tokens, "Blue Note Studio")
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE spanner.search(title_tokens, 'fifth symphony')
  AND spanner.search(studio_tokens, 'Blue Note Studio')
```

Spanner supports multi-column search queries in conjunction, disjunction, and negation operators in the `  WHERE  ` clause. You can use all of the following types of queries with a search index:

  - **Conjunction** : Find documents where `  Title  ` has the term "car" and `  Studio  ` has the term "sun".
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(Title_Tokens, 'car') AND SEARCH(Studio_Tokens, 'sun')
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(title_tokens, 'car') AND spanner.search(studio_tokens, 'sun')
    ```

  - **Disjunction** : Find documents where either `  Title  ` has the term "car" or `  Studio  ` has the term "sun"
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(Title_Tokens, 'car') OR SEARCH(Studio_Tokens, 'sun')
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(title_tokens, 'car') OR spanner.search(studio_tokens, 'sun')
    ```

  - **Negation** : Find all documents where `  Title  ` doesn't contain the term "car".
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE NOT SEARCH(Title_Tokens, 'car')
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE NOT spanner.search(title_tokens, 'car')
    ```
    
    The [rquery language](/spanner/docs/full-text-search/query-overview#rquery) can perform the same type of searches:
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(Title_Tokens, '-car')
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(title_tokens, '-car')
    ```
    
    Both forms filter documents where `  Title  ` is NULL. Tokenization and search functions are defined to return NULL on NULL input. SQL defines NOT NULL as NULL.

Additionally, you can reference the same `  TOKENLIST  ` column multiple times.

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE (SEARCH(Title_Tokens, 'car') OR SEARCH(Studio_Tokens, 'sun'))
  AND (SEARCH(Title_Tokens, 'guy') OR SEARCH(Studio_Tokens, electric))
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE (spanner.search(title_tokens, 'car') OR spanner.search(studio_tokens, 'sun'))
  AND (spanner.search(title_tokens, 'guy') OR spanner.search(studio_tokens, 'electric'))
```

Use either the [rquery language](/spanner/docs/full-text-search/query-overview#rquery) or SQL to search for multiple terms in the same column. rquery is recommended due to its efficient query [caching](/spanner/docs/whitepapers/life-of-query#caching) for parameterized queries. Aside from the better query cache hit rate, the rquery and SQL languages have the same latency and performance rates.

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(Title_Tokens, 'car OR guy')

SELECT AlbumId
FROM Albums
WHERE SEARCH(Title_Tokens, 'car') OR SEARCH(Title_Tokens, 'guy')
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE spanner.search(title_tokens, 'car OR guy')

SELECT albumid
FROM albums
WHERE spanner.search(title_tokens, 'car') OR spanner.search(title_tokens, 'guy')
```

You can also use non-text conditions accelerated with search indexes in the same query with full-text search functions.

## What's next

  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
  - Learn how to [rank search results](/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [perform a substring search](/spanner/docs/full-text-search/substring-search) .
  - Learn how to [paginate search results](/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [mix full-text and non-text queries](/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
