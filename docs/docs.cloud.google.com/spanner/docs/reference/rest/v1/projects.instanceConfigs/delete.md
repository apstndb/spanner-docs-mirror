  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Deletes the instance configuration. Deletion is only allowed when no instances are using the configuration. If any instances are using the configuration, returns `  FAILED_PRECONDITION  ` .

Only user-managed configurations can be deleted.

Authorization requires `  spanner.instanceConfigs.delete  ` permission on the resource `  name  ` .

### HTTP request

Choose a location:

  
`  DELETE https://spanner.googleapis.com/v1/{name=projects/*/instanceConfigs/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The name of the instance configuration to be deleted. Values are of the form `  projects/<project>/instanceConfigs/<instanceConfig>  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instanceConfigs.delete  `

### Query parameters

Parameters

`  etag  `

`  string  `

Used for optimistic concurrency control as a way to help prevent simultaneous deletes of an instance configuration from overwriting each other. If not empty, the API only deletes the instance configuration when the etag provided matches the current status of the requested instance configuration. Otherwise, deletes the instance configuration without checking the current status of the requested instance configuration.

`  validateOnly  `

`  boolean  `

An option to validate, but not actually execute, a request, and provide the same response.

### Request body

The request body must be empty.

### Response body

If successful, the response body is an empty JSON object.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
