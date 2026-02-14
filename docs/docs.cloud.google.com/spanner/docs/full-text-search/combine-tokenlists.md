**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to concatenate `  TOKENLIST  ` s in either a [search index](/spanner/docs/full-text-search) when you set up your schema or in a search query when performing a full-text search in Spanner.

## Combine TOKENLISTs in a search index

Sometimes, you need your application to search across individual fields. At other times, the application needs to search across all fields. For example, in a table with two string columns, you might want your application to search across both columns without differentiating which column the matches come from.

In Spanner, there are two ways to achieve this:

1.  [Tokenize words separately and concatenate the resulting `  TOKENLIST  ` s (recommended).](#tokenize-separately)
2.  [Concatenate strings and tokenize the result.](#concatenate-strings)

With the second approach, there are two problems:

1.  If you want to index `  Title  ` or `  Studio  ` individually, in addition to indexing them in a combined `  TOKENLIST  ` , the same text is tokenized twice. This causes transactions to use more resources.
2.  A phrase search spans both fields. For example, if `  @p  ` is set to `  "Blue Note"  ` , it matches a row that contains both `  Title  ` ="Big Blue Note" and `  Studio  ` ="Blue Note Studios".

The first approach solves these problems because a phrase only matches one field and each string field is only tokenized once if both the individual and combined `  TOKENLIST  ` s are indexed. Even though each string field is only tokenized once, the resulting `  TOKENLIST  ` s are stored separately in the index.

### Tokenize words separately and concatenate `     TOKENLIST    ` s

The following example tokenizes each word and uses [`  TOKENLIST_CONCAT  `](/spanner/docs/reference/standard-sql/search_functions#tokenlist_concat) to concatenate the `  TOKENLIST  ` s:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  Title STRING(MAX),
  Studio STRING(MAX),
  Title_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Title)) HIDDEN,
  Studio_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Studio)) HIDDEN,
  Combined_Tokens TOKENLIST AS (TOKENLIST_CONCAT([Title_Tokens, Studio_Tokens])) HIDDEN,
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex ON Albums(Combined_Tokens);

SELECT AlbumId FROM Albums WHERE SEARCH(Combined_Tokens, @p);
```

### PostgreSQL

PostgreSQL uses [`  spanner.tokenlist_concat  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) for concatenation. The query parameter `  $1  ` is bound to 'Hatel Kaliphorn'.

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  title character varying,
  studio character varying,
  title_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(title)) VIRTUAL HIDDEN,
  studio_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(studio)) VIRTUAL HIDDEN,
  combined_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenlist_concat(ARRAY[spanner.tokenize_fulltext(title), spanner.tokenize_fulltext(studio)])) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsindex ON albums(combined_tokens);

SELECT albumid FROM albums WHERE spanner.search(combined_tokens, $1);
```

Note that `  tokenlist_concat  ` doesn't call `  title_tokens  ` or `  studio_tokens  ` , but instead calls `  spanner.tokenize_fulltext(title)  ` and `  spanner.tokenize_fulltext(studio)  ` . This is because PostgreSQL doesn't support referencing generated columns that are within other generated columns. `  spanner.tokenlist_concat  ` needs to call tokenize functions and not reference tokenlist columns directly.

`  TOKENLIST  ` concatenation can also be implemented entirely on the query side. For more information, see [Query-side `  TOKENLIST  ` concatenation](#concatenate-tokenlists-query) .

`  TOKENLIST_CONCAT  ` is supported for both full-text and [substring](/spanner/docs/full-text-search/substring-search) searches. Spanner doesn't let you mix tokenization types, such as `  TOKENIZE_FULLTEXT  ` and `  TOKENIZE_SUBSTRING  ` in the same `  TOKENLIST_CONCAT  ` call.

In GoogleSQL, the definition of text `  TOKENLIST  ` columns can be changed in non-stored columns to add additional columns. This is useful when you want to add an additional column to `  TOKENLIST_CONCAT  ` . Changing the generated column expression doesn't backfill existing rows in the index.

### Concatenate strings and tokenize the result

The following example concatenates strings and tokenizes the result:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  Title STRING(MAX),
  Studio STRING(MAX),
  Combined_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Title || " " || Studio)) HIDDEN,
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex ON Albums(Combined_Tokens);

SELECT AlbumId FROM Albums WHERE SEARCH(Combined_Tokens, @p);
```

### PostgreSQL

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  title character varying,
  studio character varying,
  combined_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(title || ' ' || studio)) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsindex ON albums(combined_tokens);

SELECT albumid FROM albums WHERE spanner.search(combined_tokens, $1);
```

## Query-side `     TOKENLIST    ` concatenation

The tradeoff with indexing the concatenated `  TOKENLIST  ` is that it increases storage and write cost. Each token is now stored on the disk twice: once in a posting list of its original `  TOKENLIST  ` , and once in a posting list of the combined `  TOKENLIST  ` . Query-side concatenation of `  TOKENLIST  ` columns avoids this cost but the query uses more compute resources.

To concatenate multiple `  TOKENLIST  ` s, use the [`  TOKENLIST_CONCAT  `](/spanner/docs/reference/standard-sql/search_functions#tokenlist_concat) function in the [`  SEARCH  `](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) query. For this section, we're using the following sample schema:

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  Title STRING(MAX),
  Studio STRING(MAX),
  Title_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Title)) HIDDEN,
  Studio_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(Studio)) HIDDEN,
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex ON Albums(Title_Tokens, Studio_Tokens);
```

### PostgreSQL

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  title character varying,
  studio character varying,
  title_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(title)) VIRTUAL HIDDEN,
  studio_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(studio)) VIRTUAL HIDDEN,
 PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsindex ON albums(title_tokens, studio_tokens);
```

The following query searches for rows that have the tokens "blue" and "note" anywhere in the `  Title  ` and `  Studio  ` columns. This includes rows with both "blue" and "note" in the `  Title  ` column, "blue" and "note" in the `  Studio  ` column, and "blue" in the `  Title  ` column and "note" in the `  Studio  ` column, or the opposite.

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH(TOKENLIST_CONCAT([AlbumTitle_Tokens, Studio_Tokens]), 'blue note')
```

### PostgreSQL

This example uses [`  spanner.search  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) with [`  spanner.tokenlist_concat  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .

``` text
SELECT albumid
FROM albums
WHERE spanner.search(spanner.tokenlist_concat(ARRAY[albumtitle_tokens, studio_tokens]), 'blue note')
```

Write-side and query-side `  TOKENLIST  ` concatenation produce identical results. The choice between the two is a trade-off between disk cost and query cost.

Alternatively, an application could search multiple `  TOKENLIST  ` columns and use `  OR  ` along with the `  SEARCH  ` function:

### GoogleSQL

``` text
SEARCH(AlbumTitle_Tokens, 'Blue Note') OR SEARCH(Studio_Tokens, 'Blue Note')
```

### PostgreSQL

``` text
spanner.search(albumtitle_tokens, 'Blue Note') OR spanner.search(studio_tokens, 'Blue Note')
```

This, however, has different semantics. It doesn't match albums where `  AlbumTitle_Tokens  ` has "blue", but not "note" and `  Studio_Tokens  ` has "note", but not "blue".

## What's next

  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
