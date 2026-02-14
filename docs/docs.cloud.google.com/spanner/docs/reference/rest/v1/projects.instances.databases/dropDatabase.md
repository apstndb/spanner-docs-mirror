  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Drops (aka deletes) a Cloud Spanner database. Completed backups for the database will be retained according to their `  expireTime  ` . Note: Cloud Spanner might continue to accept requests for a few seconds after the database has been deleted.

### HTTP request

Choose a location:

  
`  DELETE https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  database  `

`  string  `

Required. The database to be dropped.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.databases.drop  `

### Request body

The request body must be empty.

### Response body

If successful, the response body is an empty JSON object.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
