---
name: documents/docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results
uri: https://docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results
title: Paginate search results
description: Paginate through full-text search results in Spanner efficiently using limit and offset or cursor-based pagination.
data_source: docs.cloud.google.com
---

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

Web applications often paginate data as it's presented to users. The end user receives one page of results, and when they navigate to the next page, the next batch of results is retrieved and presented. This page describes how to add pagination to search results when performing a [full-text search](https://docs.cloud.google.com/spanner/docs/full-text-search) in Spanner.

## Pagination options

Each page of a paginated result set is fetched by an independent query, and Spanner keeps no state between pages. Anything that changes between consecutive queries, such as the data or the sort order, can make the results inconsistent: when the application moves to the next page, rows can be skipped, or rows from the previous page can be repeated. This behavior applies to all paginated queries in Spanner, though the pagination method you choose determines how much these changes affect the results.

There are two ways to implement paginated queries in Spanner: *[key-based pagination](https://docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results#use_key-based_pagination)* (recommended) and *[offset-based pagination](https://docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results#use_offset-based_pagination)* .

Key-based pagination is a method for retrieving search results in smaller, more manageable chunks while ensuring consistent results across requests. A unique identifier (the "key") from the last result of a page is used as a reference point to fetch the next set of results.

Spanner generally recommends using key-based pagination, except when [paginating ranked search results](https://docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results#paginate_ranked_search_results) . While offset-based pagination is easier to implement, it has two significant drawbacks:

- **Higher query cost** : Offset-based pagination has to retrieve and discard the results of all preceding pages, leading to increased costs and decreased performance.
- **Inconsistent results** : In paginated queries, each page is typically retrieved at a different read timestamp. For example, the first page might come from a query at 1 PM, and the next from a query at 1:10 PM. Between queries, any row inserted or deleted before the current offset shifts the rows that follow it, so rows are skipped or repeated across pages.

Key-based pagination, on the other hand, uses a unique identifier (key) from the last result of a page to fetch the next set of results. This approach ensures efficient retrieval and more consistent results: the next page resumes from the last key of the previous page, so only concurrent writes that move rows across that boundary affect the results. For example, a newly inserted row that sorts before that key is skipped, while a previously returned row that is updated to sort after that key is repeated.

To provide stability in page results, the application could issue all queries for different pages at the same timestamp. This, however, might fail if the query exceeds the [version retention period](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) (the default is 1 hour). For example, this failure happens if `version_gc` is one hour, and the end user fetched the first results at 1 PM and clicked *Next* at 3 PM.

## Use key-based pagination

Key-based pagination remembers the last item of the previous page and uses it as a starting point for the next page query. To achieve this, the query must return the columns specified in the `ORDER BY` clause and limit the number of rows using `LIMIT` .

For key-based pagination to work, the query must order results by some strict total order. The easiest way to get one is to choose any [total order](https://en.wikipedia.org/wiki/Total_order) and then add tie-breaker columns, if needed. In most cases, the total order is the search index sort order and the unique combination of columns is the base table primary key.

Using our `Albums` sample schema, for the first page, the query looks like the following:

### GoogleSQL

```
SELECT AlbumId, ReleaseTimestamp
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, "fifth symphony")
ORDER BY ReleaseTimestamp DESC, AlbumId
LIMIT 10;
```

### PostgreSQL

```
SELECT albumid, releasetimestamp
FROM albums
WHERE spanner.search(albumtitle_tokens, 'fifth symphony')
ORDER BY releasetimestamp DESC, albumid
LIMIT 10;
```

The `AlbumId` is the tie breaker since `ReleaseTimestamp` isn't a key. There might be two different albums with the same value for `ReleaseTimestamp` .

To resume, the application runs the same query again, but with a `WHERE` clause that restricts the results from the previous page. The additional condition needs to account for key direction (ascending versus descending), tie breakers, and the order of NULL values for nullable columns.

In our example, `AlbumId` is the only key column (in ascending order) and it can't be NULL, so the condition is the following:

### GoogleSQL

```
SELECT AlbumId, ReleaseTimestamp
FROM Albums
WHERE (ReleaseTimestamp < @last_page_release_timestamp
    OR (ReleaseTimestamp = @last_page_release_timestamp
        AND AlbumId > @last_page_album_id))
    AND SEARCH(AlbumTitle_Tokens, @p)
ORDER BY ReleaseTimestamp DESC, AlbumId ASC
LIMIT @page_size;
```

### PostgreSQL

This example uses query parameters `$1` , `$2` , `$3` and `$4` which are bound to values specified for `last_page_release_timestamp` , `last_page_album_id` , `query` , and `page_size` , respectively.

```
SELECT albumid, releasetimestamp
FROM albums
WHERE (releasetimestamp < $1
    OR (releasetimestamp = $1
        AND albumid > $2))
    AND spanner.search(albumtitle_tokens, $3)
ORDER BY releasetimestamp DESC, albumid ASC
LIMIT $4;
```

Spanner interprets this kind of condition as [seekable](https://docs.cloud.google.com/spanner/docs/query-execution-operators#filter_scan) . This means that Spanner doesn't read the index for documents you're filtering out. This optimization is what makes key-based pagination much more efficient than offset-based pagination.

## Use offset-based pagination

Offset-based pagination leverages the `LIMIT` and `OFFSET` clauses of SQL query to simulate pages. The `LIMIT` value indicates the number of results per page. The `OFFSET` value is set to zero for the first page, page size for the second page, and double the page size for the third page.

For example, the following query fetches the third page, with a page size of 50:

### GoogleSQL

```
SELECT AlbumId
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, "fifth symphony")
ORDER BY ReleaseTimestamp DESC, AlbumId
LIMIT 50 OFFSET 100;
```

### PostgreSQL

```
SELECT albumid
FROM albums
WHERE spanner.search(albumtitle_tokens, 'fifth symphony')
ORDER BY releasetimestamp DESC, albumid
LIMIT 50 OFFSET 100;
```

Usage Notes:

- The `ORDER BY` clause is highly recommended to ensure consistent ordering between pages.
- In production queries, use query parameters rather than constants to specify `LIMIT` and `OFFSET` to make query caching more efficient. For more information, see [Query parameters](https://docs.cloud.google.com/spanner/docs/full-text-search/query-overview#query_parameters) .

## Paginate ranked search results

Spanner recommends offset-based pagination for [ranked queries](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search) , which order results by the value of the `SCORE` function rather than by key columns of the table. Although you can include the previous page's last score in the `WHERE` clause for the next page, that condition isn't seekable and therefore doesn't provide the cost and performance benefits of key-based pagination. In addition, `SCORE` values aren't comparable across scorer versions, so offset-based pagination produces fewer skipped or repeated rows when the scorer version changes.

To paginate ranked results, order by the score with the primary key as a tie breaker, and page with `LIMIT` and `OFFSET` . For example, the following query fetches the first page with a page size of two:

### GoogleSQL

```
SELECT
  AlbumId,
  SCORE(AlbumTitle_Tokens, @p) AS Score
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, @p)
ORDER BY Score DESC, AlbumId
LIMIT 2 OFFSET 0;
```

### PostgreSQL

This example uses query parameter `$1` , which is bound to the query string.

```
SELECT
  albumid,
  spanner.score(albumtitle_tokens, $1) AS score
FROM albums
WHERE spanner.search(albumtitle_tokens, $1)
ORDER BY score DESC, albumid
LIMIT 2 OFFSET 0;
```

The value of `SCORE` depends on the [scorer version](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search#scorer_versions) . Spanner uses the database's default scorer version unless the query sets one. A new scorer version can change the ordering of results, so if the scorer version changes between two pages of the same paginated query, the pages can be inconsistent. For example, suppose the matching albums have the following scores under two scorer versions:

| AlbumId | Score with version 1 | Score with version 2 |
|---------|----------------------|----------------------|
| 1       | 0.9                  | 0.7                  |
| 2       | 0.8                  | 0.95                 |
| 3       | 0.7                  | 0.6                  |
| 4       | 0.6                  | 0.85                 |

With version 1 and a page size of two, the first page returns albums 1 and 2. If the scorer version changes to version 2 before the second page is fetched, the second page is computed with `OFFSET 2` on the version 2 order, in which the albums are ranked 2, 4, 1, 3. As a result, the second page returns albums 1 and 3, so album 1 is repeated and album 4 is skipped.

Scorer version changes are infrequent: the database's default scorer version changes only when you update it or when Spanner migrates the database to a later default version, and only paginated queries that are in progress at that moment are affected. For most applications, this brief inconsistency is acceptable, and Spanner recommends using the default scorer version so that you always benefit from the latest scorer improvements.

If you need more control over the change, you have two options:

- **Database level** : Set the database's default scorer version yourself, as described in [Set the database's default scorer version](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search#set_the_databases_default_scorer_version) . Setting this option doesn't avoid the inconsistency, but it lets you choose when the switch happens—for example, during a low-traffic period—or stay on the current version until you're ready.
- **Query level** : Pin the scorer version in the query and carry it in your pagination state, as described in the following steps. This approach keeps in-progress paginated queries on the scorer version they started with, so the switch doesn't affect them. However, these queries remain subject to rows shifting because of concurrent writes, as with any offset-based pagination. This approach is more complex to implement and isn't necessary for most applications.

To pin the scorer version at the query level, complete the following steps:

1.  Set the scorer version explicitly in the `SCORE` function's `options` argument. Use the same query for every page, with the offset set to `0` for the first page:

    ### GoogleSQL

    ```
    SELECT
      AlbumId,
      SCORE(AlbumTitle_Tokens, @p,
            options=>JSON_OBJECT('version', @score_version)) AS Score
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, @p)
    ORDER BY Score DESC, AlbumId
    LIMIT @page_size OFFSET @offset;
    ```

    ### PostgreSQL

    This example uses query parameters `$1` , `$2` , `$3` , and `$4` , which are bound to the query string, the scorer options as a `JSONB` value such as `{"version": 2}` , the page size, and the offset, respectively.

    ```
    SELECT
      albumid,
      spanner.score(albumtitle_tokens, $1, options => $2) AS score
    FROM albums
    WHERE spanner.search(albumtitle_tokens, $1)
    ORDER BY score DESC, albumid
    LIMIT $3 OFFSET $4;
    ```

    For the first page, take the scorer version from your application's configuration; typically it's the same value as your database's default scorer version, so that paginated and other queries rank the same way.

2.  Store the scorer version you bound, together with the offset of the next page, in your pagination state, for example in a session cookie or in the opaque page token returned by your API.

3.  For every subsequent page, pass the stored scorer version in the query rather than reading the version from your application's current configuration. This approach keeps the paginated results consistent across scorer version changes, whether the database's default changes or you update your application's configuration.

The scorer version pinned in the query takes precedence over the database's default scorer version. To move pinned queries to a newer scorer version, change the value in your application's configuration.

## What's next

- Learn how to [rank search results](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search) .
- Learn how to [perform a substring search](https://docs.cloud.google.com/spanner/docs/full-text-search/substring-search) .
- Learn how to [mix full-text and non-text queries](https://docs.cloud.google.com/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
- Learn how to [search multiple columns](https://docs.cloud.google.com/spanner/docs/full-text-search/search-multiple-columns) .
