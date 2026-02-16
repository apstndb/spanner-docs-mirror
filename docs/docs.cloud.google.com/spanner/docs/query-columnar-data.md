**Preview — [Spanner columnar engine](/spanner/docs/columnar-engine)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This page describes how to run queries against columnar data.

## Query columnar data

Spanner query engine analyzes queries and automatically selects the columnar format if it is beneficial. However, several classes of queries might still need the query hint to use the columnar format. The following are examples of how to use the `  @{scan_method=columnar}  ` query hint:

  - `  @{scan_method=columnar} SELECT COUNT(*) FROM Singers;  `
  - `  SELECT COUNT(*) FROM Singers @{scan_method=columnar};  `
  - `  @{scan_method=columnar} SELECT m.MsgBlob FROM Messages WHERE m.id='1234';  `

In addition, you can disable the automatic selection of columnar explicitly by using the query hint `  @{scan_method=no_columnar}  ` .

## Query Spanner columnar data using BigQuery federated queries

To read Spanner columnar data from BigQuery, you can either create an [external dataset](/bigquery/docs/spanner-external-datasets#create_an_external_dataset) or use the [`  EXTERNAL_QUERY  `](/bigquery/docs/reference/standard-sql/federated_query_functions#external_query) function.

When you query external datasets, columnar data is automatically used if it's available and suitable for your query.

If you use the `  EXTERNAL_QUERY  ` function, Spanner automatically uses the columnar data if it's available and appropriate for the workload. You can also include the `  @{scan_method=columnar}  ` hint in the nested Spanner query.

In the following example for using the query hint:

  - The first argument to `  EXTERNAL_QUERY  ` specifies the external connection and dataset, `  my-project.us.albums  ` .
  - The second argument is a SQL query that selects `  MarketingBudget  ` from the `  AlbumInfo  ` table where `  MarketingBudget  ` is less than 500,000.
  - The `  @{scan_method=columnar}  ` hint optimizes the external query for columnar scanning.
  - The outer `  SELECT  ` statement calculates the sum of the `  MarketingBudget  ` values returned by the external query.
  - The `  AS total_marketing_spend  ` clause assigns an alias to the calculated sum.

<!-- end list -->

``` text
SELECT SUM(MarketingBudget) AS total_marketing_spend
FROM
  EXTERNAL_QUERY(
    'my-project.us.albums',
    '@{scan_method=columnar} SELECT AlbumInfo.MarketingBudget FROM AlbumInfo WHERE AlbumInfo.MarketingBudget < 500000;');
```

## What's next

  - Learn about [columnar engine](/spanner/docs/columnar-engine) .
  - Learn how to [enable columnar engine](/spanner/docs/configure-columnar-engine) .
  - Learn how to [monitor columnar engine](/spanner/docs/monitor-columnar-engine) .
