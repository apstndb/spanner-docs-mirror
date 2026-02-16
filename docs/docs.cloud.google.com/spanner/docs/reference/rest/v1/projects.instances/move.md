  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [DatabaseMoveConfig](#DatabaseMoveConfig)
      - [JSON representation](#DatabaseMoveConfig.SCHEMA_REPRESENTATION)
  - [EncryptionConfig](#EncryptionConfig)
      - [JSON representation](#EncryptionConfig.SCHEMA_REPRESENTATION)
  - [Try it\!](#try-it)

Moves an instance to the target instance configuration. You can use the returned long-running operation to track the progress of moving the instance.

`  instances.move  ` returns `  FAILED_PRECONDITION  ` if the instance meets any of the following criteria:

  - Is undergoing a move to a different instance configuration
  - Has backups
  - Has an ongoing update
  - Contains any CMEK-enabled databases
  - Is a free trial instance

While the operation is pending:

  - All other attempts to modify the instance, including changes to its compute capacity, are rejected.
  - The following database and backup admin operations are rejected:

<!-- end list -->

``` text
* `DatabaseAdmin.CreateDatabase`
* `DatabaseAdmin.UpdateDatabaseDdl` (disabled if defaultLeader is
   specified in the request.)
* `DatabaseAdmin.RestoreDatabase`
* `DatabaseAdmin.CreateBackup`
* `DatabaseAdmin.CopyBackup`
```

  - Both the source and target instance configurations are subject to hourly compute and storage charges.
  - The instance might experience higher read-write latencies and a higher transaction abort rate. However, moving an instance doesn't cause any downtime.

The returned long-running operation has a name of the format `  <instance_name>/operations/<operationId>  ` and can be used to track the move instance operation. The metadata field type is `  MoveInstanceMetadata  ` . The response field type is `  Instance  ` , if successful. Cancelling the operation sets its metadata's `  cancelTime  ` . Cancellation is not immediate because it involves moving any data previously moved to the target instance configuration back to the original instance configuration. You can use this operation to track the progress of the cancellation. Upon successful completion of the cancellation, the operation terminates with `  CANCELLED  ` status.

If not cancelled, upon completion of the returned operation:

  - The instance successfully moves to the target instance configuration.
  - You are billed for compute and storage in target instance configuration.

Authorization requires the `  spanner.instances.update  ` permission on the resource `  instance  ` .

For more details, see [Move an instance](https://cloud.google.com/spanner/docs/move-instance) .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{name=projects/*/instances/*}:move  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The instance to move. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.instances.update  `

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
  &quot;targetConfig&quot;: string,
  &quot;targetDatabaseMoveConfigs&quot;: [
    {
      object (DatabaseMoveConfig)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  targetConfig  `

`  string  `

Required. The target instance configuration where to move the instance. Values are of the form `  projects/<project>/instanceConfigs/<config>  ` .

`  targetDatabaseMoveConfigs[]  `

`  object ( DatabaseMoveConfig  ` )

Optional. The configuration for each database in the target instance configuration.

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

## DatabaseMoveConfig

The configuration for each database in the target instance configuration.

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
    object (EncryptionConfig)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  databaseId  `

`  string  `

Required. The unique identifier of the database resource in the Instance. For example, if the database uri is `  projects/foo/instances/bar/databases/baz  ` , then the id to supply here is baz.

`  encryptionConfig  `

`  object ( EncryptionConfig  ` )

Optional. Encryption configuration to be used for the database in the target configuration. The encryption configuration must be specified for every database which currently uses CMEK encryption. If a database currently uses Google-managed encryption and a target encryption configuration is not specified, then the database defaults to Google-managed encryption.

If a database currently uses Google-managed encryption and a target CMEK encryption is specified, the request is rejected.

If a database currently uses CMEK encryption, then a target encryption configuration must be specified. You can't move a CMEK database to a Google-managed encryption database using the instances.move API.

## EncryptionConfig

Encryption configuration for a Cloud Spanner database.

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
  &quot;kmsKeyName&quot;: string,
  &quot;kmsKeyNames&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  kmsKeyName  `

`  string  `

Optional. This field is maintained for backwards compatibility. For new callers, we recommend using `  kmsKeyNames  ` to specify the KMS key. Only use `  kmsKeyName  ` if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in `  us-central1  ` or `  nam3  ` , then the database instance must also be in `  us-central1  ` or `  nam3  ` .

The Cloud KMS key that is used to encrypt and decrypt the restored database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

`  kmsKeyNames[]  `

`  string  `

Optional. Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

The keys referenced by `  kmsKeyNames  ` must fully cover all regions of the database's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.
