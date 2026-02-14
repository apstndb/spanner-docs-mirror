  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [IAM Permissions](#body.aspect_1)
  - [Try it\!](#try-it)

Updates a pending or completed `  Backup  ` .

### HTTP request

Choose a location:

  
`  PATCH https://spanner.googleapis.com/v1/{backup.name=projects/*/instances/*/backups/*}  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  backup.name  `

`  string  `

Output only for the `  backups.create  ` operation. Required for the `  backups.patch  ` operation.

A globally unique identifier for the backup which cannot be changed. Values are of the form `  projects/<project>/instances/<instance>/backups/[a-z][a-z0-9_\-]*[a-z0-9]  ` The final segment of the name must be between 2 and 60 characters in length.

The backup is stored in the location(s) specified in the instance configuration of the instance containing the backup, identified by the prefix of the backup name of the form `  projects/<project>/instances/<instance>  ` .

### Query parameters

Parameters

`  updateMask  `

`  string ( FieldMask  ` format)

Required. A mask specifying which fields (for example, `  expireTime  ` ) in the backup resource should be updated. This mask is relative to the backup resource, not to the request message. The field mask must always be specified; this prevents any future fields from being erased accidentally by clients that do not know about them.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .

### Request body

The request body contains an instance of `  Backup  ` .

### Response body

If successful, the response body contains an instance of `  Backup  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

### IAM Permissions

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `  name  ` resource:

  - `  spanner.backups.update  `

For more information, see the [IAM documentation](https://cloud.google.com/iam/docs) .
