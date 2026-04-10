This page describes Spanner global and regional service endpoints and how to use them.

A [**service endpoint**](https://docs.cloud.google.com/apis/design/glossary#api_service_endpoint) is a base URL that specifies the network address of an API service. Spanner has both global and regional endpoints. You can use a global or regional service endpoint to make requests to Spanner.

Use the **global endpoint** if you don't have strict regional restriction requirements. Although the data is stored within the selected region, the data might be processed outside the region. The global endpoint for Spanner is `  spanner.googleapis.com  ` . The default API endpoint accesses the global endpoint.

A **regional endpoint** enforces regional restrictions. Data is stored and processed within the same region. Regional endpoints for Spanner ensure and guarantee that the data stored and processed is restricted to the Spanner regional [instance configuration](https://docs.cloud.google.com/spanner/docs/instance-configurations) where the database resides. Use regional endpoints if your data location must be restricted and controlled to comply with regulatory requirements. You can't use a regional endpoint to access resources belonging to a different instance configuration. You must first create an instance in the regional instance configuration before you can use the regional endpoint for that instance configuration.

To learn which regions you can use, see [Regions available for regional endpoints](https://docs.cloud.google.com/spanner/docs/endpoints#available-regional-endpoints) . The underlying Spanner storage policies don't change regardless of which endpoint you use.

## Security and compliance for regional endpoints

The benefit of using a Spanner regional endpoint over a global endpoint is that the regional endpoint provides regional isolation and protection to meet security, compliance, and regulatory requirements.

You can only use regional endpoints that belong to that regional instance configuration. For example, you can't use `  spanner.me-central2.rep.googleapis.com  ` to serve requests if the instance you are accessing belongs to the `  us-central1  ` regional instance configuration. The request will be rejected with an `  InvalidRegionalRequest  ` error.

## Limitations of regional endpoints

You can't access a dual-region or multi-region instance configuration with a regional endpoint. You must use the global endpoint to access your dual-region or multi-region instance configurations. For example, if you have an instance in the multi-region instance configuration `  nam7  ` , you can't use the regional endpoint `  spanner.us-central1.rep.googleapis.com  ` to send requests to your instance in `  nam7  ` .

If you have active requests that use regional endpoints on any of the instance resources, [moving the instance](https://docs.cloud.google.com/spanner/docs/move-instance) impacts all requests using the regional endpoint because regional enforcement blocks access to cross region instances. Requests using a global endpoint are unaffected.

## Regional endpoint naming convention

Spanner regional endpoint names follow the same naming convention as the regional instance configuration names. The regional endpoint follows the format `  spanner. REGION .rep.googleapis.com  ` . For example, both the regional instance configuration name and regional endpoint name for Dammam are `  me-central2  ` . Therefore, the regional endpoint is `  spanner.me-central2.rep.googleapis.com  ` .

For more information, see [Available regions for regional endpoints](https://docs.cloud.google.com/spanner/docs/endpoints#available-regional-endpoints) .

## Specify a regional endpoint

You can specify a Spanner regional endpoint using the Google Cloud CLI, REST, or RPC API requests.

### gcloud

To specify a regional endpoint and override the global endpoint, run the following command:

    gcloud config set api_endpoint_overrides/spanner https://spanner.REGION.rep.googleapis.com/

Provide the following value:

  - `  REGION  `  
    The [region](https://docs.cloud.google.com/spanner/docs/endpoints#available-regional-endpoints) for which to set a regional endpoint. For example, `  me-central2  ` .

For example, to configure the regional endpoint as `  me-central2  ` , run the following command:

    gcloud config set api_endpoint_overrides/spanner https://spanner.me-central2.rep.googleapis.com/

To reconfigure a regional endpoint to the global endpoint, run:

    gcloud config unset api_endpoint_overrides/spanner

### REST API

The default API endpoint accesses the global endpoint. To use a regional endpoint, configure the endpoint to the address of the regional endpoint using the following pattern:

    https://spanner.REGION.rep.googleapis.com

For example, if you want to enforce data guarantees in the regional instance configuration Dammam ( `  me-central2  ` ), use:

``` 
  https://spanner.me-central2.rep.googleapis.com
```

Refer to [REST API](https://docs.cloud.google.com/spanner/docs/reference/rest) and [Available regions for regional endpoints](https://docs.cloud.google.com/spanner/docs/endpoints#available-regional-endpoints) for more information.

### RPC API

The default API endpoint accesses the global endpoint. To use a regional endpoint, configure the endpoint to the address of the regional endpoint using the following pattern:

    spanner.REGION.rep.googleapis.com

For example, if you want to enforce data guarantees in the regional instance configuration Dammam ( `  me-central2  ` ), use:

    spanner.me-central2.rep.googleapis.com

Refer to [RPC API](https://docs.cloud.google.com/spanner/docs/reference/rpc) and [Available regions for regional endpoints](https://docs.cloud.google.com/spanner/docs/endpoints#available-regional-endpoints) for more information.

## Regions available for regional endpoints

Spanner regional endpoints are available in the following regions:

|                 | Base regional name            | Region description |
| --------------- | ----------------------------- | ------------------ |
| **Americas**    |                               |                    |
|                 | `        us-central1       `  | Iowa               |
|                 | `        us-central2       `  | Oklahoma           |
|                 | `        us-east1       `     | South Carolina     |
|                 | `        us-east4       `     | Northern Virginia  |
|                 | `        us-east5       `     | Columbus           |
|                 | `        us-east7       `     | Alabama            |
|                 | `        us-south1       `    | Dallas             |
|                 | `        us-west1       `     | Oregon             |
|                 | `        us-west2       `     | Los Angeles        |
|                 | `        us-west3       `     | Salt Lake City     |
|                 | `        us-west4       `     | Las Vegas          |
|                 | `        us-west8       `     | Phoenix            |
| **Middle East** |                               |                    |
|                 | `        me-central2       `  | Dammam             |
| **Europe**      |                               |                    |
|                 | `        europe-west8       ` | Milan              |

## Restrict global API endpoint usage

To help enforce the use of regional endpoints, use the `  constraints/gcp.restrictEndpointUsage  ` organization policy constraint to block requests to the global API endpoint. For more information, see [Restrict endpoint usage](https://docs.cloud.google.com/docs/security/compliance/restrict-endpoint-usage) .

## What's next

  - Learn more about Spanner [instance configurations](https://docs.cloud.google.com/spanner/docs/instance-configurations) .

  - Learn more about [Spanner REST API](https://docs.cloud.google.com/spanner/docs/reference/rest) and [Global and regional endpoints](https://docs.cloud.google.com/spanner/docs/rest-endpoints) .

  - Learn more about [Spanner RPC API](https://docs.cloud.google.com/spanner/docs/reference/rpc) and [Global and regional endpoints](https://docs.cloud.google.com/spanner/docs/rpc-endpoints) .
