  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Gets a session. Returns `  NOT_FOUND  ` if the session doesn't exist. This is mainly useful for determining whether a session is still alive.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/sessions/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The name of the session to retrieve.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.sessions.get  `

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of `  Session  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
