**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page explains how to create and manage Spanner vector indexes, which use approximate nearest neighbor (ANN) search and tree-based structures to accelerate vector similarity searches on your data.

Spanner accelerates approximate nearest neighbor (ANN) vector searches by using a specialized vector index. This index leverages Google Research's [Scalable Nearest Neighbor (ScaNN)](https://github.com/google-research/google-research/tree/master/scann) , a highly efficient nearest neighbor algorithm.

The vector index uses a tree-based structure to partition data and facilitate faster searches. Spanner offers both two-level and three-level tree configurations:

  - Two-level tree configuration: Leaf nodes ( `  num_leaves  ` ) contain groups of closely related vectors along with their corresponding centroid. The root level consists of the centroids from all leaf nodes.
  - Three-level tree configuration: Similar in concept to a two-level tree, while introducing an additional branch layer ( `  num_branches  ` ), from which leaf node centroids are further partitioned to form the root level ( `  num_leaves  ` ).

Spanner picks an index for you. However, if you know that a specific index works best, then you can use the [`  FORCE_INDEX  ` hint](/spanner/docs/secondary-indexes#index-directive) to choose to use the most appropriate vector index for your use case.

For more information, see [`  VECTOR INDEX  ` statements](/spanner/docs/reference/standard-sql/data-definition-language#vector_index_statements) .

### Limitations

  - You can't pre-split vector indexes. For more information, see [Pre-splitting overview](/spanner/docs/pre-splitting-overview#limitations) .

### Create vector index

To optimize the recall and performance of a vector index, we recommend that you:

  - Create your vector index after most of the rows with embeddings are written to your database. You might also need to periodically rebuild the vector index after you insert new data. For more information, see [Rebuild the vector index](/spanner/docs/vector-index-best-practices#rebuild) .

  - Use the `  STORING  ` clause to store a copy of a column in the vector index. If a column value is stored in the vector index, then Spanner performs filtering at the index's leaf level to improve query performance. We recommend that you store a column if it's used in a filtering condition. For more information about using `  STORING  ` in an index, see [Create an index for index-only scans](/spanner/docs/secondary-indexes#storing-clause) .

When you create your table, the embedding column must be an array of the `  FLOAT32  ` (recommended) or `  FLOAT64  ` data type, and have a *vector\_length* annotation, indicating the dimension of the vectors. The optimal vector length depends on your workload, dataset size, and available computational resources. Experiment with different dimensions to find the smallest size that maintains accuracy and performance for your application.

The following DDL statement creates a `  Documents  ` table with an embedding column `  DocEmbedding  ` with a vector length:

``` text
CREATE TABLE Documents (
  UserId INT64 NOT NULL,
  DocId INT64 NOT NULL,
  Author STRING (1024),
  DocContents Bytes(MAX),
  DocEmbedding ARRAY<FLOAT32>(vector_length=>128) NOT NULL,
  NullableDocEmbedding ARRAY<FLOAT32>(vector_length=>128),
  WordCount INT64,
) PRIMARY KEY (DocId);
```

After you populate your `  Documents  ` table, you can create a vector index with a two-level tree and 1000 leaf nodes on the `  Documents  ` table with an embedding column `  DocEmbedding  ` using the cosine distance:

``` text
CREATE VECTOR INDEX DocEmbeddingIndex
  ON Documents(DocEmbedding)
  STORING (WordCount)
  OPTIONS (distance_type = 'COSINE', tree_depth = 2, num_leaves = 1000);
```

If your embedding column isn't marked as `  NOT NULL  ` in the table definition, you must declare it with a `  WHERE COLUMN_NAME IS NOT NULL  ` clause in the vector index definition, where `  COLUMN_NAME  ` is the name of your embedding column. To create a vector index with a three-level tree and 1000000 leaf nodes on the nullable embedding column `  NullableDocEmbedding  ` using the cosine distance:

``` text
CREATE VECTOR INDEX DocEmbeddingThreeLevelIndex
  ON Documents(NullableDocEmbedding)
  STORING (WordCount)
  WHERE NullableDocEmbedding IS NOT NULL
  OPTIONS (distance_type = 'COSINE', tree_depth = 3, num_branches=1000, num_leaves = 1000000);
```

#### Filter a vector index

You can also create a filtered vector index to find the most similar items in your database that match the filter condition. A filtered vector index selectively indexes rows that satisfy the specified filter conditions, improving search performance.

In the following example, the table `  Documents2  ` has a column called `  Category  ` . In our vector search, we want to index the "Tech" category so we create a generated column that evaluates to `  NULL  ` if the category condition isn't met.

``` text
CREATE TABLE Documents2 (
  DocId INT64 NOT NULL,
  Category STRING(MAX),
  NullIfFiltered BOOL AS (IF(Category = 'Tech', TRUE, NULL)) HIDDEN,
  DocEmbedding ARRAY<FLOAT32>(vector_length=>128),
) PRIMARY KEY (DocId);
```

Then, we create a vector index with a filter. The `  TechDocEmbeddingIndex  ` vector index only indexes documents in the "Tech" category.

``` text
CREATE VECTOR INDEX TechDocEmbeddingIndex
  ON Documents2(DocEmbedding)
  STORING(NullIfFiltered)
  WHERE DocEmbedding IS NOT NULL AND NullIfFiltered IS NOT NULL
  OPTIONS (...);
```

When Spanner runs the following query, which has filters that match the `  TechDocEmbeddingIndex  ` , it automatically picks and is accelerated by `  TechDocEmbeddingIndex  ` . The query only searches documents in the "Tech" category. You can also use `  {@FORCE_INDEX=TechDocEmbeddingIndex}  ` to force Spanner to use `  TechDocEmbeddingIndex  ` explicitly.

``` text
SELECT *
FROM Documents2
WHERE DocEmbedding IS NOT NULL AND NullIfFiltered IS NOT NULL
ORDER BY APPROX_(....)
LIMIT 10;
```

**Note:** In this query, if you replace `  NullIfFiltered IS NOT NULL  ` with `  Category = 'Tech'  ` , then the query won't match the vector index `  TechDocEmbeddingIndex  ` .

## What's next

  - Learn more about Spanner [approximate nearest neighbors](/spanner/docs/find-approximate-nearest-neighbors) .

  - Learn more about the [GoogleSQL `  APPROXIMATE_COSINE_DISTANCE()  ` , `  APPROXIMATE_EUCLIDEAN_DISTANCE()  ` , `  APPROXIMATE_DOT_PRODUCT()  `](/spanner/docs/reference/standard-sql/mathematical_functions) functions.

  - Learn more about the [GoogleSQL `  VECTOR INDEX  ` statements](/spanner/docs/reference/standard-sql/data-definition-language#vector_index_statements) .

  - Learn more about [vector index best practices](/spanner/docs/vector-index-best-practices) .
