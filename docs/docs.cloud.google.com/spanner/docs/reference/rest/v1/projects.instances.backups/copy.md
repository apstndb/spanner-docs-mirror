  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [IAM Permissions](#body.aspect_1)
  - [CopyBackupEncryptionConfig](#CopyBackupEncryptionConfig)
      - [JSON representation](#CopyBackupEncryptionConfig.SCHEMA_REPRESENTATION)
  - [EncryptionType](#EncryptionType)
  - [Try it\!](#try-it)

Starts copying a Cloud Spanner Backup. The returned backup long-running operation will have a name of the format `  projects/<project>/instances/<instance>/backups/<backup>/operations/<operationId>  ` and can be used to track copying of the backup. The operation is associated with the destination backup. The metadata field type is `  CopyBackupMetadata  ` . The response field type is `  Backup  ` , if successful. Cancelling the returned operation will stop the copying and delete the destination backup. Concurrent backups.copy requests can run on the same source backup.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/backups:copy  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the destination instance that will contain the backup copy. Values are of the form: `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.backups.create  `

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
  &quot;backupId&quot;: string,
  &quot;sourceBackup&quot;: string,
  &quot;expireTime&quot;: string,
  &quot;encryptionConfig&quot;: {
    object (CopyBackupEncryptionConfig)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  backupId  `

`  string  `

Required. The id of the backup copy. The `  backupId  ` appended to `  parent  ` forms the full backup\_uri of the form `  projects/<project>/instances/<instance>/backups/<backup>  ` .

`  sourceBackup  `

`  string  `

Required. The source backup to be copied. The source backup needs to be in READY state for it to be copied. Once backups.copy is in progress, the source backup cannot be deleted or cleaned up on expiration until backups.copy is finished. Values are of the form: `  projects/<project>/instances/<instance>/backups/<backup>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  sourceBackup  ` :

  - `  spanner.backups.copy  `

`  expireTime  `

`  string ( Timestamp  ` format)

Required. The expiration time of the backup in microsecond granularity. The expiration time must be at least 6 hours and at most 366 days from the `  createTime  ` of the source backup. Once the `  expireTime  ` has passed, the backup is eligible to be automatically deleted by Cloud Spanner to free the resources used by the backup.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  encryptionConfig  `

`  object ( CopyBackupEncryptionConfig  ` )

Optional. The encryption configuration used to encrypt the backup. If this field is not specified, the backup will use the same encryption configuration as the source backup by default, namely `  encryptionType  ` = `  USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION  ` .

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

### IAM Permissions

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `  parent  ` resource:

  - `  spanner.backups.create  `

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `  sourceBackup  ` resource:

  - `  spanner.backups.copy  `

For more information, see the [IAM documentation](https://cloud.google.com/iam/docs) .

## CopyBackupEncryptionConfig

Encryption configuration for the copied backup.

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

Required. The encryption type of the backup.

`  kmsKeyName  `

`  string  `

Optional. This field is maintained for backwards compatibility. For new callers, we recommend using `  kmsKeyNames  ` to specify the KMS key. Only use `  kmsKeyName  ` if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in `  us-central1  ` or `  nam3  ` , then the database instance must also be in `  us-central1  ` or `  nam3  ` .

The Cloud KMS key that is used to encrypt and decrypt the restored database. Set this field only when `  encryptionType  ` is `  CUSTOMER_MANAGED_ENCRYPTION  ` . Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

`  kmsKeyNames[]  `

`  string  `

Optional. Specifies the KMS configuration for the one or more keys used to protect the backup. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` . KMS keys specified can be in any order.

The keys referenced by `  kmsKeyNames  ` must fully cover all regions of the backup's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

## EncryptionType

Encryption types for the backup.

Enums

`  ENCRYPTION_TYPE_UNSPECIFIED  `

Unspecified. Do not use.

`  USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION  `

This is the default option for `  backups.copy  ` when `  encryptionConfig  ` is not specified. For example, if the source backup is using `  Customer_Managed_Encryption  ` , the backup will be using the same Cloud KMS key as the source backup.

`  GOOGLE_DEFAULT_ENCRYPTION  `

Use Google default encryption.

`  CUSTOMER_MANAGED_ENCRYPTION  `

Use customer managed encryption. If specified, either `  kmsKeyName  ` or `  kmsKeyNames  ` must contain valid Cloud KMS keys.
