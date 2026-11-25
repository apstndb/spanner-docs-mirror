---
name: documents/docs.cloud.google.com/spanner/docs/full-text-search/partition-search-index
uri: https://docs.cloud.google.com/spanner/docs/full-text-search/partition-search-index
title: Partition search indexes
description: Partition your full-text search index in Spanner to improve search performance and scalability for large datasets.
data_source: docs.cloud.google.com
---

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

Spanner supports both unpartitioned and partitioned [search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) . This page describes how to create a partitioned search index in Spanner.

## Overview

An unpartitioned index is created when the `PARTITION BY` clause is omitted in the index definition. In an unpartitioned index, a query needs to read from all the index splits. This limits the potential scalability of full-text search queries.

Partitioned indexes, on the other hand, subdivide the index into smaller units, one for each unique partition. Queries can only search within a single partition at a time, specified by an equality condition in the `WHERE` clause. Queries against partitioned indexes are generally more efficient than queries against unpartitioned indexes because Spanner only needs to read data for a single partition. Partitioning the search index is analogous to the key prefix of a secondary index.

For example, suppose there are 1,000,000 `SingerIds` in a database and the following two indexes:

### GoogleSQL

```
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

```
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

To use a partitioned search index, a search query must specify `WHERE` conditions on all `PARTITION BY` columns. The following query selects the `AlbumsIndexBySingerId` index because it only searches data for a single singer:

### GoogleSQL

```
SELECT AlbumId
FROM Albums
WHERE SingerId = "singer1"
AND SEARCH(AlbumTitle_Tokens, 'happy')
```

### PostgreSQL

```
SELECT albumid
FROM albums
WHERE singerid = 'singer1'
AND spanner.search(albumtitle_tokens, 'happy')
```

It's also possible to [force](https://docs.cloud.google.com/spanner/docs/full-text-search/query-overview#index_selection) a query to use `AlbumsUnpartitionedIndex` to return the same results. However, it uses more resources, because the query needs to access all index splits and filter through all albums for all singers to find the token "happy", rather than just the splits corresponding to singer `singer1` .

However, there are times when the application needs to search through all of the albums rather than the albums for a specific singer. In these cases, you must use an unpartitioned index:

### GoogleSQL

```
SELECT AlbumId
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, 'piano concerto 1')
```

### PostgreSQL

```
SELECT albumid
FROM albums
WHERE spanner.search(albumtitle_tokens, 'piano concerto 1')
```

The general recommendation is to use the finest granularity of partitioning that's practical and appropriate for the query. For example, if the application queries an email mailbox where each query is restricted to a specific mailbox, partition the search index on the mailbox ID. However, if the query needs to search through all mailboxes, an unpartitioned index is a better fit.

Certain applications might require multiple partitioning strategies to accommodate their specific search requirements. For example, an inventory management system might need to support queries filtered by product type or manufacturer. Additionally, some applications might need multiple presorts, such as sorting by creation or modification time. In these scenarios, it's recommended that you create multiple search indexes, each optimized for the corresponding queries. The Spanner [query optimizer](https://docs.cloud.google.com/spanner/docs/query-optimizer/overview) automatically selects an index for each query.

## Partition by scalar values

You can partition a search index by column values in the base table. In the [Overview](https://docs.cloud.google.com/spanner/docs/full-text-search/partition-search-index#overview) example, the search index `AlbumsIndexBySingerId` is partitioned by the `SingerId` column. You can also partition a search index by multiple columns:

### GoogleSQL

```
CREATE SEARCH INDEX AlbumsIndexBySingerAndRelease
ON Albums(AlbumTitle_Tokens)
PARTITION BY SingerId, ReleaseTimestamp;
```

### PostgreSQL

```
CREATE SEARCH INDEX albumsindexbysingerandrelease
ON albums(albumtitle_tokens)
PARTITION BY singerid, releasetimestamp;
```

## Partition by repeated values

> **Note:** The examples in this section are intended for GoogleSQL. PostgreSQL doesn't support search indexes partitioned by repeated values.

In addition to partitioning by scalar values, Spanner also supports search indexes that are partitioned by repeated values, such as an array of scalar values.

To create a search index partitioned by repeated values, you must specify the `UNNEST` clause in the [`CREATE SEARCH INDEX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create-search-index) statement. The `UNNEST` clause indicates that the search index is partitioned by each element in the repeated value, rather than the repeated value itself.

Like indexes partitioned by scalar values, a query must have `WHERE` conditions that specify values for all `PARTITION BY` columns. You can specify the conditions using the [`ARRAY_INCLUDES`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes) , [`ARRAY_INCLUDES_ANY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes_any) , or [`ARRAY_INCLUDES_ALL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes_all) functions in GoogleSQL.

Using the GoogleSQL [`ARRAY_INCLUDES_ANY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_includes_any) function might result in searching across multiple partitions, which might not yield the best performance.

### Partition by array columns

You can partition a search index by an array column. For example, consider the following `Albums` table schema with the `Tracks` array column:

```
  CREATE TABLE Albums (
    AlbumId STRING(MAX) NOT NULL,
    SingerId STRING(MAX) NOT NULL,
    ReleaseTimestamp INT64 NOT NULL,
    AlbumTitle STRING(MAX),
    AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN,
    Tracks ARRAY<STRING(MAX)>
  ) PRIMARY KEY(SingerId, AlbumId);
```

You can create a search index on `AlbumTitle_Tokens` partitioned by the `Tracks` column:

```
  CREATE SEARCH INDEX AlbumsByTracks
  ON Albums(AlbumTitle_Tokens)
  UNNEST (Tracks) AS Track
  PARTITION BY Track;
```

The following query uses the `AlbumsByTracks` search index:

```
  SELECT AlbumId
  FROM Albums
  WHERE ARRAY_INCLUDES(Tracks, 'first')
  AND SEARCH(AlbumTitle_Tokens, 'happy')
```

### Partition by repeated proto fields

In GoogleSQL, you can use [generated columns](https://docs.cloud.google.com/spanner/docs/generated-column/how-to) and the [`ARRAY_TRANSFORM`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_transform) function to partition a search index by repeated fields in protocol buffers stored in `PROTO` columns.

For example, suppose you have the following definition of the `googlesql.example.SingerInfo` protocol buffer:

```
package googlesql.example;

message SingerInfo {
  optional string    nationality = 1;
  repeated Residence residence   = 2;

  message Residence {
    required int64  start_year   = 1;
    optional int64  end_year     = 2;
    optional string city         = 3;
    optional string country      = 4;
  }
}
```

To partition a search index on a `Singers` table by the `city` field of the repeated `residence` field in `SingerInfo` , first create a `STORED` generated column `ResidenceCities` using `ARRAY_TRANSFORM` :

```
CREATE PROTO BUNDLE (googlesql.example.SingerInfo, googlesql.example.SingerInfo.Residence);

CREATE TABLE Singers (
  SingerId INT64 NOT NULL,
  FirstName STRING(MAX),
  FirstName_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(FirstName)) HIDDEN,
  SingerInfo googlesql.example.SingerInfo,
  ResidenceCities ARRAY<STRING(MAX)> AS (ARRAY_TRANSFORM(SingerInfo.residence, r -> r.city)) STORED
) PRIMARY KEY (SingerId);
```

Then create a search index partitioned by `ResidenceCities` :

```
CREATE SEARCH INDEX SingersByResidenceCity
ON Singers(FirstName_Tokens)
UNNEST(ResidenceCities) AS City
PARTITION BY City;
```

The following query uses the `SingersByResidenceCity` search index:

```
SELECT SingerId, FirstName
FROM Singers
WHERE ARRAY_INCLUDES(ResidenceCities, 'Seattle')
AND SEARCH(FirstName_Tokens, 'John')
```

## What's next

- Learn about [tokenization and Spanner tokenizers](https://docs.cloud.google.com/spanner/docs/full-text-search/tokenization) .
- Learn about [search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) .
- Learn about [numeric indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/numeric-indexes) .
