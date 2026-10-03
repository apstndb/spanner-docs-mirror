---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy
title: 'Method: projects.instances.backups.copy'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.aspect)
- [IAM Permissions](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#body.aspect_1)
- [CopyBackupEncryptionConfig](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#CopyBackupEncryptionConfig)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#CopyBackupEncryptionConfig.SCHEMA_REPRESENTATION)
- [EncryptionType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#EncryptionType)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#try-it)

Starts copying a Cloud Spanner Backup. The returned backup long-running operation will have a name of the format `projects/<project>/instances/<instance>/backups/<backup>/operations/<operationId>` and can be used to track copying of the backup. The operation is associated with the destination backup. The metadata field type is [`CopyBackupMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CopyBackupMetadata) . The response field type is [`Backup`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups#Backup) , if successful. Cancelling the returned operation will stop the copying and delete the destination backup. Concurrent backups.copy requests can run on the same source backup.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/backups:copy`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>parent</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the destination instance that will contain the backup copy. Values are of the form: <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.backups.create</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "backupId": string,
  "sourceBackup": string,
  "expireTime": string,
  "encryptionConfig": {
    object (CopyBackupEncryptionConfig)
  }
}
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>backupId</code></td>
<td><p><code>string</code></p>
<p>Required. The id of the backup copy. The <code>backupId</code> appended to <code>parent</code> forms the full backup_uri of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/backups/&lt;backup&gt;</code> .</p></td>
</tr>
<tr class="even">
<td><code>sourceBackup</code></td>
<td><p><code>string</code></p>
<p>Required. The source backup to be copied. The source backup needs to be in READY state for it to be copied. Once backups.copy is in progress, the source backup cannot be deleted or cleaned up on expiration until backups.copy is finished. Values are of the form: <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/backups/&lt;backup&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>sourceBackup</code> :</p>
<ul>
<li><code>spanner.backups.copy</code></li>
</ul></td>
</tr>
<tr class="odd">
<td><code>expireTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Required. The expiration time of the backup in microsecond granularity. The expiration time must be at least 6 hours and at most 366 days from the <code>createTime</code> of the source backup. Once the <code>expireTime</code> has passed, the backup is eligible to be automatically deleted by Cloud Spanner to free the resources used by the backup.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="even">
<td><code>encryptionConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#CopyBackupEncryptionConfig"><code>CopyBackupEncryptionConfig</code></a><code> )</code></p>
<p>Optional. The encryption configuration used to encrypt the backup. If this field is not specified, the backup will use the same encryption configuration as the source backup by default, namely <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#CopyBackupEncryptionConfig.FIELDS.encryption_type"><code>encryptionType</code></a> = <code>USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION</code> .</p></td>
</tr>
</tbody>
</table>

### Response body

If successful, the response body contains an instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

### IAM Permissions

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `parent` resource:

- `spanner.backups.create`

Requires the following [IAM](https://cloud.google.com/iam/docs) permission on the `sourceBackup` resource:

- `spanner.backups.copy`

For more information, see the [IAM documentation](https://cloud.google.com/iam/docs) .

## CopyBackupEncryptionConfig

Encryption configuration for the copied backup.

**JSON representation**

```
{
  "encryptionType": enum (EncryptionType),
  "kmsKeyName": string,
  "kmsKeyNames": [
    string
  ]
}
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>encryptionType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#EncryptionType"><code>EncryptionType</code></a><code> )</code></p>
<p>Required. The encryption type of the backup.</p></td>
</tr>
<tr class="even">
<td><code>kmsKeyName</code></td>
<td><p><code>string</code></p>
<p>Optional. This field is maintained for backwards compatibility. For new callers, we recommend using <code>kmsKeyNames</code> to specify the KMS key. Only use <code>kmsKeyName</code> if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in <code>us-central1</code> or <code>nam3</code> , then the database instance must also be in <code>us-central1</code> or <code>nam3</code> .</p>
<p>The Cloud KMS key that is used to encrypt and decrypt the restored database. Set this field only when <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#CopyBackupEncryptionConfig.FIELDS.encryption_type"><code>encryptionType</code></a> is <code>CUSTOMER_MANAGED_ENCRYPTION</code> . Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p></td>
</tr>
<tr class="odd">
<td><code>kmsKeyNames[]</code></td>
<td><p><code>string</code></p>
<p>Optional. Specifies the KMS configuration for the one or more keys used to protect the backup. Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> . KMS keys specified can be in any order.</p>
<p>The keys referenced by <code>kmsKeyNames</code> must fully cover all regions of the backup's instance configuration. Some examples:</p>
<ul>
<li>For regional (single-region) instance configurations, specify a regional location KMS key.</li>
<li>For multi-region instance configurations of type <code>GOOGLE_MANAGED</code> , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.</li>
<li>For an instance configuration of type <code>USER_MANAGED</code> , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for <code>USER_MANAGED</code> type instance configurations.</li>
</ul></td>
</tr>
</tbody>
</table>

## EncryptionType

Encryption types for the backup.

| Enums                                     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|-------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ENCRYPTION_TYPE_UNSPECIFIED`             | Unspecified. Do not use.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION` | This is the default option for [`backups.copy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#google.spanner.admin.database.v1.DatabaseAdmin.CopyBackup) when [`encryptionConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.backups/copy#CopyBackupEncryptionConfig) is not specified. For example, if the source backup is using `Customer_Managed_Encryption` , the backup will be using the same Cloud KMS key as the source backup. |
| `GOOGLE_DEFAULT_ENCRYPTION`               | Use Google default encryption.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `CUSTOMER_MANAGED_ENCRYPTION`             | Use customer managed encryption. If specified, either `kmsKeyName` or `kmsKeyNames` must contain valid Cloud KMS keys.                                                                                                                                                                                                                                                                                                                                                                                                        |
