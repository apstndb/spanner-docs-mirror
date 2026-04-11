  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/get#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/get#body.PATH_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/get#body.request_body)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/get#body.response_body)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/get#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/get#try-it)

Gets the state of a Cloud Spanner database.

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

  
`GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*}`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`name`

`string`

Required. The name of the requested database. Values are of the form `projects/<project>/instances/<instance>/databases/<database>` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `name` :

  - `spanner.databases.get`

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of `  Database  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.admin`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
