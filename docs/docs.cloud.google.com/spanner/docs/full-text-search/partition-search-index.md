**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Spanner supports both unpartitioned and partitioned [search indexes](/spanner/docs/full-text-search/search-indexes) . This page describes how to create a partitioned search index in Spanner.

An unpartitioned index is created when the `  PARTITION BY  ` clause is omitted in the index definition. In an unpartitioned index, a query needs to read from all the index splits. This limits the potential scalability of full-text search queries.

Partitioned indexes, on the other hand, subdivide the index into smaller units, one for each unique partition. Queries can only search within a single partition at a time, specified by an equality condition in the `  WHERE  ` clause. Queries against partitioned indexes are generally more efficient than queries against unpartitioned indexes because Spanner only needs to read data for a single partition. Partitioning the search index is analogous to the key prefix of a secondary index.

For example, suppose there are 1,000,000 `  SingerIds  ` in a database and the following two indexes:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  SingerId STRING(MAX) NOT NULL,
  ReleaseTimestamp INT64 NOT NULL,
  AlbumTitle STRING(MAX),
  AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN,
  SingerId_Tokens TOKENLIST AS (TOKEN(SingerId)) HIDDEN
) PRIMARY KEY(SingerId, AlbumId);

CREATE SEARCH INDEX AlbumsUnpartitionedIndex
ON Albums(AlbumTitle_Tokens, SingerId_Tokens);

CREATE SEARCH INDEX AlbumsIndexBySingerId
ON Albums(AlbumTitle_Tokens)
PARTITION BY SingerId;
```

### PostgreSQL

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  singerid character varying NOT NULL,
  releasetimestamp bigint NOT NULL,
  albumtitle character varying,
  albumtitle_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
  singerid_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.token(singerid)) VIRTUAL HIDDEN,
PRIMARY KEY(singerid, albumid));

CREATE SEARCH INDEX albumsunpartitionedindex
ON albums(albumtitle_tokens, singerid_tokens);

CREATE SEARCH INDEX albumsindexbysingerid
ON albums(albumtitle_tokens)
PARTITION BY singerid;
```

The following query selects the `  AlbumsIndexBySingerId  ` index because it only searches data for a single singer. This type of query typically uses fewer resources.

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SingerId = "singer1"
AND SEARCH(AlbumTitle_Tokens, 'happy')
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE singerid = 'singer1'
AND spanner.search(albumtitle_tokens, 'happy')
```

It's also possible to [force](/spanner/docs/full-text-search/query-overview#index_selection) a query to use `  AlbumsUnpartitionedIndex  ` to return the same results. However, it uses more resources, because the query needs to access all index splits and filter through all albums for all singers to find the token "happy", rather than just the splits corresponding to singer `  singer1  ` .

However, there are times when the application needs to search through all of the albums rather than the albums for a specific singer. In these cases, you must use an unpartitioned index:

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, 'piano concerto 1')
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE spanner.search(albumtitle_tokens, 'piano concerto 1')
```

The general recommendation is to use the finest granularity of partitioning that's practical and appropriate for the query. For example, if the application queries an email mailbox where each query is restricted to a specific mailbox, partition the search index on the mailbox ID. However, if the query needs to search through all mailboxes, an unpartitioned index is a better fit.

Certain applications might require multiple partitioning strategies to accommodate their specific search requirements. For example, an inventory management system might need to support queries filtered by product type or manufacturer. Additionally, some applications might need multiple presorts, such as sorting by creation or modification time. In these scenarios, it's recommended that you create multiple search indexes, each optimized for the corresponding queries. The Spanner [query optimizer](/spanner/docs/query-optimizer/overview) automatically selects an index for each query.

## What's next

  - Learn about [tokenization and Spanner tokenizers](/spanner/docs/full-text-search/tokenization) .
  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
  - Learn about [numeric indexes](/spanner/docs/full-text-search/numeric-indexes) .
