**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to perform a vector similarity search in Spanner by using the cosine distance, Euclidean distance, and dot product vector functions to find K-nearest neighbors. This information applies to both GoogleSQL-dialect databases and PostgreSQL-dialect databases. Before you read this page, it's important that you understand the following concepts:

  - [Euclidean distance](https://en.wikipedia.org/wiki/Euclidean_distance) : measures the shortest distance between two vectors.
  - [Cosine distance](https://en.wikipedia.org/wiki/Cosine_similarity#Cosine_distance) : measures the cosine of the angle between two vectors.
  - [Dot product](https://mathworld.wolfram.com/DotProduct.html) : calculates the cosine of the angle multiplied by the product of corresponding vector magnitudes. If you know that all the vector embeddings in your dataset are normalized, then you can use `  DOT_PRODUCT()  ` as a distance function.
  - [K-nearest neighbors (KNN)](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) : a supervised machine learning algorithm used to solve classification or regression problems.

You can use vector distance functions to perform K-nearest neighbors (KNN) vector search for use cases like similarity search or retrieval-augmented generation. Spanner supports the `  COSINE_DISTANCE()  ` , `  EUCLIDEAN_DISTANCE()  ` , and `  DOT_PRODUCT()  ` functions, which operate on vector embeddings, allowing you to find the KNN of the input embedding.

For example, after you [generate and save your operational Spanner data as vector embeddings](/spanner/docs/ml-tutorial-embeddings) , you can then provide these vector embeddings as an input parameter in your query to find the nearest vectors in N-dimensional space to search for semantically similar or related items.

All three distance functions take the arguments `  vector1  ` and `  vector2  ` , which are of the type `  array<>  ` , and must consist of the same dimensions and have the same length. For more details about these functions, see:

  - [`  COSINE_DISTANCE()  ` in GoogleSQL](/spanner/docs/reference/standard-sql/mathematical_functions#cosine_distance)
  - [`  EUCLIDEAN_DISTANCE()  ` in GoogleSQL](/spanner/docs/reference/standard-sql/mathematical_functions#euclidean_distance)
  - [`  DOT_PRODUCT()  ` in GoogleSQL](/spanner/docs/reference/standard-sql/mathematical_functions#dot_product)
  - [Mathematical functions in PostgreSQL](/spanner/docs/reference/postgresql/functions-and-operators#mathematical) ( `  spanner.cosine_distance()  ` , `  spanner.euclidean_distance()  ` , and `  spanner.dot_product()  ` )
  - [Choose among vector distance functions to measure vector embeddings similarity](/spanner/docs/choose-vector-distance-function) .

## Examples

The following examples show KNN search, KNN search over partitioned data, and using a secondary index with KNN.

The examples all use `  EUCLIDEAN_DISTANCE()  ` . You can also use `  COSINE_DISTANCE()  ` . In addition, if all the vector embeddings in your dataset are normalized, you can use `  DOT_PRODUCT()  ` as a distance function.

### Example 1: KNN search

Consider a `  Documents  ` table that has a column ( `  DocEmbedding  ` ) of precomputed text embeddings from the `  DocContents  ` bytes column.

### GoogleSQL

``` text
CREATE TABLE Documents (
UserId       INT64 NOT NULL,
DocId        INT64 NOT NULL,
Author       STRING(1024),
DocContents  BYTES(MAX),
DocEmbedding ARRAY<FLOAT32>
) PRIMARY KEY (UserId, DocId);
```

### PostgreSQL

``` text
CREATE TABLE Documents (
UserId       bigint NOT NULL,
DocId        bigint NOT NULL,
Author       varchar(1024),
DocContents  bytea,
DocEmbedding float4[],
PRIMARY KEY  (UserId, DocId)
);
```

Assuming that an input embedding for "baseball, but not professional baseball" is the array `  [0.3, 0.3, 0.7, 0.7]  ` , you can find the top five nearest documents that match, with the following query:

### GoogleSQL

``` text
SELECT DocId, DocEmbedding FROM Documents
ORDER BY EUCLIDEAN_DISTANCE(DocEmbedding,
ARRAY<FLOAT32>[0.3, 0.3, 0.7, 0.8])
LIMIT 5;
```

### PostgreSQL

``` text
SELECT DocId, DocEmbedding FROM Documents
ORDER BY spanner.euclidean_distance(DocEmbedding,
'{0.3, 0.3, 0.7, 0.8}'::float4[])
LIMIT 5;
```

The expected results of this example:

``` text
Documents
+---------------------------+-----------------+
| DocId                     | DocEmbedding    |
+---------------------------+-----------------+
| 24                        | [8, ...]        |
+---------------------------+-----------------+
| 25                        | [6, ...]        |
+---------------------------+-----------------+
| 26                        | [3.2, ...]      |
+---------------------------+-----------------+
| 27                        | [38, ...]       |
+---------------------------+-----------------+
| 14229                     | [1.6, ...]      |
+---------------------------+-----------------+
```

### Example 2: KNN search over partitioned data

The query in the previous example can be modified by adding conditions to the `  WHERE  ` clause to limit the vector search to a subset of your data. One common application of this is to search over partitioned data, such as rows that belong to a specific `  UserId  ` .

### GoogleSQL

``` text
SELECT UserId, DocId, DocEmbedding FROM Documents
WHERE UserId=18
ORDER BY EUCLIDEAN_DISTANCE(DocEmbedding,
ARRAY<FLOAT32>[0.3, 0.3, 0.7, 0.8])
LIMIT 5;
```

### PostgreSQL

``` text
SELECT UserId, DocId, DocEmbedding FROM Documents
WHERE UserId=18
ORDER BY spanner.euclidean_distance(DocEmbedding,
'{0.3, 0.3, 0.7, 0.8}'::float4[])
LIMIT 5;
```

The expected results of this example:

``` text
Documents
+-----------+-----------------+-----------------+
| UserId    | DocId           | DocEmbedding    |
+-----------+-----------------+-----------------+
| 18        | 234             | [12, ...]       |
+-----------+-----------------+-----------------+
| 18        | 12              | [1.6, ...]      |
+-----------+-----------------+-----------------+
| 18        | 321             | [22, ...]       |
+-----------+-----------------+-----------------+
| 18        | 432             | [3, ...]        |
+-----------+-----------------+-----------------+
```

### Example 3: KNN search over secondary index ranges

If the `  WHERE  ` clause filter you're using isn't part of the table's primary key, then you can create a secondary index to accelerate the operation with an [index-only scan](/spanner/docs/secondary-indexes#storing-clause) .

### GoogleSQL

``` text
CREATE INDEX DocsByAuthor
ON Documents(Author)
STORING (DocEmbedding);

SELECT Author, DocId, DocEmbedding FROM Documents
WHERE Author="Mark Twain"
ORDER BY EUCLIDEAN_DISTANCE(DocEmbedding,
   <embeddings for "book about the time traveling American">)
LIMIT 5;
```

### PostgreSQL

``` text
CREATE INDEX DocsByAuthor
ON Documents(Author)
INCLUDE (DocEmbedding);

SELECT Author, DocId, DocEmbedding FROM Documents
WHERE Author="Mark Twain"
ORDER BY spanner.euclidean_distance(DocEmbedding,
   <embeddings for "that book about the time traveling American">)
LIMIT 5;
```

The expected results of this example:

``` text
Documents
+------------+-----------------+-----------------+
| Author     | DocId           | DocEmbedding    |
+------------+-----------------+-----------------+
| Mark Twain | 234             | [12, ...]       |
+------------+-----------------+-----------------+
| Mark Twain | 12              | [1.6, ...]      |
+------------+-----------------+-----------------+
| Mark Twain | 321             | [22, ...]       |
+------------+-----------------+-----------------+
| Mark Twain | 432             | [3, ...]        |
+------------+-----------------+-----------------+
| Mark Twain | 375             | [9, ...]        |
+------------+-----------------+-----------------+
```

## What's next

  - Learn more about the [GoogleSQL `  COSINE_DISTANCE()  ` , `  EUCLIDEAN_DISTANCE()  ` , `  DOT_PRODUCT()  `](/spanner/docs/reference/standard-sql/mathematical_functions) functions.

  - Learn more about the [PostgreSQL `  spanner.cosine_distance()  ` , `  spanner.euclidean_distance()  ` , `  spanner.dot_product()  `](/spanner/docs/reference/postgresql/functions-and-operators#mathematical) functions.

  - Learn more about how to [Choose among vector distance functions to measure vector embeddings similarity](/spanner/docs/choose-vector-distance-function) .
