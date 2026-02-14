This page describes how to conduct full-text and vector hybrid searches in Spanner. Hybrid search combines the precision of keyword matching (full-text search, FTS) with the recall of semantic matching (vector search) to produce highly relevant search results.

Spanner supports the following hybrid search patterns, which are divided into three main categories:

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Description</td>
<td>Primary Goal</td>
</tr>
<tr class="even">
<td><strong><a href="#fusion-search">Fusion</a></strong></td>
<td>Fusion independently retrieves and ranks documents using keyword and vector search, then combines (fuses) the results.</td>
<td>Achieve maximum recall and relevance by combining multiple scoring signals.</td>
</tr>
<tr class="odd">
<td><strong><a href="#filtered-search">Filtered</a></strong></td>
<td>Keywords filter or refine the search space.</td>
<td>Ensure keyword matching is a requirement while leveraging semantic relevance.</td>
</tr>
<tr class="even">
<td><strong><a href="#ml-reranking">ML reranking</a></strong></td>
<td>A machine learning model refines an initial set of candidates for a final, more precise ranking.</td>
<td>Achieve the highest possible precision for a small final set of results.</td>
</tr>
</tbody>
</table>

## Fusion search

Fusion search involves running FTS and vector search independently on the same data corpus. It then merges the results to create a single, unified, and highly relevant ranked list.

While you can execute independent queries on the client side, hybrid search in Spanner provides the following advantages:

  - Simplifies application logic by managing parallel requests and result merging on the server.
  - Avoids transferring potentially large intermediate result sets to the client.

You can use SQL to create fusion methods in Spanner. This section provides examples of [reciprocal rank fusion](#rank-based-fusion) and [relative score fusion](#score-based-fusion) . However, we recommend evaluating different fusion strategies to determine which best suits your application's requirements.

### Rank-based fusion

Use rank-based fusion when the relevance scores from different retrieval methods (such as FTS scores and vector distances) are difficult to compare or normalize because they are measured in different spaces. This method uses the rank position of each document from each retriever to generate a final score and ranking.

*Reciprocal rank fusion (RRF)* is a rank-based fusion function. The RRF score for a document is the sum of the reciprocals of its ranks from multiple retrievers is calculated as follows:

\\\[ score\\ (d\\epsilon D)\\ =\\ \\sum\\limits\_{r\\epsilon R}^{}(1\\ /\\ (\\ 𝑘\\ +\\ ran{k}\_{r}\\ (𝑑)\\ )\\ )\\ \\\]

Where:

  - *d* is the document.
  - *R* is the set of retrievers (FTS and vector search).
  - *k* is a constant (often set to 60) used to moderate the influence of very high-ranked documents.
  - *w* is the rank of the document from retriever *r* .

#### Implement RRF in a Spanner query

Vector metrics (such as [`  APPROX_DOT_PRODUCT  `](/spanner/docs/reference/standard-sql/mathematical_functions#approx_dot_product) ) and text search scores (such as [`  SCORE_NGRAMS  `](/spanner/docs/reference/standard-sql/search_functions#score_ngrams) ) operate on incompatible scales. To resolve this, the following example implements RRF to normalize data based on rank position rather than raw scores.

The query uses `  UNNEST(ARRAY(...) WITH OFFSET  ` ) to assign a rank to the top 100 candidates from each method. It then calculates a standardized score using the inverse of those rankings and aggregates the results to return the top five matches.

``` text
SELECT SUM(1 / (60 + rank)) AS rrf_score, key
FROM (
  (
    SELECT rank, x AS key
    FROM UNNEST(ARRAY(
      SELECT key
      FROM hybrid_search
      WHERE embedding IS NOT NULL
      ORDER BY APPROX_DOT_PRODUCT(@vector, embedding,
        OPTIONS => JSON '{"num_leaves_to_search": 50}') DESC
      LIMIT 100)) AS x WITH OFFSET AS rank
  )
  UNION ALL
  (
    SELECT rank, x AS key
    FROM UNNEST(ARRAY(
      SELECT key
      FROM hybrid_search
      WHERE SEARCH_NGRAMS(text_tokens_ngrams, 'foo')
      ORDER BY SCORE_NGRAMS(text_tokens_ngrams, 'foo') DESC
      LIMIT 100)) AS x WITH OFFSET AS rank
  )
)
GROUP BY key
ORDER BY rrf_score DESC
LIMIT 5;
```

### Score-based fusion

Score-based fusion is effective when the relevance scores from different methods are comparable or you can normalize them, which might enable a more precise ranking that incorporates the actual relevance weight of each method.

*Relative score fusion (RSF)* is a score-based method that normalizes scores from different methods relative to the highest and lowest scores within each method, typically using the `  MIN()  ` and `  MAX()  ` functions. The RSF score of a document retrieved by a set of retrievers is calculated as follows:

\\\[ score(d\\epsilon D)\\ =\\ \\sum\\limits\_{r\\epsilon R}^{}({w}\_{r}\*(scor{e}\_{r}(d)\\ -\\ mi{n}\_{r})\\ /\\ (ma{x}\_{r\\ }-mi{n}\_{r})\\ ) \\\]

Where:

  - *d* is the document.
  - *R* is the set of retrievers (FTS and vector search).
  - *w* is the weight assigned to a specific retriever.

#### Implement RSF in a Spanner query

To implement RSF, you must normalize the scores from the vector and FTS to a common scale. The following example calculates the minimum and maximum scores in separate `  WITH  ` clauses to derive the normalization factors. It then combines the results using a `  FULL OUTER JOIN  ` , summing the normalized scores from the approximate nearest neighbor (ANN) search (converted from `  cosine_distance  ` ) and the FTS.

``` text
WITH ann AS (
  SELECT key, APPROX_COSINE_DISTANCE(@vector, embedding,
    OPTIONS => JSON '{"num_leaves_to_search": 50}') AS cosine_distance,
  FROM hybrid_search
  WHERE embedding IS NOT NULL
  ORDER BY cosine_distance
  LIMIT 100
),
fts AS (
  SELECT key, SCORE_NGRAMS(text_tokens_ngrams, 'Green') AS score,
  FROM hybrid_search
    WHERE SEARCH_NGRAMS(text_tokens_ngrams, 'Green')
    ORDER BY score DESC
    LIMIT 100
),
ann_min AS (
  SELECT MIN(1 - cosine_distance) AS min
  FROM ann
),
ann_max AS (
  SELECT MAX(1 - cosine_distance) AS max
  FROM ann
),
fts_min AS (
  SELECT MIN(score) AS min
  FROM fts
),
fts_max AS (
  SELECT MAX(score) AS max
  FROM fts
)
SELECT IFNULL(ann.key, fts.key),
       IFNULL(((1 - ann.cosine_distance) - ann_min.min) /
               (ann_max.max - ann_min.min), 0) +
       IFNULL((fts.score - fts_min.min) /
               (fts_max.max - fts_min.min), 0) AS score
FROM ann
FULL OUTER JOIN fts
ON ann.key = fts.key
CROSS JOIN ann_min
CROSS JOIN ann_max
CROSS JOIN fts_min
CROSS JOIN fts_max
ORDER BY score DESC
LIMIT 5;
```

## Filtered search

Filtered searches use FTS to create a filter that reduces the set of documents considered for a k-nearest neighbors (KNN) vector search. You can optionally use a presort to limit the size of the result set.

### Implement filtered search in a Spanner query

The example search in this section takes the following steps to narrow down the vector search space to the subset of data that match the keywords:

  - Uses `  SEARCH (text_tokens, 'Green')  ` to find rows where the `  text_tokens  ` column contains the text `  Green  ` . The top 1000 rows are returned by a [resort order defined by the search index](/spanner/docs/full-text-search/search-indexes#search-index-sort-order) .
  - Uses a vector function, `  DOT_PRODUCT(@vector, embedding)  ` to calculate the similarity between the query vector (@vector) and the stored document vector (embedding). It then sorts the results and returns the final top 10 matches.

<!-- end list -->

``` text
SELECT key
FROM (
  SELECT key, embedding
  FROM hybrid_search
  WHERE SEARCH (text_tokens, 'Green')
  ORDER BY presort
  LIMIT 1000)
ORDER BY DOT_PRODUCT(@vector, embedding) DESC
LIMIT 10;
```

## ML reranking

ML-based reranking is a computationally intensive but highly precise approach. It applies a machine learning model to a small candidate set that has been reduced by FTS, vector search, or a combination of both. For more information about Spanner Vertex AI integration, see the [Spanner Vertex AI integration overview](/spanner/docs/ml) .

You can integrate ML reranking using the Spanner [`  ML.PREDICT  `](/spanner/docs/reference/standard-sql/ml-functions#mlpredict) function with a deployed Vertex AI model.

### Implement ML-based reranking

1.  [Deploy a reranker model](/vertex-ai/docs/general/deployment) (such as from [HuggingFace](https://huggingface.co/) ) to a Vertex AI endpoint.

2.  [Create a Spanner `  MODEL  ` object](/spanner/docs/reference/standard-sql/data-definition-language#create_model) that points to the Vertex AI endpoint. For example, in the following `  Reranker  ` model example:
    
      - `  text ARRAY<string(max)>  ` is the documents.
      - `  text_pair ARRAY<string(max)>  ` is the query text in the example.
      - `  score  ` is the relevance scores assigned by the ML model for the input pairs of texts.
    
    <!-- end list -->
    
    ``` text
    CREATE MODEL Reranker
    INPUT (text ARRAY<string(max)>, text_pair ARRAY<string(max)>)
    OUTPUT (score FLOAT32)
    REMOTE
    OPTIONS (
    endpoint = '...'
    );
    ```

3.  Use a subquery to retrieve the initial candidates (such as the top 100 results from an ANN search) and pass them to `  ML.PREDICT  ` . The function returns a new score column for the final ordering.
    
    ``` text
    SELECT key
    FROM ML.PREDICT(
        MODEL Reranker, (
          SELECT key, text, "gift for 8-year-old" AS text_pair
          FROM hybrid_search
          WHERE embedding IS NOT NULL
          ORDER BY APPROX_DOT_PRODUCT(@vector, embedding, options=>JSON '{"num_leaves_to_search": 50}') DESC
          LIMIT 100)
    )
    ORDER BY score DESC
    LIMIT 3;
    ```

## What's next

  - [Tailor your search engine with AI-powered hybrid search in Spanner](https://cloud.google.com/blog/topics/developers-practitioners/hybrid-search-in-spanner-combine-full-text-and-vector-search) .
  - Learn more about [full-text search](/spanner/docs/full-text-search) .
