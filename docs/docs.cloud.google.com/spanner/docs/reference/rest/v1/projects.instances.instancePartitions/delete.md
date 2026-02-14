  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Deletes an existing instance partition. Requires that the instance partition is not used by any database or backup and is not the default instance partition of an instance.

Authorization requires `  spanner.instancePartitions.delete  ` permission on the resource `  name  ` .

### HTTP request

Choose a location:

  
`  DELETE https://spanner.googleapis.com/v1/{name=projects/*/instances/*/instancePartitions/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The name of the instance partition to be deleted. Values are of the form `  projects/{project}/instances/{instance}/instancePartitions/{instancePartition}  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instancePartitions.delete  `

### Query parameters

Parameters

`  etag  `

`  string  `

Optional. If not empty, the API only deletes the instance partition when the etag provided matches the current status of the requested instance partition. Otherwise, deletes the instance partition without checking the current status of the requested instance partition.

### Request body

The request body must be empty.

### Response body

If successful, the response body is an empty JSON object.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
