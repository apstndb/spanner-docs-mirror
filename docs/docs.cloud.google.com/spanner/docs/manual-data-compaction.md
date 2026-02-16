**Preview — [Spanner columnar engine](/spanner/docs/columnar-engine)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This document explains how to manually trigger a major compaction in a Spanner database.

Several storage-related features in Spanner, such as tier storage or columnar engine, rely on a database-wide major compaction before they are fully enabled. By default, major compactions occur automatically across all tables over seven day periods. This means you might wait up to seven days for a new feature to be fully available. To make new features available immediately, you can manually trigger a major compaction.

The compaction process is a [long-running operation (LRO)](/spanner/docs/manage-and-observe-long-running-operations#check_the_progress_of_a_long-running_database_operation) .

## Pricing

Triggering a major compaction temporarily increases [compute capacity](/spanner/pricing#compute-capacity) on the Spanner instance. This might result in increased costs.

## Performance

Major compactions run as background operations. However, if your instance has consistently heavy CPU usage, the compaction workload might interfere with other critical operations. In such cases, you can temporarily scale up the instance to ensure stable performance during compaction.

## Manually trigger a major compaction

### Google Cloud console

1.  Open the Google Cloud console and select your instance.

2.  Select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .

5.  Invoke the following command to initiate compaction:
    
    ``` text
    CALL compact_all();
    ```
    
    This operation returns a long-running operation (LRO) ID that you can use to find the operation in the **Operations** list.

6.  To monitor the progress of the compaction operation, in the navigation menu, click **Operations** .

### C++

To trigger compactions programmatically using the C++ client library:

``` text
void Compact(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  spanner::SqlStatement select("CALL compact_all()");
  auto rows = client.ExecuteQuery(statement);
  using RowType = std::tuple<std::string>;
  auto rows = client.ExecuteQuery(std::move(select));

  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "Long-running operation ID: " << std::get<0>(*row) << "\n";
  }
}
```

You can [check the progress of a long-running database operation](/spanner/docs/manage-and-observe-long-running-operations#check_the_progress_of_a_long-running_database_operation) . You can also cancel the ongoing major compaction request using the LRO ID. For more information, see [Cancel a long-running database operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_database_operation) .
