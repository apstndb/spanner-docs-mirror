  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates a new backup schedule.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/backupSchedules  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the database that this backup schedule applies to.

Authorization requires one or more of the following [IAM](https://cloud.google.com/iam/docs/) permissions on the specified resource `  parent  ` :

  - `  spanner.backupSchedules.create  `
  - `  spanner.databases.createBackup  `

### Query parameters

Parameters

`  backupScheduleId  `

`  string  `

Required. The Id to use for the backup schedule. The `  backupScheduleId  ` appended to `  parent  ` forms the full backup schedule name of the form `  projects/<project>/instances/<instance>/databases/<database>/backupSchedules/<backupScheduleId>  ` .

### Request body

The request body contains an instance of `  BackupSchedule  ` .

### Response body

If successful, the response body contains a newly created instance of `  BackupSchedule  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
