**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes how to choose among the vector distance functions provided in Spanner to measure similarity between vector embeddings.

After you've [generated embeddings](/spanner/docs/ml-tutorial-embeddings) from your Spanner data, you can perform a similarity search using vector distance functions. The following table describes the vector distance functions in Spanner.

<table>
<thead>
<tr class="header">
<th>Function</th>
<th>Description</th>
<th>Formula</th>
<th>Relationship to increasing similarity</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Dot product</td>
<td>Calculates the cosine of angle \(\theta\) multiplied by the product of corresponding vector magnitudes.</td>
<td>\(a_1b_1+a_2b_2+...+a_nb_n\) \(=|a||b|cos(\theta)\)</td>
<td>Increases</td>
</tr>
<tr class="even">
<td>Cosine distance</td>
<td>The cosine distance function subtracts the cosine similarity from one ( <code dir="ltr" translate="no">       cosine_distance() = 1 - cosine similarity      </code> ). The cosine similarity measures the cosine of angle \(\theta\) between two vectors.</td>
<td>1 - \(\frac{a^T b}{|a| \cdot |b|}\)</td>
<td>Decreases</td>
</tr>
<tr class="odd">
<td>Euclidean distance</td>
<td>Measures the straight line distance between two vectors.</td>
<td>\(\sqrt{(a_1-b_1)^2+(a_2-b_2)^2+...+(a_N-b_N)^2}\)</td>
<td>Decreases</td>
</tr>
</tbody>
</table>

## Choose a similarity measure

Depending on whether or not all your vector embeddings are normalized, you can determine which similarity measure to use to find similarity. A normalized vector embedding has a magnitude (length) of exactly 1.0.

In addition, if you know which distance function your model was trained with, use that distance function to measure similarity between your vector embeddings.

**Normalized data**

If you have a dataset where all vector embeddings are normalized, then all three functions provide the same semantic search results. In essence, although each function returns a different value, those values sort the same way. When embeddings are normalized, `  DOT_PRODUCT()  ` is usually the most computationally efficient, but the difference is negligible in most cases. However, if your application is highly performance sensitive, `  DOT_PRODUCT()  ` might help with performance tuning.

**Non-normalized data**

If you have a dataset where vector embeddings aren't normalized, then it's not mathematically correct to use `  DOT_PRODUCT()  ` as a distance function because dot product as a function doesn't measure distance. Depending on how the embeddings were generated and what type of search is preferred, either the `  COSINE_DISTANCE()  ` or `  EUCLIDEAN_DISTANCE()  ` function produces search results that are subjectively better than the other function. Experimentation with either `  COSINE_DISTANCE()  ` or `  EUCLIDEAN_DISTANCE()  ` might be necessary to determine which is best for your use case.

**Unsure if data is normalized or non-normalized**

If you're unsure whether or not your data is normalized and you want to use `  DOT_PRODUCT()  ` , we recommend that you use `  COSINE_DISTANCE()  ` instead. `  COSINE_DISTANCE()  ` is like `  DOT_PRODUCT()  ` with normalization built-in. Similarity measured using `  COSINE_DISTANCE()  ` ranges from `  0  ` to `  2  ` . A result that is close to `  0  ` indicates the vectors are very similar.

## What's next

  - Learn more about how to [perform a vector search by finding the k-nearest neighbor](/spanner/docs/find-k-nearest-neighbors) .
  - Learn how to [export embeddings to Vertex AI Vector Search](/spanner/docs/vector-search-embeddings) .
  - Learn more about the [GoogleSQL `  COSINE_DISTANCE()  ` , `  EUCLIDEAN_DISTANCE()  ` , and `  DOT_PRODUCT()  `](/spanner/docs/reference/standard-sql/mathematical_functions) functions.
  - Learn more about the [PostgreSQL `  spanner.cosine_distance()  ` , `  spanner.euclidean_distance(), and spanner.dot_product()  `](/spanner/docs/reference/postgresql/functions-and-operators#mathematical) functions.
