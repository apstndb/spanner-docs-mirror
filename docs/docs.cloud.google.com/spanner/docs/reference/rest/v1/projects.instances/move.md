---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move
title: 'Method: projects.instances.move'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#body.aspect)
- [DatabaseMoveConfig](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#DatabaseMoveConfig)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#DatabaseMoveConfig.SCHEMA_REPRESENTATION)
- [EncryptionConfig](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#EncryptionConfig)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#EncryptionConfig.SCHEMA_REPRESENTATION)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#try-it)

Moves an instance to the target instance configuration. You can use the returned long-running operation to track the progress of moving the instance.

`instances.move` returns `FAILED_PRECONDITION` if the instance meets any of the following criteria:

- Is undergoing a move to a different instance configuration
- Has backups
- Has an ongoing update
- Contains any CMEK-enabled databases
- Is a free trial instance

While the operation is pending:

- All other attempts to modify the instance, including changes to its compute capacity, are rejected.
- The following database and backup admin operations are rejected:

```
* `DatabaseAdmin.CreateDatabase`
* `DatabaseAdmin.UpdateDatabaseDdl` (disabled if defaultLeader is
   specified in the request.)
* `DatabaseAdmin.RestoreDatabase`
* `DatabaseAdmin.CreateBackup`
* `DatabaseAdmin.CopyBackup`
```

- Both the source and target instance configurations are subject to hourly compute and storage charges.
- The instance might experience higher read-write latencies and a higher transaction abort rate. However, moving an instance doesn't cause any downtime.

The returned long-running operation has a name of the format `<instance_name>/operations/<operationId>` and can be used to track the move instance operation. The metadata field type is [`MoveInstanceMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MoveInstanceMetadata) . The response field type is [`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance) , if successful. Cancelling the operation sets its metadata's [`cancelTime`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MoveInstanceMetadata#FIELDS.cancel_time) . Cancellation is not immediate because it involves moving any data previously moved to the target instance configuration back to the original instance configuration. You can use this operation to track the progress of the cancellation. Upon successful completion of the cancellation, the operation terminates with `CANCELLED` status.

If not cancelled, upon completion of the returned operation:

- The instance successfully moves to the target instance configuration.
- You are billed for compute and storage in target instance configuration.

Authorization requires the `spanner.instances.update` permission on the resource [`instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance) .

For more details, see [Move an instance](https://cloud.google.com/spanner/docs/move-instance) .

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{name=projects/*/instances/*}:move`

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
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>Required. The instance to move. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.instances.update</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "targetConfig": string,
  "targetDatabaseMoveConfigs": [
    {
      object (DatabaseMoveConfig)
    }
  ]
}
```

| Fields                        |                                                                                                                                                                                                                                    |
|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `targetConfig`                | `string` Required. The target instance configuration where to move the instance. Values are of the form `projects/<project>/instanceConfigs/<config>` .                                                                            |
| `targetDatabaseMoveConfigs[]` | `object ( `[`DatabaseMoveConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#DatabaseMoveConfig)` )` Optional. The configuration for each database in the target instance configuration. |

### Response body

If successful, the response body contains an instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## DatabaseMoveConfig

The configuration for each database in the target instance configuration.

**JSON representation**

```
{
  "databaseId": string,
  "encryptionConfig": {
    object (EncryptionConfig)
  }
}
```

| Fields             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `databaseId`       | `string` Required. The unique identifier of the database resource in the Instance. For example, if the database uri is `projects/foo/instances/bar/databases/baz` , then the id to supply here is baz.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `encryptionConfig` | `object ( `[`EncryptionConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#EncryptionConfig)` )` Optional. Encryption configuration to be used for the database in the target configuration. The encryption configuration must be specified for every database which currently uses CMEK encryption. If a database currently uses Google-managed encryption and a target encryption configuration is not specified, then the database defaults to Google-managed encryption. If a database currently uses Google-managed encryption and a target CMEK encryption is specified, the request is rejected. If a database currently uses CMEK encryption, then a target encryption configuration must be specified. You can't move a CMEK database to a Google-managed encryption database using the instances.move API. |

## EncryptionConfig

Encryption configuration for a Cloud Spanner database.

**JSON representation**

```
{
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
<td><code>kmsKeyName</code></td>
<td><p><code>string</code></p>
<p>Optional. This field is maintained for backwards compatibility. For new callers, we recommend using <code>kmsKeyNames</code> to specify the KMS key. Only use <code>kmsKeyName</code> if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in <code>us-central1</code> or <code>nam3</code> , then the database instance must also be in <code>us-central1</code> or <code>nam3</code> .</p>
<p>The Cloud KMS key that is used to encrypt and decrypt the restored database. Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p></td>
</tr>
<tr class="even">
<td><code>kmsKeyNames[]</code></td>
<td><p><code>string</code></p>
<p>Optional. Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p>
<p>The keys referenced by <code>kmsKeyNames</code> must fully cover all regions of the database's instance configuration. Some examples:</p>
<ul>
<li>For regional (single-region) instance configurations, specify a regional location KMS key.</li>
<li>For multi-region instance configurations of type <code>GOOGLE_MANAGED</code> , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.</li>
<li>For an instance configuration of type <code>USER_MANAGED</code> , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for <code>USER_MANAGED</code> type instance configurations.</li>
</ul></td>
</tr>
</tbody>
</table>
