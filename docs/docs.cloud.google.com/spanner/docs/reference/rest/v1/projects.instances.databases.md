  - [Resource: Database](#Database)
      - [JSON representation](#Database.SCHEMA_REPRESENTATION)
  - [State](#State)
  - [RestoreInfo](#RestoreInfo)
      - [JSON representation](#RestoreInfo.SCHEMA_REPRESENTATION)
  - [RestoreSourceType](#RestoreSourceType)
  - [BackupInfo](#BackupInfo)
      - [JSON representation](#BackupInfo.SCHEMA_REPRESENTATION)
  - [EncryptionConfig](#EncryptionConfig)
      - [JSON representation](#EncryptionConfig.SCHEMA_REPRESENTATION)
  - [QuorumInfo](#QuorumInfo)
      - [JSON representation](#QuorumInfo.SCHEMA_REPRESENTATION)
  - [QuorumType](#QuorumType)
      - [JSON representation](#QuorumType.SCHEMA_REPRESENTATION)
  - [SingleRegionQuorum](#SingleRegionQuorum)
      - [JSON representation](#SingleRegionQuorum.SCHEMA_REPRESENTATION)
  - [DualRegionQuorum](#DualRegionQuorum)
  - [Initiator](#Initiator)
  - [Methods](#METHODS_SUMMARY)

## Resource: Database

A Cloud Spanner database.

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

Output only. For databases that are using customer managed encryption, this field contains the encryption information for the database, such as all Cloud KMS key versions that are in use. The `  encryptionStatus  ` field inside of each `  EncryptionInfo  ` is not populated.

For databases that are using Google default or other types of encryption, this field is empty.

This field is propagated lazily from the backend. There might be a delay from when a key version is being used and when it appears in this field.

`  versionRetentionPeriod  `

`  string  `

Output only. The period in which Cloud Spanner retains all versions of data for the database. This is the same as the value of versionRetentionPeriod database option set using `  databases.updateDdl  ` . Defaults to 1 hour, if not set.

`  earliestVersionTime  `

`  string ( Timestamp  ` format)

Output only. Earliest timestamp at which older versions of the data can be read. This value is continuously updated by Cloud Spanner and becomes stale the moment it is queried. If you are using this value to recover data, make sure to account for the time from the moment when the value is queried to the moment when you initiate the recovery.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  defaultLeader  `

`  string  `

Output only. The read-write region which contains the database's leader replicas.

This is the same as the value of defaultLeader database option set using DatabaseAdmin.CreateDatabase or DatabaseAdmin.UpdateDatabaseDdl. If not explicitly set, this is empty.

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

## State

Indicates the current state of the database.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The database is still being created. Operations on the database may fail with `  FAILED_PRECONDITION  ` in this state.

`  READY  `

The database is fully created and ready for use.

`  READY_OPTIMIZING  `

The database is fully created and ready for use, but is still being optimized for performance and cannot handle full load.

In this state, the database still references the backup it was restore from, preventing the backup from being deleted. When optimizations are complete, the full performance of the database will be restored, and the database will transition to `  READY  ` state.

## RestoreInfo

Information about the database restore.

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

## RestoreSourceType

Indicates the type of the restore source.

Enums

`  TYPE_UNSPECIFIED  `

No restore associated.

`  BACKUP  `

A backup was used as the source of the restore.

## BackupInfo

Information about a backup.

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

The backup contains an externally consistent copy of `  sourceDatabase  ` at the timestamp specified by `  versionTime  ` . If the `  backups.create  ` request did not specify `  versionTime  ` , the `  versionTime  ` of the backup is equivalent to the `  createTime  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  createTime  `

`  string ( Timestamp  ` format)

The time the `  backups.create  ` request was received.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  sourceDatabase  `

`  string  `

Name of the database the backup was created from.

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

The Cloud KMS key to be used for encrypting and decrypting the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

`  kmsKeyNames[]  `

`  string  `

Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

The keys referenced by `  kmsKeyNames  ` must fully cover all regions of the database's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

## QuorumInfo

Information about the dual-region quorum.

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

Output only. Whether this `  databases.changequorum  ` is Google or User initiated.

`  startTime  `

`  string ( Timestamp  ` format)

Output only. The timestamp when the request was triggered.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  etag  `

`  string  `

Output only. The etag is used for optimistic concurrency control as a way to help prevent simultaneous `  databases.changequorum  ` requests that might create a race condition.

## QuorumType

Information about the database quorum type. This only applies to dual-region instance configs.

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

## SingleRegionQuorum

Message type for a single-region quorum.

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

Required. The location of the serving region, for example, "us-central1". The location must be one of the regions within the dual-region instance configuration of your database. The list of valid locations is available using the \[instanceConfigs.get\]\[InstanceAdmin.GetInstanceConfig\] API.

This should only be used if you plan to change quorum to the single-region quorum type.

## DualRegionQuorum

This type has no fields.

Message type for a dual-region quorum. Currently this type has no options.

## Initiator

Describes who initiated `  databases.changequorum  ` .

Enums

`  INITIATOR_UNSPECIFIED  `

Unspecified.

`  GOOGLE  `

`  databases.changequorum  ` initiated by Google.

`  USER  `

`  databases.changequorum  ` initiated by User.

## Methods

### `             addSplitPoints           `

Adds split points to specified tables and indexes of a database.

### `             changequorum           `

`  ChangeQuorum  ` is strictly restricted to databases that use dual-region instance configurations.

### `             create           `

Creates a new Spanner database and starts to prepare it for serving.

### `             dropDatabase           `

Drops (aka deletes) a Cloud Spanner database.

### `             get           `

Gets the state of a Cloud Spanner database.

### `             getDdl           `

Returns the schema of a Cloud Spanner database as a list of formatted DDL statements.

### `             getIamPolicy           `

Gets the access control policy for a database or backup resource.

### `             list           `

Lists Cloud Spanner databases.

### `             patch           `

Updates a Cloud Spanner database.

### `             restore           `

Create a new database by restoring from a completed backup.

### `             setIamPolicy           `

Sets the access control policy on a database or backup resource.

### `             testIamPermissions           `

Returns permissions that the caller has on the specified database or backup resource.

### `             updateDdl           `

Updates the schema of a Cloud Spanner database by creating/altering/dropping tables, columns, indexes, etc.
