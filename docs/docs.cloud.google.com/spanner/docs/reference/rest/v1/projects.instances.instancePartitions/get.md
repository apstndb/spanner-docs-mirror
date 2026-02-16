  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Gets information about a particular instance partition.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/instancePartitions/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The name of the requested instance partition. Values are of the form `  projects/{project}/instances/{instance}/instancePartitions/{instancePartition}  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instancePartitions.get  `

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of `  InstancePartition  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
