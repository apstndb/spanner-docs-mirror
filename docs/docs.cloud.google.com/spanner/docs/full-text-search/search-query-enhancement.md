> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This page describes how to use query enhancement as part of a [full-text search](https://docs.cloud.google.com/spanner/docs/full-text-search) .

To increase the likelihood of finding relevant results, Spanner offers advanced capabilities that expand the search query to include related terms, synonyms and spelling corrections. Spanner provides the following options for query enhancement:

  - [Enhanced query](https://docs.cloud.google.com/spanner/docs/full-text-search/search-query-enhancement#enhanced-query)
  - [Custom dictionaries](https://docs.cloud.google.com/spanner/docs/full-text-search/search-query-enhancement#custom-dictionaries)

## Enhanced query

To use enhanced query, set `enhance_query=>true` in the `SEARCH` function. Spanner then automatically expands search queries by including related terms and synonyms, applying stemming, and making spelling corrections. For example, when using enhanced query, the search query `hotl cal` matches the album `Hotel California` .

### GoogleSQL

    SELECT AlbumId
    FROM Albums
    WHERE SEARCH(AlbumTitle_Tokens, 'hotl cal', enhance_query=>true)

### PostgreSQL

    SELECT albumid
    FROM albums
    WHERE spanner.search(albumtitle_tokens, 'hotl cal', enhance_query=>true)

`enhance_query` is a query-time option that doesn't affect tokenization. You can use the same search index with or without `enhance_query` .

Google is continuously improving the query enhancement algorithms. As a result, a query with `enhance_query => true` might yield slightly different results over time.

When `enhance_query` is used, it might increase latency due to the overhead of expanding the search query and executing the resulting larger query.

## Custom dictionaries

You can use custom dictionaries with Spanner full-text search to define synonyms for terms in your dataset. This is useful for capturing synonyms, acronyms, equivalent terms, and other word variations to improve the retrieval of search results.

### Create a custom dictionary table

A custom dictionary is a user-created table containing key-value pairs of terms and their synonyms. To create one, include the `fulltext_dictionary_table = true` option in the `CREATE TABLE` statement. The table must have two columns:

  - `Key` : A non nullable string column for the term to be expanded with synonyms.
  - `Value` : A non nullable array of strings column for an array of synonyms for the key.

There cannot be any columns other than `Key` and `Value` in the table. If a search term matches a `Key` in the dictionary, the search is expanded to include all corresponding synonyms in `Value` along with the key term itself. The following example creates a custom dictionary table named `MyCustomDictionary` . You must also set table option `fulltext_dictionary_table=true` on this table during creation.

### GoogleSQL

    CREATE TABLE MyCustomDictionary (
      Key STRING(MAX) NOT NULL,
      Value ARRAY<STRING(MAX)> NOT NULL,
    ) PRIMARY KEY(Key),
    OPTIONS (fulltext_dictionary_table = true);

### PostgreSQL

    CREATE TABLE mycustomdictionary (
      key character varying  NOT NULL,
      value character varying [] NOT NULL,
      PRIMARY KEY(key)
    )  WITH ( type = 'fulltext_dictionary')

After creating the table, insert your synonyms:

### GoogleSQL

    INSERT INTO MyCustomDictionary (Key, Value) VALUES
    ('album', ['vinyl', 'cassette']),
    ('edm', ['electronic dance music']);

### PostgreSQL

    INSERT INTO mycustomdictionary (key, value) VALUES
    ('album', ARRAY['vinyl', 'cassette']),
    ('edm', ARRAY['electronic dance music']);

When populating the table, keep the following in mind:

  - Keys must be single words.
  - Values can be single words or multi-word phrases. If a value contains multiple words, it is treated as a phrase search during query expansion.
  - All keys and values should be in lowercase. This ensures they match search tokens, which are converted to lowercase by the default tokenizer.

Search expansion only occurs when a search term matches a `Key` . If a search term matches a synonym in `Value` but not a `Key` , the search is not expanded. If you require bidirectional mapping (for example, so that searching for `vinyl` or `cassette` also finds `album` ), you must also insert reverse mappings into the dictionary table. The following example shows how to insert bidirectional mappings for `album` , `vinyl` , and `cassette` :

### GoogleSQL

    INSERT INTO MyCustomDictionary (Key, Value)
    -- 1. Insert album -> vinyl, cassette
    SELECT 'album', ['vinyl', 'cassette']
    UNION ALL
    -- 2. Insert vinyl -> album and cassette -> album
    SELECT syn, ['album']
    FROM UNNEST(['vinyl', 'cassette']) AS syn;

### PostgreSQL

    INSERT INTO mycustomdictionary (key, value)
    -- 1. Insert album -> vinyl, cassette
    SELECT 'album', ARRAY['vinyl', 'cassette']
    UNION ALL
    -- 2. Insert vinyl -> album and cassette -> album
    SELECT syn, ARRAY['album']
    FROM unnest(ARRAY['vinyl', 'cassette']) AS syn;

### Use a custom dictionary in search queries

To use a custom dictionary, specify the dictionary table name in the `dictionary` argument of the `SEARCH` function.

  - If a search term is a key in the dictionary, `SEARCH` also looks for its values. The term `album` matches messages containing `vinyl` or `cassette` .
    
    ### GoogleSQL
    
        SELECT MessageId, Body
        FROM Messages
        WHERE SEARCH(Body_Tokens, 'album', dictionary=>'MyCustomDictionary');
    
    ### PostgreSQL
    
        SELECT messageid, body
        FROM messages
        WHERE spanner.search(body_tokens, 'album', dictionary=>'mycustomdictionary');

  - When a dictionary value contains multiple words such as `electronic dance music` for the key `edm` , it is treated as a phrase search. This means the terms must appear adjacently and in the exact order specified to be considered a match. For example, the following query returns results containing the exact phrase "electronic dance music", but does not match "dance electronic music". This is primarily useful for expanding acronyms and can be used in the following way:
    
    ### GoogleSQL
    
        SELECT MessageId, Body
        FROM Messages
        WHERE SEARCH(Body_Tokens, 'edm', dictionary=>'MyCustomDictionary');
    
    ### PostgreSQL
    
        SELECT messageid, body
        FROM messages
        WHERE spanner.search(body_tokens, 'edm', dictionary=>'mycustomdictionary');

### Dictionary lookup staleness

By default, custom dictionary entries are read with a maximum staleness of 15 seconds. This reduces the overhead of the read operation because reading data with some staleness is more efficient than reading the most current data. Consequently, changes to dictionary entries might take up to 15 seconds to be reflected in search queries. You can override this behavior in the following ways:

  - Using the `fulltext_dictionary_staleness` table option.
  - Using the `fulltext_dictionary_staleness` query hint for more granular control.

The query hint overrides the table option if both are used.

#### `fulltext_dictionary_staleness` table option

You can set the `fulltext_dictionary_staleness` option for the dictionary table. All queries using this dictionary table use this staleness value, unless overridden by the query hint.

### GoogleSQL

The following example shows how to `CREATE` a table with the `fulltext_dictionary_staleness` option:

    CREATE TABLE MyCustomDictionary (
      Key STRING(MAX) NOT NULL,
      Value ARRAY<STRING(MAX)> NOT NULL,
    ) PRIMARY KEY(Key),
    OPTIONS (
      fulltext_dictionary_table = true,
      fulltext_dictionary_staleness = '5s'
    );

The following example shows how to `ALTER` table to change or set the `fulltext_dictionary_staleness` option:

    ALTER TABLE MyCustomDictionary SET OPTIONS (
      fulltext_dictionary_staleness = '60s'
    );

### PostgreSQL

The following example shows how to `CREATE` a table with the `fulltext_dictionary_staleness` option:

    -- Create with 5s staleness
    CREATE TABLE mycustomdictionary (
      key character varying NOT NULL,
      value character varying[]  NOT NULL,
      PRIMARY KEY(key)
    ) WITH (
      type = 'fulltext_dictionary',
      fulltext_dictionary_staleness = '5s'
    );

The PostgreSQL interface doesn't support `ALTER` table to change or set the `fulltext_dictionary_staleness` option.

#### `fulltext_dictionary_staleness` query hint

For more granular control, you can use the `fulltext_dictionary_staleness` query hint to specify a different staleness for an individual query. This hint overrides the table-level setting.

The following example uses a hint to perform dictionary table lookups with zero staleness. This ensures that the most current dictionary entries are read. This approach can increase query latency because reading the most current data is less efficient than allowing for some staleness.

### GoogleSQL

    @{fulltext_dictionary_staleness="0s"}
    SELECT MessageId, Body
    FROM Messages
    WHERE SEARCH(Body_Tokens, 'Bill', dictionary=>'MyCustomDictionary');

### PostgreSQL

    /*@ fulltext_dictionary_staleness='0s' */
    SELECT messageid, body
    FROM messages
    WHERE spanner.search(body_tokens, 'Bill', dictionary=>'mycustomdictionary');

### Known limitations with custom dictionary tables

  - There is limited support for using Spanner Import and Export with custom dictionary tables.
  - Custom dictionary tables must be created in the default schema, and cannot be created in [named schemas](https://docs.cloud.google.com/spanner/docs/named-schemas) .
  - Custom dictionary tables only support the default [SEARCH query dialect](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/search_functions#search_fulltext) .

## Combining custom dictionaries with enhanced query

You can combine custom dictionary synonyms with [`enhanced query`](https://docs.cloud.google.com/spanner/docs/full-text-search/search-query-enhancement#enhanced-query) by setting both `dictionary` and `enhance_query=>true` in the `SEARCH` function. Query enhancement can expand queries with common synonyms or spelling corrections, while custom dictionaries allow you to define your own expansions. For example, if `enhance_query` expands `album` to include `record` , and `MyCustomDictionary` maps `album` to `['vinyl', 'cassette']` , the following query matches messages containing `album` , `record` , `vinyl` , or `cassette` :

### GoogleSQL

    SELECT MessageId, Body
    FROM Messages
    WHERE SEARCH(Body_Tokens, 'album', enhance_query=>true, dictionary=>'MyCustomDictionary');

### PostgreSQL

    SELECT messageid, body
    FROM messages
    WHERE spanner.search(body_tokens, 'album', enhance_query=>true, dictionary=>'mycustomdictionary');

When both enhancements are enabled, they operate independently on the original search terms, and terms generated by one enhancement aren't used as input for the other.

## What's next

  - Learn about [finding approximate matches with fuzzy search](https://docs.cloud.google.com/spanner/docs/full-text-search/fuzzy-search) .
  - Learn how to [rank search results](https://docs.cloud.google.com/spanner/docs/full-text-search/ranked-search) .
