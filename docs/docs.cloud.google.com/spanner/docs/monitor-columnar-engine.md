**Preview — [Spanner columnar engine](/spanner/docs/columnar-engine)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This page describes how to monitor Spanner's columnar engine. You can monitor using:

  - [The query execution plan](#columnar-data-query-plan)
  - [Storage usage](#monitor-storage-usage) graphs in [system insights](#monitor-storage-usage)

## View columnar data in the query plan

1.  Run a query.

2.  View its [query plan](/spanner/docs/query-execution-plans) .

3.  In the **Query summary** section of the query execution, check for the **Columnar read share** information.
    
    The following image shows the **Columnar read share** percentage:

This metric shows the percentage of bytes read from columnar storage out of total bytes read from both row-based and columnar storage. A high percentage is optimal, while a low percentage suggests that much of the data remains unconverted to columnar format.

## Columnar data storage metrics

To view columnar data storage metrics:

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Select the instance you want to view metrics for.

3.  Click **System insights** .

4.  In the **Storage usage** section, you can see two new metrics:
    
    **columnar storage - hdd** and **columnar storage - ssd** . These metrics show the columnar representation usage of HDD and SSD. The total storage metrics include both PAX and columnar data. You can see these metrics on storage usage graphs on the **System insights** page. For example:

## What's next

  - Learn about [columnar engine](/spanner/docs/columnar-engine) .
  - Learn how to [enable columnar engine](/spanner/docs/configure-columnar-engine) .
  - Learn how to [query columnar data](/spanner/docs/query-columnar-data) .
