  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [IAM Permissions](#body.aspect_1)
  - [Try it\!](#try-it)

Starts creating a new Cloud Spanner Backup. The returned backup long-running operation will have a name of the format `  projects/<project>/instances/<instance>/backups/<backup>/operations/<operationId>  ` and can be used to track creation of the backup. The metadata field type is `  CreateBackupMetadata  ` . The response field type is `  Backup  ` , if successful. Cancelling the returned operation will stop the creation and delete the backup. There can be only one pending backup creation per database. Backup creation of different databases can run concurrently.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/backups  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the instance in which the backup is created. This must be the same instance that contains the database the backup is created from. The backup will be stored in the locations specified in the instance configuration of this instance. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.backups.create  `

### Query parameters

Parameters

`  backupId  `

`  string  `

Required. The id of the backup to be created. The `  backupId  ` appended to `  parent  ` forms the full backup name of the form `  projects/<project>/instances/<instance>/backups/<backupId>  ` .

`  encryptionConfig  `

`  object ( CreateBackupEncryptionConfig  ` )

Optional. The encryption configuration used to encrypt the backup. If this field is not specified, the backup will use the same encryption configuration as the database by default, namely `  encryptionType  ` = `  USE_DATABASE_ENCRYPTION  ` .

### Request body

The request body contains an instance of `  Backup  ` .

### Response body

If successful, the response body contains a newly created instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

### IAM Permissions

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `  database  ` resource:

  - `  spanner.databases.createBackup  `

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `  parent  ` resource:

  - `  spanner.backups.create  `

For more information, see the [IAM documentation](https://cloud.google.com/iam/docs) .
