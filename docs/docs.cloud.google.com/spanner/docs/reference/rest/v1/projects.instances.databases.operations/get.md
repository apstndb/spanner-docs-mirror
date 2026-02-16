  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Gets the latest state of a long-running operation. Clients can use this method to poll the operation result at intervals as recommended by the API service.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/operations/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

The name of the operation resource.

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
