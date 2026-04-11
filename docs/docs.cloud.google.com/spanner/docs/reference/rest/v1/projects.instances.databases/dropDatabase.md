  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#body.PATH_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#body.request_body)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#body.response_body)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#try-it)

Drops (aka deletes) a Cloud Spanner database. Completed backups for the database will be retained according to their `expireTime` . Note: Cloud Spanner might continue to accept requests for a few seconds after the database has been deleted.

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

  
`DELETE https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`database`

`string`

Required. The database to be dropped.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `database` :

  - `spanner.databases.drop`

### Request body

The request body must be empty.

### Response body

If successful, the response body is an empty JSON object.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.admin`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
