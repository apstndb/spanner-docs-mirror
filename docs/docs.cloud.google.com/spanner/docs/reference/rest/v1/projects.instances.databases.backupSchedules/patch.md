  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Updates a backup schedule.

### HTTP request

Choose a location:

  
`  PATCH https://spanner.googleapis.com/v1/{backupSchedule.name=projects/*/instances/*/databases/*/backupSchedules/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  backupSchedule.name  `

`  string  `

Identifier. Output only for the `  backupSchedules.create  ` operation. Required for the `  backupSchedules.patch  ` operation. A globally unique identifier for the backup schedule which cannot be changed. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>/backupSchedules/[a-z][a-z0-9_\-]*[a-z0-9]  ` The final segment of the name must be between 2 and 60 characters in length.

### Query parameters

Parameters

`  updateMask  `

`  string ( FieldMask  ` format)

Required. A mask specifying which fields in the BackupSchedule resource should be updated. This mask is relative to the BackupSchedule resource, not to the request message. The field mask must always be specified; this prevents any future fields from being erased accidentally.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .

### Request body

The request body contains an instance of `  BackupSchedule  ` .

### Response body

If successful, the response body contains an instance of `  BackupSchedule  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
