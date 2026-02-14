**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Many applications query a database to populate a single page in their applications. In such applications, the application doesn't need all of the matches, but only the top-k matches based on index sort order. Search indexes can implement this type of search very efficiently. This page describes how to create and search an index that has top-k matching.

## Create search indexes for top-k matches

To configure a search index for top-k matching, use `  ORDER BY  ` to order the search index by a specific column. Queries need to have an `  ORDER BY  ` clause that exactly matches the search index sort order (including ascending versus descending direction) and a `  LIMIT  ` clause that requests the query to stop after finding k-matching rows.

You can also implement pagination using these clauses. For more information, see [Paginate search queries](/spanner/docs/full-text-search/paginate-search-results) .

For some use cases, it might make sense to maintain multiple search indexes sorted by different columns. Like [partitioning](/spanner/docs/full-text-search/partition-search-index) , it's a trade-off between storage and write cost versus query latency.

For example, consider a table that uses the following schema:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  RecordTimestamp INT64 NOT NULL,
  ReleaseTimestamp INT64 NOT NULL,
  ListenTimestamp INT64 NOT NULL,
  AlbumTitle STRING(MAX),
  AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsRecordTimestampIndex
ON Albums(AlbumTitle_Tokens, SingerId_Tokens)
STORING (ListenTimestamp)
ORDER BY RecordTimestamp DESC

CREATE SEARCH INDEX AlbumsReleaseTimestampIndex
ON Albums(AlbumTitle_Tokens)
STORING (ListenTimestamp)
ORDER BY ReleaseTimestamp DESC
```

### PostgreSQL

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  recordtimestamp bigint NOT NULL,
  releasetimestamp bigint NOT NULL,
  listentimestamp bigint NOT NULL,
  albumtitle character varying,
  albumtitle_tokens spanner.tokenlist
      GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsrecordtimestampindex
ON Albums(albumtitle_tokens, singerid_tokens)
INCLUDE (listentimestamp)
ORDER BY recordtimestamp DESC

CREATE SEARCH INDEX albumsreleasetimestampindex
ON Albums(albumtitle_tokens)
INCLUDE (listentimestamp)
ORDER BY releasetimestamp DESC
```

## Query search indexes for top-k matches

As stated previously, queries need to have an `  ORDER BY  ` clause that exactly matches the search index sort order (including ascending versus descending direction) and a `  LIMIT  ` clause that requests the query to stop after finding k-matching rows.

The following list analyzes the efficiency of some common queries.

  - This query is very efficient. It selects the `  AlbumsRecordTimestampIndex  ` index. Even if there are many albums with the word "happy", the query only scans a small number of rows:
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, 'happy')
    ORDER BY RecordTimestamp DESC
    LIMIT 10
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'happy')
    ORDER BY recordtimestamp DESC
    LIMIT 10
    ```

  - The same query, requesting sort order by `  ReleaseTimestamp  ` in descending order, uses the `  AlbumsReleaseTimestampIndex  ` index and is equally efficient:
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, 'happy')
    ORDER BY ReleaseTimestamp DESC
    LIMIT 10
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'happy')
    ORDER BY releasetimestamp DESC
    LIMIT 10
    ```

  - A query that requests sort order by `  ListenTimestamp  ` doesn't execute a top-k query efficiently. It has to fetch all matching albums, sort them by `  ListenTimestamp,  ` and return the top 10. Such a query uses more resources if there's a large number of documents that contain the term "happy".
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, 'happy')
    ORDER BY ListenTimestamp DESC
    LIMIT 10
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'happy')
    ORDER BY listentimestamp DESC
    LIMIT 10
    ```

  - Similarly, a query doesn't run efficiently if it requests that results are ordered using the `  RecordTimestamp  ` column in ascending order. It scans all rows with the word "happy", despite having a `  LIMIT  ` .
    
    ### GoogleSQL
    
    ``` text
    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, 'happy')
    ORDER BY RecordTimestamp ASC
    LIMIT 10
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'happy')
    ORDER BY recordtimestamp ASC
    LIMIT 10
    ```

## What's next

  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
  - Learn how to [rank search results](/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [paginate search results](/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [mix full-text and non-text queries](/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
  - Learn how to [search multiple columns](/spanner/docs/full-text-search/search-multiple-columns) .
