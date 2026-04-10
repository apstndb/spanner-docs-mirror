  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#body.PATH_PARAMETERS)
  - [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#body.QUERY_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#body.request_body)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#body.response_body)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#try-it)

Updates a Cloud Spanner database. The returned long-running operation can be used to track the progress of updating the database. If the named database does not exist, returns `  NOT_FOUND  ` .

While the operation is pending:

  - The database's `  reconciling  ` field is set to true.
  - Cancelling the operation is best-effort. If the cancellation succeeds, the operation metadata's `  cancelTime  ` is set, the updates are reverted, and the operation terminates with a `  CANCELLED  ` status.
  - New databases.patch requests will return a `  FAILED_PRECONDITION  ` error until the pending operation is done (returns successfully or with error).
  - Reading the database via the API continues to give the pre-request values.

Upon completion of the returned operation:

  - The new values are in effect and readable via the API.
  - The database's `  reconciling  ` field becomes false.

The returned long-running operation will have a name of the format `  projects/<project>/instances/<instance>/databases/<database>/operations/<operationId>  ` and can be used to track the database modification. The metadata field type is `  UpdateDatabaseMetadata  ` . The response field type is `  Database  ` , if successful.

### HTTP request

Choose a location:

global

europe-west8

me-central2

us-central1

us-central2

us-east1

us-east4

us-east5

us-south1

us-west1

us-west2

us-west3

us-west4

us-west8

us-east7

  
`  PATCH https://spanner.googleapis.com/v1/{database.name=projects/*/instances/*/databases/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  database.name  `

`  string  `

Required. The name of the database. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` , where `  <database>  ` is as specified in the `  CREATE DATABASE  ` statement. This name can be passed to other API methods to identify the database.

### Query parameters

Parameters

`  updateMask  `

`  string ( FieldMask  ` format)

Required. The list of fields to update. Currently, only `  enableDropProtection  ` field can be updated.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .

### Request body

The request body contains an instance of `  Database  ` .

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
