**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to use vector search in Spanner Graph to find K-nearest neighbors (KNN) and approximate nearest neighbors (ANN). You can use vector distance functions to perform KNN and ANN vector search for use cases like similarity search or retrieval-augmented generation for generative AI applications.

Spanner Graph supports the following distance functions to [perform KNN vector similarity search](#find-knn) :

  - [`  COSINE_DISTANCE()  `](/spanner/docs/reference/standard-sql/mathematical_functions#cosine_distance) : measures the shortest distance between two vectors.
  - [`  EUCLIDEAN_DISTANCE()  `](/spanner/docs/reference/standard-sql/mathematical_functions#euclidean_distance) : measures the cosine of the angle between two vectors.
  - [`  DOT_PRODUCT()  `](/spanner/docs/reference/standard-sql/mathematical_functions#dot_product) : calculates the cosine of the angle multiplied by the product of corresponding vector magnitudes. If you know that all the vector embeddings in your dataset are normalized, then you can use `  DOT_PRODUCT()  ` as a distance function.

For more information, see [Perform vector similarity search in Spanner by finding the K-nearest neighbors](/spanner/docs/find-k-nearest-neighbors) .

Spanner Graph also supports the following approximate distance functions to [perform ANN vector similarity search](#create-vector-index-find-ann) :

  - [`  APPROX_COSINE_DISTANCE  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_cosine_distance) : measures the approximate shortest distance between two vectors.
  - [`  APPROX_EUCLIDEAN_DISTANCE  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_euclidean_distance) : measures the approximate cosine of the angle between two vectors.
  - [`  APPROX_DOT_PRODUCT  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product) : calculates the approximate cosine of the angle multiplied by the product of corresponding vector magnitudes. If you know that all the vector embeddings in your dataset are normalized, then you can use `  DOT_PRODUCT()  ` as a distance function.

For more information, see [Find approximate nearest neighbors, create vector index, and query vector embeddings](/spanner/docs/find-approximate-nearest-neighbors) .

## Before you begin

To run the examples in this document, you must first follow the steps in [Set up and query Spanner Graph](/spanner/docs/graph/set-up) to do the following:

1.  [Create an instance](/spanner/docs/graph/set-up#create-instance) .
2.  [Create a database with a Spanner Graph schema](/spanner/docs/graph/set-up#create-database) .
3.  [Insert essential graph data](/spanner/docs/graph/set-up#insert-graph-data) .

After you insert the essential graph data, make the following updates to your database.

### Insert additional vector data in graph database

To make the required updates to your graph database, do the following:

1.  Add a new column, `  nick_name_embeddings  ` , to the `  Account  ` input table.
    
    ``` text
    ALTER TABLE Account
    ADD COLUMN nick_name_embeddings ARRAY<FLOAT32>(vector_length=>4);
    ```

2.  Add data to the `  nick_name  ` column.
    
    ``` text
    UPDATE Account SET nick_name = "Fund for a refreshing tropical vacation" WHERE id = 7;
    UPDATE Account SET nick_name = "Fund for a rainy day!" WHERE id = 16;
    UPDATE Account SET nick_name = "Saving up for travel" WHERE id = 20;
    ```

3.  Create embeddings for the text in the `  nick_name  ` column, and populate them into the new `  nick_name_embeddings  ` column.
    
    To generate Vertex AI embeddings for your operational data in Spanner Graph, see [Get Vertex AI text embeddings](/spanner/docs/ml-tutorial-embeddings) .
    
    For illustrative purposes, our examples use artificial, low-dimensional vector values.
    
    ``` text
    UPDATE Account SET nick_name_embeddings = ARRAY<FLOAT32>[0.3, 0.5, 0.8, 0.7] WHERE id = 7;
    UPDATE Account SET nick_name_embeddings = ARRAY<FLOAT32>[0.4, 0.9, 0.7, 0.1] WHERE id = 16;
    UPDATE Account SET nick_name_embeddings = ARRAY<FLOAT32>[0.2, 0.5, 0.6, 0.6] WHERE id = 20;
    ```

4.  Add two new columns to the `  AccountTransferAccount  ` input table: `  notes  ` and `  notes_embeddings  ` .
    
    ``` text
    ALTER TABLE AccountTransferAccount
    ADD COLUMN notes STRING(MAX);
    ALTER TABLE AccountTransferAccount
    ADD COLUMN notes_embeddings ARRAY<FLOAT32>(vector_length=>4);
    ```

5.  Create embeddings for the text in the `  notes  ` column, and populate them into the `  notes_embeddings  ` column.
    
    To generate Vertex AI embeddings for your operational data in Spanner Graph, see [Get Vertex AI text embeddings](/spanner/docs/ml-tutorial-embeddings) .
    
    For illustrative purposes, our examples use artificial, low-dimensional vector values.
    
    ``` text
    UPDATE AccountTransferAccount
    SET notes = "for shared cost of dinner",
      notes_embeddings = ARRAY<FLOAT32>[0.3, 0.5, 0.8, 0.7]
    WHERE id = 16 AND to_id = 20;
    UPDATE AccountTransferAccount
    SET notes = "fees for tuition",
      notes_embeddings = ARRAY<FLOAT32>[0.1, 0.9, 0.1, 0.7]
    WHERE id = 20 AND to_id = 7;
    UPDATE AccountTransferAccount
    SET notes = 'loved the lunch',
      notes_embeddings = ARRAY<FLOAT32>[0.4, 0.5, 0.7, 0.9]
    WHERE id = 20 AND to_id = 16;
    ```

6.  After adding new columns to the `  Account  ` and `  AccountTransferAccount  ` input tables, update the property graph definition using the following statements. For more information, see [Update existing node or edge definitions](/spanner/docs/graph/create-update-drop-schema#update-existing-node-or-edge) .
    
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

## Find K-nearest neighbors

In the following example, use the `  EUCLIDEAN_DISTANCE()  ` function to perform KNN vector search on the nodes and edges of your graph database.

### Perform KNN vector search on graph nodes

You can perform a KNN vector search on the `  nick_name_embeddings  ` property of the `  Account  ` node. This KNN vector search returns the account owner's `  name  ` and the account's `  nick_name  ` . In the following example, the result shows the top two K-nearest neighbors for *accounts for leisure travel and vacation* , which is represented by the `  [0.2, 0.4, 0.9, 0.6]  ` vector embedding.

``` text
GRAPH FinGraph
MATCH (p:Person)-[:Owns]->(a:Account)
RETURN p.name, a.nick_name
ORDER BY EUCLIDEAN_DISTANCE(a.nick_name_embeddings,
  -- An illustrative embedding for 'accounts for leisure travel and vacation'
  ARRAY<FLOAT32>[0.2, 0.4, 0.9, 0.6])
LIMIT 2;
```

#### Results

<table>
<thead>
<tr class="header">
<th>name</th>
<th>nick_name</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Alex</td>
<td>Fund for a refreshing tropical vacation</td>
</tr>
<tr class="even">
<td>Dana</td>
<td>Saving up for travel</td>
</tr>
</tbody>
</table>

### Perform KNN vector search on graph edges

You can perform a KNN vector search on the `  notes_embeddings  ` property of the `  Owns  ` edge. This KNN vector search returns the account owner's `  name  ` and the transfer's `  notes  ` . In the following example, the result shows the top two K-nearest neighbors for *food expenses* , which is represented by the `  [0.2, 0.4, 0.9, 0.6]  ` vector embedding.

``` text
GRAPH FinGraph
MATCH (p:Person)-[:Owns]->(:Account)-[t:Transfers]->(:Account)
WHERE t.notes_embeddings IS NOT NULL
RETURN p.name, t.notes
ORDER BY EUCLIDEAN_DISTANCE(t.notes_embeddings,
  -- An illustrative vector embedding for 'food expenses'
  ARRAY<FLOAT32>[0.2, 0.4, 0.9, 0.6])
LIMIT 2;
```

#### Results

<table>
<thead>
<tr class="header">
<th>name</th>
<th>notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Lee</td>
<td>for shared cost of dinner</td>
</tr>
<tr class="even">
<td>Dana</td>
<td>loved the lunch</td>
</tr>
</tbody>
</table>

## Create a vector index and find approximate nearest neighbors

**Note:** You can't use ANN search to search the edges in a Spanner Graph database.

To perform an ANN search, you must create a [specialized vector index](/spanner/docs/find-approximate-nearest-neighbors#vector-index) that Spanner Graph uses to accelerate the vector search. The vector index must use a specific distance metric. You can choose the distance metric most appropriate for your use case by setting the `  distance_type  ` parameter to one of `  COSINE  ` , `  DOT_PRODUCT  ` or `  EUCLIDEAN  ` . For more information, see [VECTOR INDEX statements](/spanner/docs/reference/standard-sql/data-definition-language#vector_index_statements) .

In the following example, you create a vector index using the euclidean distance type on the `  nick_name_embedding  ` column of the `  Account  ` input table:

``` text
CREATE VECTOR INDEX NickNameEmbeddingIndex
ON Account(nick_name_embeddings)
WHERE nick_name_embeddings IS NOT NULL
OPTIONS (distance_type = 'EUCLIDEAN', tree_depth = 2, num_leaves = 1000);
```

### Perform ANN vector search on graph nodes

After you create a vector index, you can perform a ANN vector search on the `  nick_name  ` property of the `  Account  ` node. The ANN vector search returns the account owner's `  name  ` and the account's `  nick_name  ` . In the following example, the result shows the top two approximate nearest neighbors for *accounts for leisure travel and vacation* , which is represented by the `  [0.2, 0.4, 0.9, 0.6]  ` vector embedding.

The [graph hint](/spanner/docs/reference/standard-sql/graph-query-statements#graph_hints) forces the query optimizer to use the specified, vector index in the query execution plan.

``` text
GRAPH FinGraph
MATCH (@{FORCE_INDEX=NickNameEmbeddingIndex} a:Account)
WHERE a.nick_name_embeddings IS NOT NULL
RETURN a, APPROX_EUCLIDEAN_DISTANCE(a.nick_name_embeddings,
  -- An illustrative embedding for 'accounts for leisure travel and vacation'
  ARRAY<FLOAT32>[0.2, 0.4, 0.9, 0.6],
  options => JSON '{"num_leaves_to_search": 10}') AS distance
ORDER BY distance
LIMIT 2

NEXT

MATCH (p:Person)-[:Owns]->(a)
RETURN p.name, a.nick_name;
```

#### Results

<table>
<thead>
<tr class="header">
<th>name</th>
<th>nick_name</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Alex</td>
<td>Fund for a refreshing tropical vacation</td>
</tr>
<tr class="even">
<td>Dana</td>
<td>Saving up for travel</td>
</tr>
</tbody>
</table>

## What's next

  - [Perform vector similarity search in Spanner by finding the K-nearest neighbors](/spanner/docs/find-k-nearest-neighbors) .
  - [Find approximate nearest neighbors, create vector index, and query vector embeddings](/spanner/docs/find-approximate-nearest-neighbors) .
  - [Get Vertex AI text embeddings](/spanner/docs/ml-tutorial-embeddings)
  - Learn more about [Spanner Graph queries](/spanner/docs/graph/queries-overview) .
  - Learn [best practices for tuning queries](/spanner/docs/graph/best-practices-tuning-queries) .
