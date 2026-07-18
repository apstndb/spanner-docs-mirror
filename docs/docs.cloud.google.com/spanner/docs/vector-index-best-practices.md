---
name: documents/docs.cloud.google.com/spanner/docs/vector-index-best-practices
uri: https://docs.cloud.google.com/spanner/docs/vector-index-best-practices
title: Vector indexing best practices
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This page describes vector indexing best practices that optimize your [vector indexes](https://docs.cloud.google.com/spanner/docs/vector-indexes) and improve [approximate nearest neighbor (ANN) query results](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors#query-vector-embeddings) .

## Tune the vector search options

If you don't specify any vector index options, Spanner attempts to choose optimized options automatically. For advanced users who want to tune the vector index options for their specific workload, you can set and tune these values by creating a new vector index and setting the [`index_option_list`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#index_option_list) in the `CREATE VECTOR INDEX` statement. The most optimal values for your vector index options depend on your use case, vector dataset, and the query vectors. You might need to perform iterative tuning to find the best values for your specific workload.

Here are some helpful guidelines to follow when picking appropriate values:

  - `tree_depth` (tree level): If the table you're indexing has fewer than 10 million rows, use a `tree_depth` of `2` . Otherwise, a `tree_depth` of `3` supports tables of up to about 10 billion rows. If not specified, Spanner will automatically determine the value for `tree_depth` .

  - `num_leaves` : We recommend targeting 200-1000 rows per leaf. A larger value of `num_leaves` increases the vector index build time, but can reduce the query cost for a given target recall. However, overly large values of `num_leaves` can cause query cost to increase again due to overheads associated with searching many leaf clusters that are very small. If not specified, Spanner automatically determines the value for `num_leaves` .

  - `num_branches` : This option is only applicable when `tree_depth` is 3. We recommend targeting 50-500 leaves per branch, and `num_branches` should be less than `num_leaves` . A larger value of `num_branches` increases vector index build time, but can reduce query cost for a given target recall. However, overly large values of `num_branches` can cause query cost to increase again due to overheads associated with searching many leaf clusters that are very small. If not specified, Spanner automatically determines the value for `num_branches` .

  - `num_leaves_to_search` : This option specifies how many leaf nodes of the index are searched. Increasing `num_leaves_to_search` improves recall but also increases latency and cost. We recommend using a number that is 1% the total number of leaves defined in the `CREATE VECTOR INDEX` statement as the value for `num_leaves_to_search` . If you're using a filter clause, increase this value to widen the search.

If acceptable recall is achieved, but the cost of querying is too high, resulting in low maximum QPS, try increasing `num_leaves` by following these steps:

1.  Set `num_leaves` to some multiple `k` of its original value (for example, `2 * (table_row_count / 1000)` ).
2.  Set `num_leaves_to_search` to be the same multiple k of its original value.
3.  Experiment with reducing `num_leaves_to_search` to improve cost and QPS while maintaining recall.

### Determine vector search option values

To determine the `tree_depth` , `num_leaves` , and `num_branches` parameters that Spanner is using for the vector index, query the `INFORMATION_SCHEMA.INDEX_OPTIONS` view. The parameter values might differ slightly from any values you explicitly specified during index creation because Spanner sometimes adjusts them to better suit your data. If you didn't specify these parameters, the `INFORMATION_SCHEMA.INDEX_OPTIONS` view shows the values that Spanner chose.

Run this query to display the parameter values:

    SELECT
      opt.option_name,
      opt.option_type,
      opt.option_value
    FROM
      INFORMATION_SCHEMA.INDEX_OPTIONS AS opt
    WHERE
      opt.index_name = @vector_index_name;

The query returns rows including `option_name` : `system_optimized_tree_depth` , `system_optimized_num_leaves` , and `system_optimized_num_branches` , which reflect parameters used by the index.

## Improve recall

To improve recall, consider tuning the `num_leaves_to_search` value or rebuilding your vector index.

### Increase the `num_leaves_to_search` value

If the `num_leaves_to_search` value is too small, you might find it more challenging to find the nearest neighbors for some query vectors. Creating a new vector index with an increased `num_leaves_to_search` value can help improve recall by searching more leaves. Recent queries might contain more of these challenging vectors.

### Rebuild the vector index

The tree structure of the vector index is optimized for the dataset at the time of creation, and is static thereafter. Therefore, if significantly different vectors are added after creating the initial vector index, then the tree structure might be sub-optimal, leading to poorer recall.

To rebuild your vector index without downtime:

1.  Create a new vector index on the same embedding column as the current vector index, updating parameters (for example, `OPTIONS` ) as appropriate. After the index creation completes, you might consider evaluating which of your two indexes performs better. If so, then proceed to the next step. Otherwise, proceed to dropping the outdated vector index.

2.  Spanner automatically decides which index to use in the query's execution. Spanner provides two ways that let you specify the index to be used. Choose one of the following methods to evaluate and compare your indexes:
    
    a. Change your application: You can update some subset of your queries so that they use the [`FORCE_INDEX` hint](https://docs.cloud.google.com/spanner/docs/secondary-indexes#index-directive) to point at the new index to update the vector search query. This ensures that the query uses the new vector index. Using this method, you might need to retune `num_leaves_to_search` in your new query.
    
    b. Change your schema: You can set the `disable_search` option on one of your vector indexes. When set to `true` , Spanner disables the vector index. You can do this by running the `ALTER VECTOR INDEX` schema change statement:
    
    ``` 
      ALTER VECTOR INDEX IncidentVectorIndex SET OPTIONS (disable_search=true);
    ```
    
    This method prevents Spanner from using this vector index in your database. If you have two indexes and set this option on the older index, all queries use the new index after the schema change applies. If you use the `FORCE_INDEX` hint to specify a vector index which has the `disable_search` option set to `true` , the query fails.

3.  Drop the outdated vector index.

## What's next

  - Learn more about Spanner [vector indexes](https://docs.cloud.google.com/spanner/docs/vector-indexes) .

  - Learn more about Spanner [approximate nearest neighbors](https://docs.cloud.google.com/spanner/docs/find-approximate-nearest-neighbors) .

  - Learn more about the [GoogleSQL `APPROXIMATE_COSINE_DISTANCE()` , `APPROXIMATE_EUCLIDEAN_DISTANCE()` , `APPROXIMATE_DOT_PRODUCT()`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions) functions.

  - Learn more about the [GoogleSQL `VECTOR INDEX` statements](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#vector_index_statements) .
