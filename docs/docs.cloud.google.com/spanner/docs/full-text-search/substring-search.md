**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

In addition to full token matching, Spanner [search indexes](https://docs.cloud.google.com/spanner/docs/full-text-search/search-indexes) support substring searches. This page describes how to perform a substring search as part of a [full-text search](https://docs.cloud.google.com/spanner/docs/full-text-search) in Spanner.

Substring searches have the following characteristics:

  - Case insensitive, discards most punctuation, and normalizes whitespace.
  - No Chinese, Japanese, Korean (CJK) segmentation, since partial CJK queries often segment incorrectly.
  - For multiple search terms, the result must contain a substring from each term. For example, `'happ momen'` matches `"happy moment"` , because both substrings are found in the text. It doesn't match `"happy day"` .

**Examples**

| Stored text                | Substring query      | Match |
| -------------------------- | -------------------- | ----- |
| Bridge over Troubled Water | ridg roub            | Yes   |
| Bridge over Troubled Water | ridg , roub          | Yes   |
| Bridge over Troubled Water | over brid            | Yes   |
| Bridge over Troubled Water | ate bridge           | Yes   |
| Bridge over Troubled Water | Bridge bridge bridge | Yes   |
| Bridge over Troubled Water | bri trou ter         | Yes   |
| Bridge over Troubled Water | bri dge              | Yes   |
| Bridge over Troubled Water | troubledwater        | No    |
| Bridge over Troubled Water | trubled              | No    |

For a substring search, use the [`TOKENIZE_SUBSTRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) function in the `TOKENLIST` column definition, as shown in the following DDL example:

### GoogleSQL

    CREATE TABLE Albums (
    AlbumId STRING(MAX) NOT NULL,
    AlbumTitle STRING(MAX),
    AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_SUBSTRING(AlbumTitle)) HIDDEN
    ) PRIMARY KEY(AlbumId);
    
    CREATE SEARCH INDEX AlbumsTitleIndex ON Albums(AlbumTitle_Tokens);

### PostgreSQL

This example uses [`spanner.tokenize_substring`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#tokenize_substring) .

    CREATE TABLE albums (
    albumid character varying NOT NULL,
    albumtitle character varying,
    albumtitle_tokens spanner.tokenlist
        GENERATED ALWAYS AS (spanner.tokenize_substring(albumtitle)) VIRTUAL HIDDEN,
    PRIMARY KEY(albumid));
    
    CREATE SEARCH INDEX albumstitleindex ON albums(albumtitle_tokens);

In the SQL query, use the [`SEARCH_SUBSTRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_substring) function in the `WHERE` clause. For example, the following query matches an album with title "happy" from the table created in the previous example:

### GoogleSQL

    SELECT AlbumId
    FROM Albums
    WHERE SEARCH_SUBSTRING(AlbumTitle_Tokens, 'happ');

### PostgreSQL

This example uses [`spanner.search_substring`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .

    SELECT albumid
    FROM albums
    WHERE spanner.search_substring(albumtitle_tokens, 'happ');

`TOKENIZE_SUBSTRING` generates *[n-grams](https://en.wikipedia.org/wiki/N-gram)* for each token and stores these n-grams in the search index. The minimum and maximum length of n-grams to generate are configured through optional arguments.

Substring search indexes can use 10-30x more storage as full-text indexes over the same data, because the tokenization produces a lot more tokens. This is especially true if as the difference between `ngram_size_min` and `ngram_size_max` grows. Substring queries also use more resources to execute.

Like [`TOKENIZE_FULLTEXT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#tokenize_fulltext) , you can configure `TOKENIZE_SUBSTRING` to use specific types of content.

## Enable a relative substring search

In addition to the basic substring search, [`SEARCH_SUBSTRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_substring) supports the relative search mode. A relative search refines substring search results.

To enable the relative search mode, set the `relative_search_types` parameter of [`TOKENIZE_SUBSTRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) to a non-empty array with elements of supported relative search types.

When relative search is enabled in tokenization, `SEARCH_SUBSTRING` can perform queries with the following relative search types:

  - `phrase` : matches contiguous substrings
    
    **Examples**
    
    | Stored text                | Substring query.     | Match |
    | -------------------------- | -------------------- | ----- |
    | Bridge over Troubled Water | bridge over          | Yes   |
    | Bridge over Troubled Water | Bridge bridge bridge | No    |
    | Bridge over Troubled Water | brid over            | No    |
    | Bridge over Troubled Water | ridge over trouble   | Yes   |
    | Bridge over Troubled Water | bridge ove troubled  | No    |
    | Bridge over Troubled Water | idge ove             | Yes   |
    | Bridge over Troubled Water | idge , ove           | Yes   |
    | Bridge over Troubled Water | RIDGE OVE            | Yes   |
    | Bridge over Troubled Water | bridge water         | No    |
    

  - `value_prefix` : matches contiguous substrings and the match has to start at the beginning of the value. This is conceptually similar to the `STARTS_WITH` function for case and whitespace normalized strings.
    
    **Examples**
    
    | Stored text                | Substring query | Match |
    | -------------------------- | --------------- | ----- |
    | Bridge over Troubled Water | bridge over     | Yes   |
    | Bridge over Troubled Water | bridge , over   | Yes   |
    | Bridge over Troubled Water | ridge over      | No    |
    | Bridge over Troubled Water | troubled water  | No    |
    

  - `value_suffix` : matches contiguous substrings and the match has to match at the end of the value. This is conceptually similar to the `ENDS_WITH` function for case and whitespace normalized strings.
    
    **Examples**
    
    | Stored text                | Substring query. | Match |
    | -------------------------- | ---------------- | ----- |
    | Bridge over Troubled Water | troubled water   | Yes   |
    | Bridge over Troubled Water | troubled ; water | Yes   |
    | Bridge over Troubled Water | roubled water    | Yes   |
    | Bridge over Troubled Water | troubled wate    | No    |
    | Bridge over Troubled Water | trouble water    | No    |
    | Bridge over Troubled Water | bridge over      | No    |
    

  - `word_prefix:` like `value_prefix` , but the string has to match at a term boundary (rather than a value boundary).
    
    **Examples**
    
    | Stored text                | Substring query | Match |
    | -------------------------- | --------------- | ----- |
    | Bridge over Troubled Water | over trouble    | Yes   |
    | Bridge over Troubled Water | Over , trouble  | Yes   |
    | Bridge over Troubled Water | troub water     | No    |
    | Bridge over Troubled Water | over water      | No    |
    | Bridge over Troubled Water | ove troubled    | No    |
    | Bridge over Troubled Water | ver troubled    | Yes   |
    

  - `word_suffix` : like `value_suffix` , but the string has to match at the end of a term boundary.
    
    **Examples**
    
    | Stored text                | Substring query | Match |
    | -------------------------- | --------------- | ----- |
    | Bridge over Troubled Water | ver troubled    | Yes   |
    | Bridge over Troubled Water | over trouble    | No    |
    | Bridge over Troubled Water | over water      | No    |
    | Bridge over Troubled Water | ove troubled    | No    |
    

## What's next

  - Learn about [full-text search queries](https://docs.cloud.google.com/spanner/docs/full-text-search/query-overview) .
  - Learn how to [rank search results](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [paginate search results](https://docs.cloud.google.com/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [mix full-text and non-text queries](https://docs.cloud.google.com/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
  - Learn how to [search multiple columns](https://docs.cloud.google.com/spanner/docs/full-text-search/search-multiple-columns) .
