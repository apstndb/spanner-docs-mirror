**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to use [full-text search](/spanner/docs/full-text-search) in Spanner Graph.

Spanner Graph combines graph and full-text search in one system. This combination lets you derive insights from unstructured data in conjunction with relationships in the graph.

## Before you begin

To run the examples on this page, you need to perform the [set up and query Spanner Graph using the Google Cloud console](/spanner/docs/graph/set-up) procedures. These procedures do the following:

1.  [Create an instance](/spanner/docs/graph/set-up#create-instance) .
2.  [Create a database](/spanner/docs/graph/set-up#create-database) .
3.  [Create a schema for your Spanner Graph database](/spanner/docs/graph/set-up#create-schema) .
4.  [Insert graph data](/spanner/docs/graph/set-up#insert-graph-data) .

## Create tokens and search indexes

The first [step to using full-text search](/spanner/docs/full-text-search#full-text_search_steps) is to tokenize the content you want to search against and create a search index. Full-text search runs queries against the search index.

The following example adds the `  nick_name_token  ` column and uses the [TOKENIZE\_FULLTEXT](/spanner/docs/reference/standard-sql/search_functions#tokenize_fulltext) function to tokenize the text in the `  Account.nick_name  ` column. Next, the search index is created on the `  nick_name_token  ` column.

``` text
ALTER TABLE Account
ADD COLUMN nick_name_token TOKENLIST
AS (TOKENIZE_FULLTEXT(nick_name)) STORED HIDDEN;

CREATE SEARCH INDEX AccountTextSearchIndex
ON Account(nick_name_token) STORING (nick_name);
```

The following example uses the [TOKENIZE\_FULLTEXT](/spanner/docs/reference/standard-sql/search_functions#tokenize_fulltext) function to tokenize the text in `  Account.nick_name  ` and creates a search index on the `  nick_name_token  ` column that contains the tokens.

``` text
ALTER TABLE AccountTransferAccount
ADD COLUMN notes STRING(MAX);
ALTER TABLE AccountTransferAccount
ADD COLUMN notes_token TOKENLIST AS (TOKENIZE_FULLTEXT(notes)) STORED HIDDEN;

CREATE SEARCH INDEX TransferTextSearchIndex
ON AccountTransferAccount(notes_token) STORING (notes);
```

Since some new columns were added to `  Account  ` and `  AccountTransferAccount  ` and you access them as new graph properties in search functions, you need to update the property graph definition using the following statement (more explained in [update existing node or edge definitions](/spanner/docs/graph/create-update-drop-schema#update-existing-node-or-edge) ).

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (Account, Person)
  EDGE TABLES (
    PersonOwnAccount
      SOURCE KEY (id) REFERENCES Person (id)
      DESTINATION KEY (account_id) REFERENCES Account (id)
      LABEL Owns,
    AccountTransferAccount
      SOURCE KEY (id) REFERENCES Account (id)
      DESTINATION KEY (to_id) REFERENCES Account (id)
      LABEL Transfers
  );
```

You can now use full-text search on your graph data.

## Search graph node property

This example shows you how to search for nodes in the graph and explore their relationships.

1.  Update `  Account.nick_name  ` with some text messages.
    
    ``` text
    UPDATE Account SET nick_name = "Fund for vacation at the north pole" WHERE id = 7;
    UPDATE Account SET nick_name = "Fund -- thrill rides!" WHERE id = 16;
    UPDATE Account SET nick_name = "Rainy day fund for the things I still want to do" WHERE id = 20;
    ```

2.  Use the [`  SEARCH  `](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) function to find `  Account  ` nodes in the graph that have either "rainy day" OR "vacation" in their `  nick_name  ` . Use graph traversal to find the amount of money that was transferred into those Accounts. Score the matches by search relevance. Sort and return the results in descending relevance order. Note that you can look for the disjunction of tokens in the same search function call.
    
    ``` text
    GRAPH FinGraph
    MATCH (n:Account)<-[e:Transfers]-(:Account)
    WHERE SEARCH(n.nick_name_token, '"rainy day" | vacation')
    RETURN n.nick_name, e.amount AS amount_added
    ORDER BY SCORE(n.nick_name_token, '"rainy day" | vacation') DESC
    ```
    
    Result:
    
    ``` text
    nick_name                                             amount_added
    Rainy day fund for the things I still want to do      300
    Fund for vacation at the north pole                   500
    ```

## Search graph edge property

This example shows you how to search for specific edges in the graph

1.  Update `  AccountTransferAccount.notes  ` with a text message.
    
    ``` text
    UPDATE AccountTransferAccount SET notes = 'for trip fund'
    WHERE id = 16 AND to_id = 20;
    UPDATE AccountTransferAccount SET notes = '&lt;trip&#39;s very fun!&gt;'
    WHERE id = 20 AND to_id = 7;
    UPDATE AccountTransferAccount SET notes = 'book fee'
    WHERE id = 20 AND to_id = 16;
    ```

2.  Use full-text search to find Transfers edges that contain "trip". Use graph traversal to find the source and destination nodes of those transfers.
    
    ``` text
    GRAPH FinGraph
    MATCH (a:Account)-[e:Transfers WHERE SEARCH(e.notes_token, 'trip')]->(b:Account)
    RETURN a.id AS src_id, b.id AS dst_id, e.notes
    ```
    
    Result:
    
    ``` text
    src_id  dst_id  notes
    
    20      7      &lt;trip&#39;s very fun!&gt;
    16      20     for trip fund
    ```

The search function correctly recalled the first result despite the HTML tags in the text.

## What's next

  - [Learn about Spanner Graph queries](/spanner/docs/graph/queries-overview) .
  - [Learn about full-text search](/spanner/docs/reference/standard-sql/search_functions#search_fulltext) .
  - [Learn about full-text search queries](/spanner/docs/full-text-search/query-overview) .
