**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

In addition to full token matching, Spanner [search indexes](/spanner/docs/full-text-search/search-indexes) support substring searches. This page describes how to perform a substring search as part of a [full-text search](/spanner/docs/full-text-search) in Spanner.

Substring searches have the following characteristics:

  - Case insensitive, discards most punctuation, and normalizes whitespace.
  - No Chinese, Japanese, Korean (CJK) segmentation, since partial CJK queries often segment incorrectly.
  - For multiple search terms, the result must contain a substring from each term. For example, `  'happ momen'  ` matches `  "happy moment"  ` , because both substrings are found in the text. It doesn't match `  "happy day"  ` .

**Examples**

<table>
<thead>
<tr class="header">
<th>Stored text</th>
<th>Substring query</th>
<th>Match</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Bridge over Troubled Water</td>
<td>ridg roub</td>
<td>Yes</td>
</tr>
<tr class="even">
<td>Bridge over Troubled Water</td>
<td>ridg , roub</td>
<td>Yes</td>
</tr>
<tr class="odd">
<td>Bridge over Troubled Water</td>
<td>over brid</td>
<td>Yes</td>
</tr>
<tr class="even">
<td>Bridge over Troubled Water</td>
<td>ate bridge</td>
<td>Yes</td>
</tr>
<tr class="odd">
<td>Bridge over Troubled Water</td>
<td>Bridge bridge bridge</td>
<td>Yes</td>
</tr>
<tr class="even">
<td>Bridge over Troubled Water</td>
<td>bri trou ter</td>
<td>Yes</td>
</tr>
<tr class="odd">
<td>Bridge over Troubled Water</td>
<td>bri dge</td>
<td>Yes</td>
</tr>
<tr class="even">
<td>Bridge over Troubled Water</td>
<td>troubledwater</td>
<td>No</td>
</tr>
<tr class="odd">
<td>Bridge over Troubled Water</td>
<td>trubled</td>
<td>No</td>
</tr>
</tbody>
</table>

For a substring search, use the [`  TOKENIZE_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) function in the `  TOKENLIST  ` column definition, as shown in the following DDL example:

### GoogleSQL

``` text
CREATE TABLE Albums (
AlbumId STRING(MAX) NOT NULL,
AlbumTitle STRING(MAX),
AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_SUBSTRING(AlbumTitle)) HIDDEN
) PRIMARY KEY(AlbumId);
```

### PostgreSQL

This example uses [`  spanner.tokenize_substring  `](/spanner/docs/reference/postgresql/functions#tokenize_substring) .

``` text
CREATE TABLE albums (
albumid character varying NOT NULL,
albumtitle character varying,
albumtitle_tokens spanner.tokenlist
    GENERATED ALWAYS AS (spanner.tokenize_substring(albumtitle)) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));
```

In the SQL query, use the [`  SEARCH_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#search_substring) function in the `  WHERE  ` clause. For example, the following query matches an album with title "happy" from the table created in the previous example:

### GoogleSQL

``` text
SELECT Album
FROM Albums
WHERE SEARCH_SUBSTRING(AlbumTitle_Tokens, 'happ');
```

### PostgreSQL

This example uses [`  spanner.search_substring  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .

``` text
SELECT album
FROM albums
WHERE spanner.search_substring(albumtitle_tokens, 'happ');
```

`  TOKENIZE_SUBSTRING  ` generates *[n-grams](https://en.wikipedia.org/wiki/N-gram)* for each token and stores these n-grams in the search index. The minimum and maximum length of n-grams to generate are configured through optional arguments.

Substring search indexes can use 10-30x more storage as full-text indexes over the same data, because the tokenization produces a lot more tokens. This is especially true if as the difference between `  ngram_size_min  ` and `  ngram_size_max  ` grows. Substring queries also use more resources to execute.

Like [`  TOKENIZE_FULLTEXT  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_fulltext) , you can configure `  TOKENIZE_SUBSTRING  ` to use specific types of content.

## Enable a relative substring search

In addition to the basic substring search, [`  SEARCH_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#search_substring) supports the relative search mode. A relative search refines substring search results.

To enable the relative search mode, set the `  relative_search_types  ` parameter of [`  TOKENIZE_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) to a non-empty array with elements of supported relative search types.

When relative search is enabled in tokenization, `  SEARCH_SUBSTRING  ` can perform queries with the following relative search types:

  - `  phrase  ` : matches contiguous substrings
    
    **Examples**
    
    <table>
    <thead>
    <tr class="header">
    <th>Stored text</th>
    <th>Substring query.</th>
    <th>Match</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>bridge over</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>Bridge bridge bridge</td>
    <td>No</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>brid over</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>ridge over trouble</td>
    <td>Yes</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>bridge ove troubled</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>idge ove</td>
    <td>Yes</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>idge , ove</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>RIDGE OVE</td>
    <td>Yes</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>bridge water</td>
    <td>No</td>
    </tr>
    </tbody>
    </table>

  - `  value_prefix  ` : matches contiguous substrings and the match has to start at the beginning of the value. This is conceptually similar to the `  STARTS_WITH  ` function for case and whitespace normalized strings.
    
    **Examples**
    
    <table>
    <thead>
    <tr class="header">
    <th>Stored text</th>
    <th>Substring query</th>
    <th>Match</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>bridge over</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>bridge , over</td>
    <td>Yes</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>ridge over</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>troubled water</td>
    <td>No</td>
    </tr>
    </tbody>
    </table>

  - `  value_suffix  ` : matches contiguous substrings and the match has to match at the end of the value. This is conceptually similar to the `  ENDS_WITH  ` function for case and whitespace normalized strings.
    
    **Examples**
    
    <table>
    <thead>
    <tr class="header">
    <th>Stored text</th>
    <th>Substring query.</th>
    <th>Match</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>troubled water</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>troubled ; water</td>
    <td>Yes</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>roubled water</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>troubled wate</td>
    <td>No</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>trouble water</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>bridge over</td>
    <td>No</td>
    </tr>
    </tbody>
    </table>

  - `  word_prefix:  ` like `  value_prefix  ` , but the string has to match at a term boundary (rather than a value boundary).
    
    **Examples**
    
    <table>
    <thead>
    <tr class="header">
    <th>Stored text</th>
    <th>Substring query</th>
    <th>Match</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>over trouble</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>Over , trouble</td>
    <td>Yes</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>troub water</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>over water</td>
    <td>No</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>ove troubled</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>ver troubled</td>
    <td>Yes</td>
    </tr>
    </tbody>
    </table>

  - `  word_suffix  ` : like `  value_suffix  ` , but the string has to match at the end of a term boundary.
    
    **Examples**
    
    <table>
    <thead>
    <tr class="header">
    <th>Stored text</th>
    <th>Substring query</th>
    <th>Match</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>ver troubled</td>
    <td>Yes</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>over trouble</td>
    <td>No</td>
    </tr>
    <tr class="odd">
    <td>Bridge over Troubled Water</td>
    <td>over water</td>
    <td>No</td>
    </tr>
    <tr class="even">
    <td>Bridge over Troubled Water</td>
    <td>ove troubled</td>
    <td>No</td>
    </tr>
    </tbody>
    </table>

## What's next

  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
  - Learn how to [rank search results](/spanner/docs/full-text-search/ranked-search) .
  - Learn how to [paginate search results](/spanner/docs/full-text-search/paginate-search-results) .
  - Learn how to [mix full-text and non-text queries](/spanner/docs/full-text-search/mix-full-text-and-non-text-queries) .
  - Learn how to [search multiple columns](/spanner/docs/full-text-search/search-multiple-columns) .
