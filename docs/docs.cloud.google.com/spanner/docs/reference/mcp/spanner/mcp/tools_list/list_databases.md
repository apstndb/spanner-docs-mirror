---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases
title: 'MCP Tools Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `list_databases`

List Spanner databases in a given spanner instance. \* Response may include next_page_token to fetch additional databases using list_databases tool with page_token set.

The following sample demonstrate how to use `curl` to invoke the `list_databases` MCP tool.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "list_databases",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request for `ListDatabases` .

### ListDatabasesRequest

**JSON representation**

```
{
  "parent": string,
  "pageSize": integer,
  "pageToken": string
}
```

| Fields      |                                                                                                                                                                                                                                                        |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `parent`    | `string` Required. The instance whose databases should be listed. Values are of the form `projects/<project>/instances/<instance>` .                                                                                                                   |
| `pageSize`  | `integer` Number of databases to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.                                                                                                                        |
| `pageToken` | `string` If non-empty, `page_token` should contain a `next_page_token` from a previous [`ListDatabasesResponse`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.ListDatabasesResponse) . |

## Output Schema

The response for `ListDatabases` .

### ListDatabasesResponse

**JSON representation**

```
{
  "databases": [
    {
      object (Database)
    }
  ],
  "nextPageToken": string
}
```

| Fields          |                                                                                                                                                                                        |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `databases[]`   | `object ( `[`Database`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.Database)` )` Databases that matched the request. |
| `nextPageToken` | `string` `next_page_token` can be sent in a subsequent `ListDatabases` call to fetch more of the matching databases.                                                                   |

### Database

**JSON representation**

```
{
  "name": string,
  "state": enum (State),
  "createTime": string,
  "restoreInfo": {
    object (RestoreInfo)
  },
  "encryptionConfig": {
    object (EncryptionConfig)
  },
  "encryptionInfo": [
    {
      object (EncryptionInfo)
    }
  ],
  "versionRetentionPeriod": string,
  "earliestVersionTime": string,
  "defaultLeader": string,
  "databaseDialect": enum (DatabaseDialect),
  "enableDropProtection": boolean,
  "reconciling": boolean,
  "quorumInfo": {
    object (QuorumInfo)
  }
}
```

| Fields                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                   | `string` Required. The name of the database. Values are of the form `projects/<project>/instances/<instance>/databases/<database>` , where `<database>` is as specified in the `CREATE DATABASE` statement. This name can be passed to other API methods to identify the database.                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `state`                  | `enum ( ``State`` )` Output only. The current database state.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `createTime`             | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Output only. If exists, the time at which the database creation started. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                                                                                                                                                                |
| `restoreInfo`            | `object ( `[`RestoreInfo`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.RestoreInfo)` )` Output only. Applicable only for restored databases. Contains information about the restore source.                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `encryptionConfig`       | `object ( `[`EncryptionConfig`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_database#Input.Schema.EncryptionConfig)` )` Output only. For databases that are using customer managed encryption, this field contains the encryption configuration for the database. For databases that are using Google default or other types of encryption, this field is empty.                                                                                                                                                                                                                                                                                                                   |
| `encryptionInfo[]`       | `object ( `[`EncryptionInfo`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.EncryptionInfo)` )` Output only. For databases that are using customer managed encryption, this field contains the encryption information for the database, such as all Cloud KMS key versions that are in use. The `encryption_status` field inside of each `EncryptionInfo` is not populated. For databases that are using Google default or other types of encryption, this field is empty. This field is propagated lazily from the backend. There might be a delay from when a key version is being used and when it appears in this field.                                   |
| `versionRetentionPeriod` | `string` Output only. The period in which Cloud Spanner retains all versions of data for the database. This is the same as the value of version_retention_period database option set using `UpdateDatabaseDdl` . Defaults to 1 hour, if not set.                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `earliestVersionTime`    | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Output only. Earliest timestamp at which older versions of the data can be read. This value is continuously updated by Cloud Spanner and becomes stale the moment it is queried. If you are using this value to recover data, make sure to account for the time from the moment when the value is queried to the moment when you initiate the recovery. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `defaultLeader`          | `string` Output only. The read-write region which contains the database's leader replicas. This is the same as the value of default_leader database option set using DatabaseAdmin.CreateDatabase or DatabaseAdmin.UpdateDatabaseDdl. If not explicitly set, this is empty.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `databaseDialect`        | `enum ( ``DatabaseDialect`` )` Output only. The dialect of the Cloud Spanner Database.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `enableDropProtection`   | `boolean` Optional. Whether drop protection is enabled for this database. Defaults to false, if not set. For more details, please see how to [prevent accidental database deletion](https://cloud.google.com/spanner/docs/prevent-database-deletion) .                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `reconciling`            | `boolean` Output only. If true, the database is being updated. If false, there are no ongoing update operations for the database.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `quorumInfo`             | `object ( `[`QuorumInfo`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.QuorumInfo)` )` Output only. Applicable only for databases that use dual-region instance configurations. Contains information about the quorum.                                                                                                                                                                                                                                                                                                                                                                                                                                        |

### Timestamp

**JSON representation**

```
{
  "seconds": string,
  "nanos": integer
}
```

| Fields    |                                                                                                                                                                                                                                                                                                                      |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `seconds` | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).                            |
| `nanos`   | `integer` Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive. |

### RestoreInfo

**JSON representation**

```
{
  "sourceType": enum (RestoreSourceType),

  // Union field source_info can be only one of the following:
  "backupInfo": {
    object (BackupInfo)
  }
  // End of list of possible types for union field source_info.
}
```

| Fields                                                                                                                                 |                                                                                                                                                                                                                                                   |
|----------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sourceType`                                                                                                                           | `enum ( ``RestoreSourceType`` )` The type of the restore source.                                                                                                                                                                                  |
| Union field `source_info` . Information about the source used to restore the database. `source_info` can be only one of the following: |                                                                                                                                                                                                                                                   |
| `backupInfo`                                                                                                                           | `object ( `[`BackupInfo`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.BackupInfo)` )` Information about the backup used to restore the database. The backup may no longer exist. |

### BackupInfo

**JSON representation**

```
{
  "backup": string,
  "versionTime": string,
  "createTime": string,
  "sourceDatabase": string
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `backup`         | `string` Name of the backup.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `versionTime`    | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The backup contains an externally consistent copy of `source_database` at the timestamp specified by `version_time` . If the `CreateBackup` request did not specify `version_time` , the `version_time` of the backup is equivalent to the `create_time` . Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `createTime`     | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time the `CreateBackup` request was received. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                                                                                          |
| `sourceDatabase` | `string` Name of the database the backup was created from.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

### EncryptionConfig

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
<p>The Cloud KMS key to be used for encrypting and decrypting the database. Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kms_key_name&gt;</code> .</p></td>
</tr>
<tr class="even">
<td><code>kmsKeyNames[]</code></td>
<td><p><code>string</code></p>
<p>Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kms_key_name&gt;</code> .</p>
<p>The keys referenced by <code>kms_key_names</code> must fully cover all regions of the database's instance configuration. Some examples:</p>
<ul>
<li>For regional (single-region) instance configurations, specify a regional location KMS key.</li>
<li>For multi-region instance configurations of type <code>GOOGLE_MANAGED</code> , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.</li>
<li>For an instance configuration of type <code>USER_MANAGED</code> , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for <code>USER_MANAGED</code> type instance configurations.</li>
</ul></td>
</tr>
</tbody>
</table>

### EncryptionInfo

**JSON representation**

```
{
  "encryptionType": enum (Type),
  "encryptionStatus": {
    object (Status)
  },
  "kmsKeyVersion": string
}
```

| Fields             |                                                                                                                                                                                                                                                                                                                              |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `encryptionType`   | `enum ( ``Type`` )` Output only. The type of encryption.                                                                                                                                                                                                                                                                     |
| `encryptionStatus` | `object ( `[`Status`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_instance#Output.Schema.Status)` )` Output only. If present, the status of a recent encrypt/decrypt call on underlying data for this database or backup. Regardless of status, data is always encrypted at rest. |
| `kmsKeyVersion`    | `string` Output only. A Cloud KMS key version that is being used to protect the database or backup.                                                                                                                                                                                                                          |

### Status

**JSON representation**

```
{
  "code": integer,
  "message": string,
  "details": [
    {
      "@type": string,
      field1: ...,
      ...
    }
  ]
}
```

| Fields      |                                                                                                                                                                                                                                                                                                              |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `code`      | `integer` The status code, which should be an enum value of `google.rpc.Code` .                                                                                                                                                                                                                              |
| `message`   | `string` A developer-facing error message, which should be in English. Any user-facing error message should be localized and sent in the `google.rpc.Status.details` field, or localized by the client.                                                                                                      |
| `details[]` | `object` A list of messages that carry the error details. There is a common set of message types for APIs to use. An object containing fields of an arbitrary type. An additional field `"@type"` contains a URI identifying the type. Example: `{ "id": 1234, "@type": "types.example.com/standard/id" }` . |

### Any

**JSON representation**

```
{
  "typeUrl": string,
  "value": string
}
```

| Fields    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `typeUrl` | `string` Identifies the type of the serialized Protobuf message with a URI reference consisting of a prefix ending in a slash and the fully-qualified type name. Example: type.googleapis.com/google.protobuf.StringValue This string must contain at least one `/` character, and the content after the last `/` must be the fully-qualified name of the type in canonical form, without a leading dot. Do not write a scheme on these URI references so that clients do not attempt to contact them. The prefix is arbitrary and Protobuf implementations are expected to simply strip off everything up to and including the last `/` to identify the type. `type.googleapis.com/` is a common default prefix that some legacy implementations require. This prefix does not indicate the origin of the type, and URIs containing it are not expected to respond to any requests. All type URL strings must be legal URI references with the additional restriction (for the text format) that the content of the reference must consist only of alphanumeric characters, percent-encoded escapes, and characters in the following set (not including the outer backticks): `/-.~_!$&()*+,;=` . Despite our allowing percent encodings, implementations should not unescape them to prevent confusion with existing parsers. For example, `type.googleapis.com%2FFoo` should be rejected. In the original design of `Any` , the possibility of launching a type resolution service at these type URLs was considered but Protobuf never implemented one and considers contacting these URLs to be problematic and a potential security issue. Do not attempt to contact type URLs. |
| `value`   | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Holds a Protobuf serialization of the type described by type_url. A base64-encoded string.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |

### QuorumInfo

**JSON representation**

```
{
  "quorumType": {
    object (QuorumType)
  },
  "initiator": enum (Initiator),
  "startTime": string,
  "etag": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `quorumType` | `object ( `[`QuorumType`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.QuorumType)` )` Output only. The type of this quorum. See [`QuorumType`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.QuorumType) for more information about quorum type specifications.                                  |
| `initiator`  | `enum ( ``Initiator`` )` Output only. Whether this `ChangeQuorum` is Google or User initiated.                                                                                                                                                                                                                                                                                                                                   |
| `startTime`  | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Output only. The timestamp when the request was triggered. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `etag`       | `string` Output only. The etag is used for optimistic concurrency control as a way to help prevent simultaneous `ChangeQuorum` requests that might create a race condition.                                                                                                                                                                                                                                                      |

### QuorumType

**JSON representation**

```
{

  // Union field type can be only one of the following:
  "singleRegion": {
    object (SingleRegionQuorum)
  },
  "dualRegion": {
    object (DualRegionQuorum)
  }
  // End of list of possible types for union field type.
}
```

| Fields                                                                            |                                                                                                                                                                                                   |
|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `type` . The type of quorum. `type` can be only one of the following: |                                                                                                                                                                                                   |
| `singleRegion`                                                                    | `object ( `[`SingleRegionQuorum`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_databases#Output.Schema.SingleRegionQuorum)` )` Single-region quorum type. |
| `dualRegion`                                                                      | `object ( ``DualRegionQuorum`` )` Dual-region quorum type.                                                                                                                                        |

### SingleRegionQuorum

**JSON representation**

```
{
  "servingLocation": string
}
```

| Fields            |                                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `servingLocation` | `string` Required. The location of the serving region, for example, "us-central1". The location must be one of the regions within the dual-region instance configuration of your database. The list of valid locations is available using the \[GetInstanceConfig\]\[InstanceAdmin.GetInstanceConfig\] API. This should only be used if you plan to change quorum to the single-region quorum type. |

### Tool Annotations

Destructive Hint: ❌ \| Idempotent Hint: ✅ \| Read Only Hint: ✅ \| Open World Hint: ❌
