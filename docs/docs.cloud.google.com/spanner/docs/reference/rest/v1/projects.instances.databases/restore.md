---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore
title: 'Method: projects.instances.databases.restore'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#body.aspect)
- [RestoreDatabaseEncryptionConfig](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig.SCHEMA_REPRESENTATION)
- [EncryptionType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#EncryptionType)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#try-it)

Create a new database by restoring from a completed backup. The new database must be in the same project and in an instance with the same instance configuration as the instance containing the backup. The returned database long-running operation has a name of the format `projects/<project>/instances/<instance>/databases/<database>/operations/<operationId>` , and can be used to track the progress of the operation, and to cancel it. The metadata field type is [`RestoreDatabaseMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/RestoreDatabaseMetadata) . The response type is [`Database`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#Database) , if successful. Cancelling the returned operation will stop the restore and delete the database. There can be only one database being restored into an instance at a time. Once the restore operation completes, a new restore operation can be initiated, without waiting for the optimize operation associated with the first restore to complete.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databases:restore`

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
<p>Required. The name of the instance in which to create the restored database. This instance must be in the same project and have the same instance configuration as the instance containing the source backup. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.databases.create</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "databaseId": string,
  "encryptionConfig": {
    object (RestoreDatabaseEncryptionConfig)
  },

  // Union field source can be only one of the following:
  "backup": string
  // End of list of possible types for union field source.
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
<td><code>databaseId</code></td>
<td><p><code>string</code></p>
<p>Required. The id of the database to create and restore to. This database must not already exist. The <code>databaseId</code> appended to <code>parent</code> forms the full database name of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/databases/&lt;databaseId&gt;</code> .</p></td>
</tr>
<tr class="even">
<td><code>encryptionConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig"><code>RestoreDatabaseEncryptionConfig</code></a><code> )</code></p>
<p>Optional. An encryption configuration describing the encryption type and key resources in Cloud KMS used to encrypt/decrypt the database to restore to. If this field is not specified, the restored database will use the same encryption configuration as the backup by default, namely <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig.FIELDS.encryption_type"><code>encryptionType</code></a> = <code>USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION</code> .</p></td>
</tr>
<tr class="odd">
<td>Union field <code>source</code> . Required. The source from which to restore. <code>source</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>backup</code></td>
<td><p><code>string</code></p>
<p>Name of the backup from which to restore. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/backups/&lt;backup&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>backup</code> :</p>
<ul>
<li><code>spanner.backups.restoreDatabase</code></li>
</ul></td>
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

## RestoreDatabaseEncryptionConfig

Encryption configuration for the restored database.

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
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#EncryptionType"><code>EncryptionType</code></a><code> )</code></p>
<p>Required. The encryption type of the restored database.</p></td>
</tr>
<tr class="even">
<td><code>kmsKeyName</code></td>
<td><p><code>string</code></p>
<p>Optional. This field is maintained for backwards compatibility. For new callers, we recommend using <code>kmsKeyNames</code> to specify the KMS key. Only use <code>kmsKeyName</code> if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in <code>us-central1</code> or <code>nam3</code> , then the database instance must also be in <code>us-central1</code> or <code>nam3</code> .</p>
<p>The Cloud KMS key that is used to encrypt and decrypt the restored database. Set this field only when <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig.FIELDS.encryption_type"><code>encryptionType</code></a> is <code>CUSTOMER_MANAGED_ENCRYPTION</code> . Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p></td>
</tr>
<tr class="odd">
<td><code>kmsKeyNames[]</code></td>
<td><p><code>string</code></p>
<p>Optional. Specifies the KMS configuration for one or more keys used to encrypt the database. Values have the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p>
<p>The keys referenced by <code>kmsKeyNames</code> must fully cover all regions of the database's instance configuration. Some examples:</p>
<ul>
<li>For regional (single-region) instance configurations, specify a regional location KMS key.</li>
<li>For multi-region instance configurations of type <code>GOOGLE_MANAGED</code> , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.</li>
<li>For an instance configuration of type <code>USER_MANAGED</code> , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for <code>USER_MANAGED</code> type instance configurations.</li>
</ul></td>
</tr>
</tbody>
</table>

## EncryptionType

Encryption types for the database to be restored.

| Enums                                     |                                                                                                                                                                                                           |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ENCRYPTION_TYPE_UNSPECIFIED`             | Unspecified. Do not use.                                                                                                                                                                                  |
| `USE_CONFIG_DEFAULT_OR_BACKUP_ENCRYPTION` | This is the default option when [`encryptionConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/restore#RestoreDatabaseEncryptionConfig) is not specified. |
| `GOOGLE_DEFAULT_ENCRYPTION`               | Use Google default encryption.                                                                                                                                                                            |
| `CUSTOMER_MANAGED_ENCRYPTION`             | Use customer managed encryption. If specified, `kmsKeyName` must must contain a valid Cloud KMS key.                                                                                                      |
