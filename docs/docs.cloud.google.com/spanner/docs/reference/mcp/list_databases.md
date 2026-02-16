## Tool: `       list_databases      `

List Spanner databases in a given spanner instance. \* Response may include next\_page\_token to fetch additional databases using list\_databases tool with page\_token set.

The following sample demonstrate how to use `  curl  ` to invoke the `  list_databases  ` MCP tool.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Curl Request</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                  
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;list_databases&quot;,
    &quot;arguments&quot;: {
      // provide these details according to the tool&#39;s MCP specification
    }
  },
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;id&quot;: 1
}&#39;
                </code></pre></td>
</tr>
</tbody>
</table>

## Input Schema

The request for `  ListDatabases  ` .

### ListDatabasesRequest

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
  &quot;parent&quot;: string,
  &quot;pageSize&quot;: integer,
  &quot;pageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  parent  `

`  string  `

Required. The instance whose databases should be listed. Values are of the form `  projects/<project>/instances/<instance>  ` .

`  pageSize  `

`  integer  `

Number of databases to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListDatabasesResponse  ` .

## Output Schema

The response for `  ListDatabases  ` .

### ListDatabasesResponse

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
  &quot;databases&quot;: [
    {
      object (Database)
    }
  ],
  &quot;nextPageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  databases[]  `

`  object ( Database  ` )

Databases that matched the request.

`  nextPageToken  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListDatabases  ` call to fetch more of the matching databases.

### Database

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
  &quot;name&quot;: string,
  &quot;state&quot;: enum (State),
  &quot;createTime&quot;: string,
  &quot;restoreInfo&quot;: {
    object (RestoreInfo)
  },
  &quot;encryptionConfig&quot;: {
    object (EncryptionConfig)
  },
  &quot;encryptionInfo&quot;: [
    {
      object (EncryptionInfo)
    }
  ],
  &quot;versionRetentionPeriod&quot;: string,
  &quot;earliestVersionTime&quot;: string,
  &quot;defaultLeader&quot;: string,
  &quot;databaseDialect&quot;: enum (DatabaseDialect),
  &quot;enableDropProtection&quot;: boolean,
  &quot;reconciling&quot;: boolean,
  &quot;quorumInfo&quot;: {
    object (QuorumInfo)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Required. The name of the database. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` , where `  <database>  ` is as specified in the `  CREATE DATABASE  ` statement. This name can be passed to other API methods to identify the database.

`  state  `

`  enum ( State  ` )

Output only. The current database state.

`  createTime  `

`  string ( Timestamp  ` format)

Output only. If exists, the time at which the database creation started.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  restoreInfo  `

`  object ( RestoreInfo  ` )

Output only. Applicable only for restored databases. Contains information about the restore source.

`  encryptionConfig  `

`  object ( EncryptionConfig  ` )

Output only. For databases that are using customer managed encryption, this field contains the encryption configuration for the database. For databases that are using Google default or other types of encryption, this field is empty.

`  encryptionInfo[]  `

`  object ( EncryptionInfo  ` )

Output only. For databases that are using customer managed encryption, this field contains the encryption information for the database, such as all Cloud KMS key versions that are in use. The `  encryption_status  ` field inside of each `  EncryptionInfo  ` is not populated.

For databases that are using Google default or other types of encryption, this field is empty.

This field is propagated lazily from the backend. There might be a delay from when a key version is being used and when it appears in this field.

`  versionRetentionPeriod  `

`  string  `

Output only. The period in which Cloud Spanner retains all versions of data for the database. This is the same as the value of version\_retention\_period database option set using `  UpdateDatabaseDdl  ` . Defaults to 1 hour, if not set.

`  earliestVersionTime  `

`  string ( Timestamp  ` format)

Output only. Earliest timestamp at which older versions of the data can be read. This value is continuously updated by Cloud Spanner and becomes stale the moment it is queried. If you are using this value to recover data, make sure to account for the time from the moment when the value is queried to the moment when you initiate the recovery.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  defaultLeader  `

`  string  `

Output only. The read-write region which contains the database's leader replicas.

This is the same as the value of default\_leader database option set using DatabaseAdmin.CreateDatabase or DatabaseAdmin.UpdateDatabaseDdl. If not explicitly set, this is empty.

`  databaseDialect  `

`  enum ( DatabaseDialect  ` )

Output only. The dialect of the Cloud Spanner Database.

`  enableDropProtection  `

`  boolean  `

Optional. Whether drop protection is enabled for this database. Defaults to false, if not set. For more details, please see how to [prevent accidental database deletion](https://cloud.google.com/spanner/docs/prevent-database-deletion) .

`  reconciling  `

`  boolean  `

Output only. If true, the database is being updated. If false, there are no ongoing update operations for the database.

`  quorumInfo  `

`  object ( QuorumInfo  ` )

Output only. Applicable only for databases that use dual-region instance configurations. Contains information about the quorum.

### Timestamp

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
  &quot;seconds&quot;: string,
  &quot;nanos&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  seconds  `

`  string ( int64 format)  `

Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).

`  nanos  `

`  integer  `

Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive.

### RestoreInfo

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
  &quot;sourceType&quot;: enum (RestoreSourceType),

  // Union field source_info can be only one of the following:
  &quot;backupInfo&quot;: {
    object (BackupInfo)
  }
  // End of list of possible types for union field source_info.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  sourceType  `

`  enum ( RestoreSourceType  ` )

The type of the restore source.

Union field `  source_info  ` . Information about the source used to restore the database. `  source_info  ` can be only one of the following:

`  backupInfo  `

`  object ( BackupInfo  ` )

Information about the backup used to restore the database. The backup may no longer exist.

### BackupInfo

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
  &quot;backup&quot;: string,
  &quot;versionTime&quot;: string,
  &quot;createTime&quot;: string,
  &quot;sourceDatabase&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  backup  `

`  string  `

Name of the backup.

`  versionTime  `

`  string ( Timestamp  ` format)

The backup contains an externally consistent copy of `  source_database  ` at the timestamp specified by `  version_time  ` . If the `  CreateBackup  ` request did not specify `  version_time  ` , the `  version_time  ` of the backup is equivalent to the `  create_time  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  createTime  `

`  string ( Timestamp  ` format)

The time the `  CreateBackup  ` request was received.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  sourceDatabase  `

`  string  `

Name of the database the backup was created from.

### EncryptionConfig

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

The Cloud KMS key to be used for encrypting and decrypting the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kms_key_name>  ` .

`  kmsKeyNames[]  `

`  string  `

Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kms_key_name>  ` .

The keys referenced by `  kms_key_names  ` must fully cover all regions of the database's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

### EncryptionInfo

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
  &quot;encryptionType&quot;: enum (Type),
  &quot;encryptionStatus&quot;: {
    object (Status)
  },
  &quot;kmsKeyVersion&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  encryptionType  `

`  enum ( Type  ` )

Output only. The type of encryption.

`  encryptionStatus  `

`  object ( Status  ` )

Output only. If present, the status of a recent encrypt/decrypt call on underlying data for this database or backup. Regardless of status, data is always encrypted at rest.

`  kmsKeyVersion  `

`  string  `

Output only. A Cloud KMS key version that is being used to protect the database or backup.

### Status

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
  &quot;code&quot;: integer,
  &quot;message&quot;: string,
  &quot;details&quot;: [
    {
      &quot;@type&quot;: string,
      field1: ...,
      ...
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  code  `

`  integer  `

The status code, which should be an enum value of `  google.rpc.Code  ` .

`  message  `

`  string  `

A developer-facing error message, which should be in English. Any user-facing error message should be localized and sent in the `  google.rpc.Status.details  ` field, or localized by the client.

`  details[]  `

`  object  `

A list of messages that carry the error details. There is a common set of message types for APIs to use.

An object containing fields of an arbitrary type. An additional field `  "@type"  ` contains a URI identifying the type. Example: `  { "id": 1234, "@type": "types.example.com/standard/id" }  ` .

### Any

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
  &quot;typeUrl&quot;: string,
  &quot;value&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  typeUrl  `

`  string  `

A URL/resource name that uniquely identifies the type of the serialized protocol buffer message. This string must contain at least one "/" character. The last segment of the URL's path must represent the fully qualified name of the type (as in `  path/google.protobuf.Duration  ` ). The name should be in a canonical form (e.g., leading "." is not accepted).

In practice, teams usually precompile into the binary all types that they expect it to use in the context of Any. However, for URLs which use the scheme `  http  ` , `  https  ` , or no scheme, one can optionally set up a type server that maps type URLs to message definitions as follows:

  - If no scheme is provided, `  https  ` is assumed.
  - An HTTP GET on the URL must yield a `  google.protobuf.Type  ` value in binary format, or produce an error.
  - Applications are allowed to cache lookup results based on the URL, or have them precompiled into a binary to avoid any lookup. Therefore, binary compatibility needs to be preserved on changes to types. (Use versioned type names to manage breaking changes.)

Note: this functionality is not currently available in the official protobuf release, and it is not used for type URLs beginning with type.googleapis.com. As of May 2023, there are no widely used type server implementations and no plans to implement one.

Schemes other than `  http  ` , `  https  ` (or the empty scheme) might be used with implementation specific semantics.

`  value  `

`  string ( bytes format)  `

Must be a valid serialized protocol buffer of the above specified type.

A base64-encoded string.

### QuorumInfo

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
  &quot;quorumType&quot;: {
    object (QuorumType)
  },
  &quot;initiator&quot;: enum (Initiator),
  &quot;startTime&quot;: string,
  &quot;etag&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  quorumType  `

`  object ( QuorumType  ` )

Output only. The type of this quorum. See `  QuorumType  ` for more information about quorum type specifications.

`  initiator  `

`  enum ( Initiator  ` )

Output only. Whether this `  ChangeQuorum  ` is Google or User initiated.

`  startTime  `

`  string ( Timestamp  ` format)

Output only. The timestamp when the request was triggered.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  etag  `

`  string  `

Output only. The etag is used for optimistic concurrency control as a way to help prevent simultaneous `  ChangeQuorum  ` requests that might create a race condition.

### QuorumType

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

  // Union field type can be only one of the following:
  &quot;singleRegion&quot;: {
    object (SingleRegionQuorum)
  },
  &quot;dualRegion&quot;: {
    object (DualRegionQuorum)
  }
  // End of list of possible types for union field type.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  type  ` . The type of quorum. `  type  ` can be only one of the following:

`  singleRegion  `

`  object ( SingleRegionQuorum  ` )

Single-region quorum type.

`  dualRegion  `

`  object ( DualRegionQuorum  ` )

Dual-region quorum type.

### SingleRegionQuorum

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
  &quot;servingLocation&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  servingLocation  `

`  string  `

Required. The location of the serving region, for example, "us-central1". The location must be one of the regions within the dual-region instance configuration of your database. The list of valid locations is available using the \[GetInstanceConfig\]\[InstanceAdmin.GetInstanceConfig\] API.

This should only be used if you plan to change quorum to the single-region quorum type.

### Tool Annotations

Destructive Hint: ❌ | Idempotent Hint: ❌ | Read Only Hint: ✅ | Open World Hint: ❌
