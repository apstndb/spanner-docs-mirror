  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [RestoreDatabaseEncryptionConfig](#RestoreDatabaseEncryptionConfig)
      - [JSON representation](#RestoreDatabaseEncryptionConfig.SCHEMA_REPRESENTATION)
  - [EncryptionType](#EncryptionType)
  - [Try it\!](#try-it)

Create a new database by restoring from a completed backup. The new database must be in the same project and in an instance with the same instance configuration as the instance containing the backup. The returned database long-running operation has a name of the format `  projects/<project>/instances/<instance>/databases/<database>/operations/<operationId>  ` , and can be used to track the progress of the operation, and to cancel it. The metadata field type is `  RestoreDatabaseMetadata  ` . The response type is `  Database  ` , if successful. Cancelling the returned operation will stop the restore and delete the database. There can be only one database being restored into an instance at a time. Once the restore operation completes, a new restore operation can be initiated, without waiting for the optimize operation associated with the first restore to complete.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databases:restore  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the instance in which to create the restored database. This instance must be in the same project and have the same instance configuration as the instance containing the source backup. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.databases.create  `

### Request body

The request body contains data with the following structure:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;databaseId&quot;: string,
  &quot;encryptionConfig&quot;: {
    object (RestoreDatabaseEncryptionConfig)
  },

  // Union field source can be only one of the following:
  &quot;backup&quot;: string
  // End of list of possible types for union field source.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  databaseId  `

`  string  `

Required. The id of the database to create and restore to. This database must not already exist. The `  databaseId  ` appended to `  parent  ` forms the full database name of the form `  projects/<project>/instances/<instance>/databases/<databaseId>  ` .

`  encryptionConfig  `

`  object ( RestoreDatabaseEncryptionConfig  ` )

Optional. An encryption configuration describing the encryption type and key resources in Cloud KMS used to encrypt/decrypt the database to restore to. If this field is not specified, the restored database will use the same encryption configuration as the backup by default, namely `  encryptionType  ` = `  USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION  ` .

Union field `  source  ` . Required. The source from which to restore. `  source  ` can be only one of the following:

`  backup  `

`  string  `

Name of the backup from which to restore. Values are of the form `  projects/<project>/instances/<instance>/backups/<backup>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  backup  ` :

  - `  spanner.backups.restoreDatabase  `

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

## RestoreDatabaseEncryptionConfig

Encryption configuration for the restored database.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;encryptionType&quot;: enum (EncryptionType),
  &quot;kmsKeyName&quot;: string,
  &quot;kmsKeyNames&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  encryptionType  `

`  enum ( EncryptionType  ` )

Required. The encryption type of the restored database.

`  kmsKeyName  `

`  string  `

Optional. This field is maintained for backwards compatibility. For new callers, we recommend using `  kmsKeyNames  ` to specify the KMS key. Only use `  kmsKeyName  ` if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in `  us-central1  ` or `  nam3  ` , then the database instance must also be in `  us-central1  ` or `  nam3  ` .

The Cloud KMS key that is used to encrypt and decrypt the restored database. Set this field only when `  encryptionType  ` is `  CUSTOMER_MANAGED_ENCRYPTION  ` . Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

`  kmsKeyNames[]  `

`  string  `

Optional. Specifies the KMS configuration for one or more keys used to encrypt the database. Values have the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

The keys referenced by `  kmsKeyNames  ` must fully cover all regions of the database's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

## EncryptionType

Encryption types for the database to be restored.

Enums

`  ENCRYPTION_TYPE_UNSPECIFIED  `

Unspecified. Do not use.

`  USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION  `

This is the default option when `  encryptionConfig  ` is not specified.

`  GOOGLE_DEFAULT_ENCRYPTION  `

Use Google default encryption.

`  CUSTOMER_MANAGED_ENCRYPTION  `

Use customer managed encryption. If specified, `  kmsKeyName  ` must must contain a valid Cloud KMS key.
