**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to add tokenization to tables. Tokenization is necessary to create the tokens that are used in the [search index](/spanner/docs/full-text-search/search-indexes) .

Tokenization is the process of transforming values into tokens. The method you use to tokenize a document determines the types and efficiency of the searches that users can perform on it.

Spanner provides *tokenizers* for natural language text, substrings, verbatim text, numbers, and booleans. The database schema uses the tokenizer that matches the type of search needed for the column. Tokenizers have the following characteristics:

  - Each tokenizer is a SQL function that gets an input, such as a string or a number, and named arguments for additional options.
  - The tokenizer outputs a `  TOKENLIST  ` .

For example, a text string `  The quick brown fox jumps over the lazy dog  ` is tokenized into `  [the,quick,brown,fox,jumps,over,the,lazy,dog]  ` . An HTML string `  The <b>apple</b> is <i>red</i>  ` is tokenized into `  [the,apple,is,red]  ` .

Tokens have the following characteristics:

  - Tokens are stored in columns that use the `  TOKENLIST  ` data type.
  - Each token is stored as a sequence of bytes, with an optional set of associated attributes. For example, in full-text applications, a token is typically a single word from a textual document.
  - When tokenizing HTML values, Spanner generates attributes that indicate the prominence of a token within the document. Spanner uses these attributes for scoring to boost more prominent terms (such as a heading).

## Tokenizers

Spanner supports the following tokenizer functions:

  - **Full-text tokenizer** ( [`  TOKENIZE_FULLTEXT  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_fulltext) ) produces whole-word tokens for natural language queries.
    
    **Example**
    
    Both of the following functions
    
    ### GoogleSQL
    
    ``` text
    TOKENIZE_FULLTEXT("Yellow apple")
    TOKENIZE_FULLTEXT("Yellow <b>apple</b>", content_type=>"text/html")
    ```
    
    ### PostgreSQL
    
    This example uses [`  spanner.tokenize_fulltext  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .
    
    ``` text
    spanner.tokenize_fulltext("Yellow apple")
    spanner.tokenize_fulltext('Yellow <b>apple</b>', context_type=>'text/html')
    ```
    
    produce the same tokens: `  [yellow,apple]  ` .

  - **Substring tokenizer** ( [`  TOKENIZE_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) ) generates tokens for each n-gram of each word. It's used to find substrings of words in a text.
    
    **Example**
    
    ### GoogleSQL
    
    ``` text
    TOKENIZE_SUBSTRING('hello world', ngram_size_min=>4, ngram_size_max=>6)
    ```
    
    ### PostgreSQL
    
    This example uses [`  spanner.tokenize_substring  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .
    
    ``` text
    spanner.tokenize_substring('hello world', ngram_size_min=>4, ngram_size_max=>6)
    ```
    
    Produces the following tokens: `  [ello,hell,hello,orld,worl,world]  ` .

  - **N-gram tokenizer** ( [`  TOKENIZE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_ngrams) ) generates n-grams from an input (without splitting it into separate words). It is used to accelerate regular expression predicates.
    
    **Example**
    
    The following function:
    
    ### GoogleSQL
    
    ``` text
    TOKENIZE_NGRAMS("Big Time", ngram_size_min=>4, ngram_size_max=>4)
    ```
    
    ### PostgreSQL
    
    This example uses [`  spanner.tokenize_ngrams  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .
    
    ``` text
    spanner.tokenize_ngrams('big time', ngram_size_min=>4, ngram_size_max=>4)
    ```
    
    Produces the following tokens: `  ["Big ","ig T","g Ti"," Tim", "Time"]  ` .

  - **Exact match tokenizers** ( [`  TOKEN  `](/spanner/docs/reference/standard-sql/search_functions#token) and [`  TOKENIZE_BOOL  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_bool) ) are used to look up rows containing a certain value in one of their columns. For example, an application that indexes a products catalog might want to search products of a particular brand and color.
    
    **Examples**
    
    The following functions:
    
    ### GoogleSQL
    
    ``` text
    TOKEN("hello")
    TOKEN(["hello", "world"])
    ```
    
    ### PostgreSQL
    
    This example uses [`  spanner.token  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .
    
    ``` text
    spanner.token('hello')
    ```
    
    Produces the following tokens: `  [hello]  ` .
    
    The following function:
    
    ### GoogleSQL
    
    ``` text
    TOKENIZE_BOOL(true)
    ```
    
    ### PostgreSQL
    
    This example uses [`  spanner.tokenize_bool  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .
    
    ``` text
    spanner.tokenize_bool(true)
    ```
    
    Produces the following token: `  [y]  ` .

  - **Number tokenizers** ( [`  TOKENIZE_NUMBER  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_number) ) are used to generate a set of tokens that accelerate numeric comparison searches. For equality conditions, the token is the number itself. For range conditions (like `  rating >= 3.5  ` ) the set of tokens are more elaborate.
    
    **Examples**
    
    The following function statements:
    
    ### GoogleSQL
    
    ``` text
    TOKENIZE_NUMBER(42, comparison_type=>'equality')
    TOKENIZE_NUMBER(42, comparison_type=>'all', granularity=>10, min=>1, max=>100)
    ```
    
    ### PostgreSQL
    
    This example uses [`  spanner.tokenize_number  `](/spanner/docs/reference/postgresql/functions-and-operators#search_functions) .
    
    ``` text
    spanner.tokenize_number(42, comparison_type=>'equality')
    spanner.tokenize_number(42, comparison_type=>'all', granularity=>10, min=>1, max=>100)
    ```
    
    Produce the following tokens, respectively: `  "==42"  ` and `  "==42"  ` , `  "[1,75]"  ` , `  "[36, 45]"  ` , `  "[36,55]"  ` , `  "[36, 75]"  ` .

  - **JSON and JSONB tokenizers** ( [`  TOKENIZE_JSON  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_json) and [`  TOKENIZE_JSONB  `](/spanner/docs/reference/postgresql/functions-and-operators#indexing) are used to generate a set of tokens that accelerate JSON containment and key existence predicates, such as `  doc[@key] IS NOT NULL  ` (GoogleSQL) or `  doc ? 'key'  ` (PostgreSQL).

Tokenization functions are usually used in a [generated column](/spanner/docs/generated-column/how-to) expression. These columns are defined as `  HIDDEN  ` so that they aren't included in `  SELECT *  ` query results.

The following example uses a full-text tokenizer and a numeric tokenizer to create a database that stores music album names and ratings. The DDL statement does two things:

1.  Defines the data columns `  AlbumTitle  ` and `  Rating  ` .

2.  Defines `  AlbumTitle_Tokens  ` and `  AlbumRating_Tokens  ` . These `  TOKENLIST  ` columns tokenize the values in the data columns so that Spanner can index them.
    
    ### GoogleSQL
    
    ``` text
    CREATE TABLE Albums (
      AlbumId STRING(MAX) NOT NULL,
      AlbumTitle STRING(MAX),
      Rating FLOAT64,
      AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN,
      Rating_Tokens TOKENLIST AS (TOKENIZE_NUMBER(Rating)) HIDDEN
    ) PRIMARY KEY(AlbumId);
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE TABLE albums (
      albumid character varying NOT NULL,
      albumtitle character varying,
      albumtitle_Tokens spanner.tokenlist GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
    PRIMARY KEY(albumid));
    ```

Whenever the base values are modified, `  AlbumTitle_Tokens  ` and `  Rating_Tokens  ` are automatically updated.

## Tokenize plain text or HTML content

Text tokenization supports plain text and HTML content types. Use the Spanner [`  TOKENIZE_FULLTEXT  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_fulltext) function to create tokens. Then use the [`  CREATE SEARCH INDEX  `](/spanner/docs/reference/standard-sql/data-definition-language#create-search-index) DDL statement to generate the search index.

For example, the following `  CREATE TABLE  ` DDL statement uses the `  TOKENIZE_FULLTEXT  ` function to create tokens from `  AlbumTitles  ` in the `  Albums  ` table. The [`  CREATE SEARCH INDEX  `](/spanner/docs/reference/standard-sql/data-definition-language#create-search-index) DDL statement creates a search index with the new `  AlbumTitles_Tokens  ` .

### GoogleSQL

``` text
CREATE TABLE Albums (
  AlbumId STRING(MAX) NOT NULL,
  AlbumTitle STRING(MAX),
  AlbumTitle_Tokens TOKENLIST AS (TOKENIZE_FULLTEXT(AlbumTitle)) HIDDEN
) PRIMARY KEY(AlbumId);

CREATE SEARCH INDEX AlbumsIndex ON Albums(AlbumTitle_Tokens)
```

### PostgreSQL

``` text
CREATE TABLE albums (
  albumid character varying NOT NULL,
  albumtitle character varying,
  albumtitle_tokens spanner.tokenlist
      GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle)) VIRTUAL HIDDEN,
PRIMARY KEY(albumid));

CREATE SEARCH INDEX albumsindex ON albums(albumtitle_tokens)
```

The tokenization process uses the following rules:

  - Tokenization doesn't include [stemming](https://en.wikipedia.org/wiki/Stemming) or correction of misspelled words. For example, in a sentence like "A cat was looking at a group of cats", the token "cat" is indexed separately from the token "cats". Compared to other search engines that normalize tokens during writes, Spanner provides an option to expand the search query to include different forms of words. For more information, see [Enhanced query mode](/spanner/docs/full-text-search/query-overview#enhanced_query_mode) .
  - Stopwords (like "a") are included in the search index.
  - Full-text search is always case insensitive. The tokenization process converts all tokens to lowercase.

The tokenization process tracks the positions for each token in the original text. These positions are later used to match phrases. The positions are stored in the search index alongside docids.

Google continues to improve tokenization algorithms. In some cases, this might lead to a string getting tokenized differently in the future from the way it is tokenized now. We expect such cases to be extremely rare. An example of this is if there's an improvement in the Chinese, Japanese, and Korean (CJK) language segmentation.

The `  content_type  ` argument specifies whether the content format uses plain text or HTML. Use the following settings to set the `  content_type  ` :

  - For text tokenization, set the `  content_type  ` argument to " `  text/plain  ` ". This is the default setting.
  - For HTML tokenization, set the `  content_type  ` argument to `  "text/html  ` ". Without this argument, HTML tags are treated as punctuation. In HTML mode, Spanner uses heuristics to infer how prominent the text is on the page. For example, whether the text is in a heading or its font size. The supported attributes for HTML include `  small  ` , `  medium  ` , `  large  ` , `  title  ` , and \`link'. Like position, the attribute is stored alongside the token in the search index. Tokenization doesn't create tokens for any HTML tags.

Token attributes don't impact matching or the results of the `  SEARCH  ` or `  SEARCH_SUBSTRING  ` function. They're only used for [ranking](/spanner/docs/full-text-search/ranked-search) .

The following example shows how to tokenize text:

### GoogleSQL

``` text
CREATE TABLE T (
  ...
  Text STRING(MAX),
  Html STRING(MAX),
  Text_Tokens TOKENLIST
    AS (TOKENIZE_FULLTEXT(Text, content_type=>"text/plain")) HIDDEN,
  Html_Tokens TOKENLIST
    AS (TOKENIZE_FULLTEXT(Html, content_type=>"text/html")) HIDDEN
) PRIMARY KEY(...);
```

### PostgreSQL

``` text
CREATE TABLE t (
  ...
  text character varying,
  html character varying,
  text_tokens spanner.tokenlist
      GENERATED ALWAYS AS (spanner.tokenize_fulltext(text, content_type=>"text/plain")) VIRTUAL HIDDEN,
  html_tokens spanner.tokenlist
      GENERATED ALWAYS AS (spanner.tokenize_fulltext(html, content_type=>'type/html')) VIRTUAL HIDDEN,
PRIMARY KEY(...));
```

## Language detection refinement with the `     language_tag    ` argument

Tokenization detects the input language automatically, by default. When the input language is known, a `  language_tag  ` argument can be used to refine this behavior:

### GoogleSQL

``` text
AlbumTitle_Tokens TOKENLIST
  AS (TOKENIZE_FULLTEXT(AlbumTitle, language_tag=>"en-us")) HIDDEN
```

### PostgreSQL

``` text
albumtitle_tokens spanner.tokenlist
      GENERATED ALWAYS AS (spanner.tokenize_fulltext(albumtitle, language_tag=>'en-us')) VIRTUAL HIDDEN
```

Most applications leave the `  language_tag  ` argument unspecified and instead rely on automatic language detection. Segmentation for Asian languages like Chinese, Korean, and Japanese doesn't require setting the tokenization language.

The following examples show cases where the `  language_tag  ` affects tokenization:

<table>
<thead>
<tr class="header">
<th>Tokenization function</th>
<th>Produced tokens</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       TOKENIZE_FULLTEXT("A tout pourquoi il y a un parce que")      </code></td>
<td>[a, tout, pourquoi, il, ya, un, parce, que]</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TOKENIZE_FULLTEXT("A tout pourquoi il y a un parce que", \ language_tag=&gt;"fr"      </code> )</td>
<td>[a, tout, pourquoi, il, y, a, un, parce, que]</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TOKENIZE_FULLTEXT("旅 行")      </code></td>
<td>Two tokens: [旅, 行]</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TOKENIZE_FULLTEXT("旅 行", language_tag=&gt;"zh")      </code></td>
<td>One token: [旅行]</td>
</tr>
</tbody>
</table>

## What's next

  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
  - Learn about [numeric indexes](/spanner/docs/full-text-search/numeric-indexes) .
  - Learn about [index partitioning](/spanner/docs/full-text-search/partition-search-index) .
