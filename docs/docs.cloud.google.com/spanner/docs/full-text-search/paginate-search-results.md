**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Web applications often paginate data as it's presented to users. The end user receives one page of results, and when they navigate to the next page, the next batch of results is retrieved and presented. This page describes how to add pagination to search results when performing a [full-text search](/spanner/docs/full-text-search) in Spanner.

## Pagination options

There are two ways to implement paginated queries in Spanner: *[key-based pagination](#use_key-based_pagination)* (recommended) and *[offset-based pagination](#use_offset-based_pagination)* .

Key-based pagination is a method for retrieving search results in smaller, more manageable chunks while ensuring consistent results across requests. A unique identifier (the "key") from the last result of a page is used as a reference point to fetch the next set of results.

Spanner generally recommends using key-based pagination. While offset-based pagination is easier to implement, it has two significant drawbacks:

1.  **Higher query cost** :Offset-based pagination repeatedly retrieves and discards the same results, leading to increased costs and decreased performance.
2.  **Inconsistent Results:** In paginated queries, each page is typically retrieved at a different read timestamp. For example, the first page might come from a query at 1 PM, and the next from a query at 1:10 PM. This means that search results can change between queries, leading to inconsistent results across pages.

Key-based pagination, on the other hand, uses a unique identifier (key) from the last result of a page to fetch the next set of results. This ensures both efficient retrieval and consistent results, even if the underlying data changes.

To provide stability in page results, the application could issue all queries for different pages at the same timestamp. This, however, might fail if the query exceeds the [version retention period](/spanner/docs/reference/rest/v1/projects.instances.databases#Database.FIELDS.version_retention_period) (the default is 1 hour). For example, this failure happens if `  version_gc  ` is one hour, and the end user fetched the first results at 1 PM and clicked *Next* at 3 PM.

## Use key-based pagination

Key-based pagination remembers the last item of the previous page and uses it as a starting point for the next page query. To achieve this, the query must return the columns specified in the `  ORDER BY  ` clause and limit the number of rows using `  LIMIT  ` .

For key-based pagination to work, the query must order results by some strict total order. The easiest way to get one is to choose any [total order](https://en.wikipedia.org/wiki/Total_order) and then add tie-breaker columns, if needed. In most cases, the total order is the search index sort order and the unique combination of columns is the base table primary key.

Using our `  Albums  ` sample schema, for the first page, the query looks like the following:

### GoogleSQL

``` text
SELECT AlbumId, ReleaseTimestamp
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, "fifth symphony")
ORDER BY ReleaseTimestamp DESC, AlbumId
LIMIT 10;
```

### PostgreSQL

``` text
SELECT albumid, releasetimestamp
FROM albums
WHERE spanner.search(albumtitle_tokens, 'fifth symphony')
ORDER BY releasetimestamp DESC, albumid
LIMIT 10;
```

The `  AlbumId  ` is the tie breaker since `  ReleaseTimestamp  ` isn't a key. There might be two different albums with the same value for `  ReleaseTimestamp  ` .

To resume, the application runs the same query again, but with a `  WHERE  ` clause that restricts the results from the previous page. The additional condition needs to account for key direction (ascending versus descending), tie breakers, and the order of NULL values for nullable columns.

In our example, `  AlbumId  ` is the only key column (in ascending order) and it can't be NULL, so the condition is the following:

### GoogleSQL

``` text
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

This example uses query parameters `  $1  ` , `  $2  ` , `  $3  ` and `  $4  ` which are bound to values specified for `  last_page_release_timestamp  ` , `  last_page_album_id  ` , `  query  ` , and `  page_size  ` , respectively.

``` text
SELECT albumid, releasetimestamp
FROM albums
WHERE (releasetimestamp < $1
    OR (releasetimestamp = $1
        AND albumid > $2))
    AND spanner.search(albumtitle_tokens, $3)
ORDER BY releasetimestamp DESC, albumid ASC
LIMIT $4;
```

Spanner interprets this kind of condition as [seekable](/spanner/docs/query-execution-operators#filter_scan) . This means that Spanner doesn't read the index for documents you're filtering out. This optimization is what makes key-based pagination much more efficient than offset-based pagination.

## Use offset-based pagination

Offset-based pagination leverages the `  LIMIT  ` and `  OFFSET  ` clauses of SQL query to simulate pages. The `  LIMIT  ` value indicates the number of results per page. The `  OFFSET  ` value is set to zero for the first page, page size for the second page, and double the page size for the third page.

For example, the following query fetches the third page, with a page size of 50:

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(AlbumTitle_Tokens, "fifth symphony")
ORDER BY ReleaseTimestamp DESC, AlbumId
LIMIT 50 OFFSET 100;
```

### PostgreSQL

``` text
SELECT albumid
FROM albums
WHERE spanner.search(albumtitle_tokens, 'fifth symphony')
ORDER BY releasetimestamp DESC, albumid
LIMIT 50 OFFSET 100;
```

Usage Notes:

  - The `  ORDER BY  ` clause is highly recommended to ensure consistent ordering between pages.
  - In production queries, use query parameters rather than constants to specify `  LIMIT  ` and `  OFFSET  ` to make query caching more efficient. For more information, see [Query parameters](/spanner/docs/full-text-search/query-overview#query_parameters) .

## What's next

  - Learn how to [rank search results](/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [perform a substring search](/spanner/docs/full-text-search/substring-search) .
  - Learn how to [mix full-text and non-text queries](/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
  - Learn how to [search multiple columns](/spanner/docs/full-text-search/search-multiple-columns) .
