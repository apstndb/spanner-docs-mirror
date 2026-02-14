This page describes Spanner RPC global and regional endpoints. You can use a global or regional service endpoint to make requests to Spanner.

Use a [regional endpoint](/spanner/docs/endpoints) to enforce regional restriction and ensure that your data is stored and processed within the same region.

The Spanner **RPC global endpoint** is `  spanner.googleapis.com  ` .

The **RPC regional endpoint** follows the format `  spanner.<REGION>.rep.googleapis.com  ` . For example, if you want to enforce data guarantees in the regional instance configuration Dammam ( `  me-central2  ` ), use `  spanner.me-central2.rep.googleapis.com  ` as your regional endpoints. For more information, see [Global and regional service endpoints](/spanner/docs/endpoints) .

Only `  me-central2  ` is available as a possible regional endpoint.
