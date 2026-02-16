This page describes Spanner REST global and regional service endpoints. You can use a global or regional service endpoint to make requests to Spanner.

Use a [regional endpoint](/spanner/docs/endpoints) to enforce regional restriction and ensure that your data is stored and processed within the same region.

The Spanner **REST global endpoint** is `  https://spanner.googleapis.com  ` .

The **REST regional endpoint** follows the format `  https://spanner.<REGION>.rep.googleapis.com  ` . For example, if you want to enforce data guarantees in the regional instance configuration Dammam ( `  me-central2  ` ), use `  https://spanner.me-central2.rep.googleapis.com  ` as your regional endpoint. For more information, see [Global and regional service endpoints](/spanner/docs/endpoints) .

Only `  me-central2  ` is available as a possible regional endpoint.
