**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to use a fuzzy search as part of a [full-text search](/spanner/docs/full-text-search) .

In addition to performing exact token searches using the [`  SEARCH  `](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) and [`  SEARCH_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#search_substring) functions, Spanner also supports approximate (or fuzzy) searches. Fuzzy searches find matching documents despite small differences between the query and the document.

Spanner supports the following types of fuzzy search:

  - N-grams-based approximate search
  - Phonetic search using [Soundex](https://en.wikipedia.org/wiki/Soundex)

## Use an n-grams-based approximate search

N-grams-based fuzzy search relies on the same substring tokenization that a [substring search](/spanner/docs/full-text-search/substring-search) requires. The configuration of the tokenizer is important as it affects search quality and performance. The following example shows how to create a query with misspelled or differently spelled words to find approximate matches in the search index.

**Schema**

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  AlbumTitle STRING(MAX),
  AlbumTitle_Tokens TOKENLIST AS (
    TOKENIZE_SUBSTRING(AlbumTitle, ngram_size_min=>2, ngram_size_max=>3,
                      relative_search_types=>["word_prefix", "word_suffix"])) HIDDEN
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex
ON Albums(AlbumTitle_Tokens)
STORING (AlbumTitle);
```

### PostgreSQL

This example uses [`  spanner.tokenize_substring  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  albumtitle character varying,
  albumtitle_tokens spanner.tokenlist GENERATED ALWAYS AS (
    spanner.tokenize_substring(albumtitle, ngram_size_min=>2, ngram_size_max=>3,
                      relative_search_types=>'{word_prefix, word_suffix}'::text[])) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsindex
ON albums(albumtitle_tokens)
INCLUDE (albumtitle);
```

**Query**

The following query finds the albums with titles that are the closest to "Hatel Kaliphorn", such as "Hotel California".

### GoogleSQL

``` text
SELECT AlbumId
FROM Albums
WHERE SEARCH_NGRAMS(AlbumTitle_Tokens, "Hatel Kaliphorn")
ORDER BY SCORE_NGRAMS(AlbumTitle_Tokens, "Hatel Kaliphorn") DESC
LIMIT 10
```

### PostgreSQL

This examples uses [`  spanner.score_ngrams  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) and [`  spanner.search_ngrams  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .

``` text
SELECT albumid
FROM albums
WHERE spanner.search_ngrams(albumtitle_tokens, 'Hatel Kaliphorn')
ORDER BY spanner.score_ngrams(albumtitle_tokens, 'Hatel Kaliphorn') DESC
LIMIT 10
```

### Optimize performance and recall for an n-grams-based approximate search

The sample query in the previous section searches in two phases, using two different functions:

1.  [`  SEARCH_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#search_ngrams) finds all candidate albums that have shared n-grams with the search query. For example, three-character n-grams for "California" include `  [cal, ali, lif, ifo, for, orn, rni, nia]  ` and for "Kaliphorn" include `  [kal, ali, lip, iph, pho, hor, orn]  ` . The shared n-grams in these data sets are `  [ali, orn]  ` . By default, `  SEARCH_NGRAMS  ` matches all documents with at least two shared n-grams, therefore "Kaliphorn" matches "California".
2.  [`  SCORE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#score_ngrams) ranks matches by similarity. The similarity of two strings is defined as a ratio of distinct shared n-grams to distinct non-shared n-grams:

$$ \\frac{shared\\\_ngrams}{total\\\_ngrams\_{index} + total\\\_ngrams\_{query} - shared\\\_ngrams} $$

Usually the search query is the same across both the `  SEARCH_NGRAMS  ` and `  SCORE_NGRAMS  ` functions. The recommended way to do this is to use the argument with [query parameters](/spanner/docs/reference/standard-sql/lexical#query_parameters) rather than with string literals, and specify the same query parameter in the `  SEARCH_NGRAMS  ` and `  SCORE_NGRAMS  ` functions.

Spanner has three configuration arguments that can be used with `  SEARCH_NGRAMS  ` :

  - The minimum and maximum sizes for n-grams are specified with the [`  TOKENIZE_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) or [`  TOKENIZE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_ngrams) functions. We don't recommend one character n-grams because they could match a very large number of documents. On the other hand, long n-grams cause `  SEARCH_NGRAMS  ` to miss short misspelled words.
  - The minimum number of n-grams that `  SEARCH_NGRAMS  ` must match (set with the `  min_ngrams  ` and `  min_ngrams_percent  ` arguments in `  SEARCH_NGRAMS  ` ). Higher numbers typically make the query faster, but reduce recall.

In order to achieve a good balance between performance and recall, you can configure these arguments to fit the specific query and workload.

We also recommend including an inner `  LIMIT  ` to avoid creating very expensive queries when a combination of popular n-grams is encountered.

### GoogleSQL

``` text
SELECT AlbumId
FROM (
  SELECT AlbumId,
        SCORE_NGRAMS(AlbumTitle_Tokens, @p) AS score
  FROM Albums
  WHERE SEARCH_NGRAMS(AlbumTitle_Tokens, @p)
  LIMIT 10000  # inner limit
)
ORDER BY score DESC
LIMIT 10  # outer limit
```

### PostgreSQL

This examples uses [`  spanner.score_ngrams  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) and [`  spanner.search_ngrams  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) . The query parameter `  $1  ` is bound to 'Hatel Kaliphorn'.

``` text
SELECT albumid
FROM
  (
    SELECT albumid, spanner.score_ngrams(albumtitle_tokens, $1) AS score
    FROM albums
    WHERE spanner.search_ngrams(albumtitle_tokens, $1)
    LIMIT 10000
  ) AS inner_query
ORDER BY inner_query.score DESC
LIMIT 10
```

### N-grams-based fuzzy search versus enhanced query mode

Alongside n-grams-based fuzzy search, the [enhanced query mode](/spanner/docs/full-text-search/query-overview#enhanced_query_mode) also handles some misspelled words. Thus, there is some overlap between the two features. The following table summarizes the differences:

<table>
<tbody>
<tr class="odd">
<td></td>
<td><strong>n-grams-based fuzzy search</strong></td>
<td><strong>Enhanced query mode</strong></td>
</tr>
<tr class="even">
<td>Cost</td>
<td>Requires a more expensive substring tokenization based on n-grams</td>
<td>Requires a less expensive full-text tokenization</td>
</tr>
<tr class="odd">
<td>Search query types</td>
<td>Works well with short documents with a few words, such as with a person name, city name, or product name</td>
<td>Works equally well with any size documents and any size search queries</td>
</tr>
<tr class="even">
<td>Partial words search</td>
<td>Performs a substring search that allows for misspellings</td>
<td>Only supports a search for entire words ( <code dir="ltr" translate="no">       SEARCH_SUBSTRING      </code> doesn't support the <code dir="ltr" translate="no">       enhance_query      </code> argument)</td>
</tr>
<tr class="odd">
<td>Misspelled words</td>
<td>Supports misspelled words in either index or query</td>
<td>Only supports misspelled words in the query</td>
</tr>
<tr class="even">
<td>Corrections</td>
<td>Finds any misspelled matches, even if the match isn't a real word</td>
<td>Corrects misspellings for common, well-known words</td>
</tr>
</tbody>
</table>

## Perform a phonetic search with Soundex

Spanner provides the [`  SOUNDEX  `](/spanner/docs/reference/standard-sql/string_functions#soundex) function for finding words that are spelled differently, but sound the same. For example, `  SOUNDEX("steven")  ` , `  SOUNDEX("stephen")  ` and `  SOUNDEX("stefan")  ` are all "s315", while `  SOUNDEX("stella")  ` is "s340". `  SOUNDEX  ` is case sensitive and only works for Latin-based alphabets.

Phonetic search with `  SOUNDEX  ` can be implemented with a generated column and a search index as shown in the following example:

### GoogleSQL

``` text
CREATE TABLE Singers (
  SingerId INT64,
  AlbumTitle STRING(MAX),
  AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN,
  Name STRING(MAX),
  NameSoundex STRING(MAX) AS (LOWER(SOUNDEX(Name))),
  NameSoundex_Tokens TOKENLIST AS (TOKEN(NameSoundex)) HIDDEN
) PRIMARY KEY(SingerId);

CREATE SEARCH INDEX SingersPhoneticIndex ON Singers(AlbumTitle_Tokens, NameSoundex_Tokens);
```

### PostgreSQL

This example uses [`  spanner.soundex  `](/spanner/docs/reference/postgresql/functions-and-operators#string_functions) .

``` text
CREATE TABLE singers (
  singerid bigint,
  albumtitle character varying,
  albumtitle_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
  name character varying,
  namesoundex character varying GENERATED ALWAYS AS (lower(spanner.soundex(name))) VIRTUAL,
  namesoundex_tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.token(lower(spanner.soundex(name))) VIRTUAL HIDDEN,
PRIMARY KEY(singerid));

CREATE SEARCH INDEX singersphoneticindex ON singers(albumtitle_tokens, namesoundex_tokens);
```

The following query matches "stefan" to "Steven" on `  SOUNDEX  ` , along with `  AlbumTitle  ` containing "cat":

### GoogleSQL

``` text
SELECT SingerId
FROM Singers
WHERE NameSoundex = LOWER(SOUNDEX("stefan")) AND SEARCH(AlbumTitle_Tokens, "cat")
```

### PostgreSQL

``` text
SELECT singerid
FROM singers
WHERE namesoundex = lower(spanner.soundex('stefan')) AND spanner.search(albumtitle_tokens, 'cat')
```

## What's next

  - Learn about [tokenization and Spanner tokenizers](/spanner/docs/full-text-search/tokenization) .
  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
