  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Deletes an instance.

Immediately upon completion of the request:

  - Billing ceases for all of the instance's reserved resources.

Soon afterward:

  - The instance and *all of its databases* immediately and irrevocably disappear from the API. All data in the databases is permanently deleted.

### HTTP request

Choose a location:

  
`  DELETE https://spanner.googleapis.com/v1/{name=projects/*/instances/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The name of the instance to be deleted. Values are of the form `  projects/<project>/instances/<instance>  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instances.delete  `

### Request body

The request body must be empty.

### Response body

If successful, the response body is an empty JSON object.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
