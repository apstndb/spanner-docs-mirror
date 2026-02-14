**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes what full-text search is and how it works.

A full-text search lets you build an application that can search a table to find words, phrases, or numbers, instead of just searching for exact matches in structured fields. Full-text searches return the latest transactionally-consistent committed data to your application. Spanner full-text search capabilities also include making spelling corrections, automating language detection of search input, and ranking search results. Spanner automatically expands text searches to include approximate word matching.

You need to create a search index on any columns that you want to make available for full-text searches. Spanner analyzes the data in those columns to identify individual words to add to the search index. Spanner updates the search index with new or modified data as soon as it's committed in the database.

## Types of full-text search

  - **Basic text search** : Searches for content using the entire or part of a word or phrase to reliably receive results. Example query predicates:
      - Matching all words \[tricolor rat terrier\]
      - Exact word or phrases \["rat terrier"\]
      - Any of these words \[miniature OR standard\]
      - Word within close proximity \[world AROUND(3) cup\]
      - Substring \[start\*\]
  - **Numeric search** : Performs numeric equality and inequality searches. Equality searches match a number. Range and inequality searches match a number within a specific range.
  - **n-gram-based search** : Matches words with spelling variations, including proper nouns and names. This type of search also helps to match query text with misspelled names, names with alternate spellings, and text with other spelling variations.
  - **Soundex searches** : Matches similar-sounding words.

## Full-text search features

Spanner full-text search has the following features:

  - **Ranked search results** : Computes a score to gauge how well a query matches a document (for example, giving a heavier weight for column\_A). Use SQL expressions to customize ranking.
  - **Snippets** : Highlights the matching text in the search result.
  - **Global support** : Automatically supports tokenization in different languages, including [CJK](https://en.wikipedia.org/wiki/CJK_characters) segmentation. Manual specification of language lets you perform additional fine-tuning.
  - **Governance** : Finds every occurrence of specific words.
  - **Spelling correction** : Automatically corrects misspelled words in queries to match the correctly-spelled word in storage. For example, if the user searches for "girafe", the search finds documents with "giraffe".
  - **Contextual synonym addition, including stop words** : Automatically adds contextually-relevant synonyms to increase recall. For example, "the house" matches "this house" and "cat picture" matches "kitty picture".
  - **Contextual number translation to and from text** : Matches the textual version of a number to the numeric representation and vice-versa. For example, "five cats" matches "5 cats".
  - **Automatic plural conversion** : Matches "cat" to "cats".

## Full-text search concepts

Full-text search has the following key concepts:

  - A *document* refers to the searchable data in a given row.
  - A *token* refers to each word of a document that's stored in a search index.
  - A *tokenization* process splits a document into tokens.
  - A *tokenizer* is a SQL function used for tokenization.
  - An *inverted index* stores tokens. Use SQL queries to search the inverted index.

## Use case example for full-text search

To understand full-text search, let's take a look at an application that uses a database to store songs for each singer. Each row is a single song. Each song contains columns like title, lyrics, singer, and album. The application uses full-text search to let a user search for a song using natural language queries:

  - The search supports queries that use the `  OR  ` operator, like `  Prince OR Camille  ` . Applications can directly feed the end user input from the search box into the SQL [`  SEARCH  `](/spanner/docs/full-text-search/query-overview#query_a_search_index) function (using the rquery syntax). For more information, see [Query a search index](/spanner/docs/full-text-search/query-overview#query_a_search_index) .
  - Spanner uses search indexes to look for matching documents across different fields. For example, an application can issue a query to search for "cry" in the title, with "so cold" in the lyrics, and "Prince" as the singer.

## Other uses for search indexes

Search indexes have a variety of uses in addition to full-text search, such as the following:

  - Indexing elements in array columns. Consider an application that uses an array column to store tags associated with an item. With search indexes, the application can efficiently look up rows containing a specific tag. For more information, see [Array tokenization](/spanner/docs/full-text-search/numeric-indexes#array-tokenization) .

  - Finding data that resides in the intersection of a set of query conditions. For example, you can use an arbitrary set of attributes (color, size, brand, rating, and so on) to search for a product in a catalog.

  - Using numeric search conditions, alone or in combination with full-text conditions. Some examples for when a search index is useful for numeric searches:
    
      - When it's combined with a full-text application. For example, to find an email with the subject **Picture** and size greater than 1 MB.
      - When it's part of an intersection of conditions described previously. For example, to find products where `  color = "yellow" AND size = 14 AND rating >= 4.5  ` .
      - When searching for the intersection of numeric columns. For example, consider a table storing event start and end times. Search indexes can efficiently implement a query that looks for events that took place at a particular point in time: `  start_time <= @p AND end_time > @p  ` .
    
    For more information, see [Numeric indexes](/spanner/docs/full-text-search/numeric-indexes) .

## Full-text search steps

In Spanner, full-text search requires the following steps:

1.  Tokenize a document using the Spanner tokenizer functions, such as [`  TOKENIZE_SUBSTRING  `](/spanner/docs/reference/standard-sql/search_functions#tokenize_substring) . For more information, see [Tokenization](/spanner/docs/full-text-search/tokenization) .
2.  Create a search index to hold the tokens using the [`  CREATE SEARCH INDEX  `](/spanner/docs/reference/standard-sql/data-definition-language#create-search-index) DDL statement. For more information, see [Search indexes](/spanner/docs/full-text-search/search-indexes) .
3.  Query documents in the search index using the Spanner [`  SEARCH  `](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) function. For more information, see [Query overview](/spanner/docs/full-text-search/query-overview) .
4.  Rank the results of the query using the Spanner [`  SCORE  `](/spanner/docs/reference/standard-sql/search_functions#score) function. For more information, see [Rank search results](/spanner/docs/full-text-search/ranked-search) .

## Limitations

  - Full-text search doesn't support [Assured Workloads](/assured-workloads/docs/overview) .

## Pricing

There are no additional charges from Spanner when you use full-text search, although the implementation of full-text search increases costs due to the need for additional compute and storage resources.

For more information, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

## What's next

  - Learn about [tokenization and Spanner tokenizers](/spanner/docs/full-text-search/tokenization) .
  - Learn about [search indexes](/spanner/docs/full-text-search/search-indexes) .
  - Learn about [full-text search queries](/spanner/docs/full-text-search/query-overview) .
