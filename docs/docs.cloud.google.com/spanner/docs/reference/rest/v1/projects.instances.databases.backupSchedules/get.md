  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Gets backup schedule for the input schedule name.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/backupSchedules/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The name of the schedule to retrieve. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>/backupSchedules/<backupScheduleId>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.backupSchedules.get  `

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of `  BackupSchedule  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
