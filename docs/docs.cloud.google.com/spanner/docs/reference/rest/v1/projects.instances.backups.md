  - [Resource: Backup](#Backup)
      - [JSON representation](#Backup.SCHEMA_REPRESENTATION)
  - [State](#State)
  - [BackupInstancePartition](#BackupInstancePartition)
      - [JSON representation](#BackupInstancePartition.SCHEMA_REPRESENTATION)
  - [Methods](#METHODS_SUMMARY)

## Resource: Backup

A backup of a Cloud Spanner database.

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
  &quot;database&quot;: string,
  &quot;versionTime&quot;: string,
  &quot;expireTime&quot;: string,
  &quot;name&quot;: string,
  &quot;createTime&quot;: string,
  &quot;sizeBytes&quot;: string,
  &quot;freeableSizeBytes&quot;: string,
  &quot;exclusiveSizeBytes&quot;: string,
  &quot;state&quot;: enum (State),
  &quot;referencingDatabases&quot;: [
    string
  ],
  &quot;encryptionInfo&quot;: {
    object (EncryptionInfo)
  },
  &quot;encryptionInformation&quot;: [
    {
      object (EncryptionInfo)
    }
  ],
  &quot;databaseDialect&quot;: enum (DatabaseDialect),
  &quot;referencingBackups&quot;: [
    string
  ],
  &quot;maxExpireTime&quot;: string,
  &quot;backupSchedules&quot;: [
    string
  ],
  &quot;incrementalBackupChainId&quot;: string,
  &quot;oldestVersionTime&quot;: string,
  &quot;instancePartitions&quot;: [
    {
      object (BackupInstancePartition)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  database  `

`  string  `

Required for the `  backups.create  ` operation. Name of the database from which this backup was created. This needs to be in the same instance as the backup. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` .

`  versionTime  `

`  string ( Timestamp  ` format)

The backup will contain an externally consistent copy of the database at the timestamp specified by `  versionTime  ` . If `  versionTime  ` is not specified, the system will set `  versionTime  ` to the `  createTime  ` of the backup.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  expireTime  `

`  string ( Timestamp  ` format)

Required for the `  backups.create  ` operation. The expiration time of the backup, with microseconds granularity that must be at least 6 hours and at most 366 days from the time the backups.create request is processed. Once the `  expireTime  ` has passed, the backup is eligible to be automatically deleted by Cloud Spanner to free the resources used by the backup.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  name  `

`  string  `

Output only for the `  backups.create  ` operation. Required for the `  backups.patch  ` operation.

A globally unique identifier for the backup which cannot be changed. Values are of the form `  projects/<project>/instances/<instance>/backups/[a-z][a-z0-9_\-]*[a-z0-9]  ` The final segment of the name must be between 2 and 60 characters in length.

The backup is stored in the location(s) specified in the instance configuration of the instance containing the backup, identified by the prefix of the backup name of the form `  projects/<project>/instances/<instance>  ` .

`  createTime  `

`  string ( Timestamp  ` format)

Output only. The time the `  backups.create  ` request is received. If the request does not specify `  versionTime  ` , the `  versionTime  ` of the backup will be equivalent to the `  createTime  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  sizeBytes  `

`  string ( int64 format)  `

Output only. Size of the backup in bytes. For a backup in an incremental backup chain, this is the sum of the `  exclusiveSizeBytes  ` of itself and all older backups in the chain.

`  freeableSizeBytes  `

`  string ( int64 format)  `

Output only. The number of bytes that will be freed by deleting this backup. This value will be zero if, for example, this backup is part of an incremental backup chain and younger backups in the chain require that we keep its data. For backups not in an incremental backup chain, this is always the size of the backup. This value may change if backups on the same chain get created, deleted or expired.

`  exclusiveSizeBytes  `

`  string ( int64 format)  `

Output only. For a backup in an incremental backup chain, this is the storage space needed to keep the data that has changed since the previous backup. For all other backups, this is always the size of the backup. This value may change if backups on the same chain get deleted or expired.

This field can be used to calculate the total storage space used by a set of backups. For example, the total space used by all backups of a database can be computed by summing up this field.

`  state  `

`  enum ( State  ` )

Output only. The current state of the backup.

`  referencingDatabases[]  `

`  string  `

Output only. The names of the restored databases that reference the backup. The database names are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` . Referencing databases may exist in different instances. The existence of any referencing database prevents the backup from being deleted. When a restored database from the backup enters the `  READY  ` state, the reference to the backup is removed.

`  encryptionInfo  `

`  object ( EncryptionInfo  ` )

Output only. The encryption information for the backup.

`  encryptionInformation[]  `

`  object ( EncryptionInfo  ` )

Output only. The encryption information for the backup, whether it is protected by one or more KMS keys. The information includes all Cloud KMS key versions used to encrypt the backup. The `  encryptionStatus  ` field inside of each `  EncryptionInfo  ` is not populated. At least one of the key versions must be available for the backup to be restored. If a key version is revoked in the middle of a restore, the restore behavior is undefined.

`  databaseDialect  `

`  enum ( DatabaseDialect  ` )

Output only. The database dialect information for the backup.

`  referencingBackups[]  `

`  string  `

Output only. The names of the destination backups being created by copying this source backup. The backup names are of the form `  projects/<project>/instances/<instance>/backups/<backup>  ` . Referencing backups may exist in different instances. The existence of any referencing backup prevents the backup from being deleted. When the copy operation is done (either successfully completed or cancelled or the destination backup is deleted), the reference to the backup is removed.

`  maxExpireTime  `

`  string ( Timestamp  ` format)

Output only. The max allowed expiration time of the backup, with microseconds granularity. A backup's expiration time can be configured in multiple APIs: backups.create, backups.patch, backups.copy. When updating or copying an existing backup, the expiration time specified must be less than `  Backup.max_expire_time  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  backupSchedules[]  `

`  string  `

Output only. List of backup schedule URIs that are associated with creating this backup. This is only applicable for scheduled backups, and is empty for on-demand backups.

To optimize for storage, whenever possible, multiple schedules are collapsed together to create one backup. In such cases, this field captures the list of all backup schedule URIs that are associated with creating this backup. If collapsing is not done, then this field captures the single backup schedule URI associated with creating this backup.

`  incrementalBackupChainId  `

`  string  `

Output only. Populated only for backups in an incremental backup chain. Backups share the same chain id if and only if they belong to the same incremental backup chain. Use this field to determine which backups are part of the same incremental backup chain. The ordering of backups in the chain can be determined by ordering the backup `  versionTime  ` .

`  oldestVersionTime  `

`  string ( Timestamp  ` format)

Output only. Data deleted at a time older than this is guaranteed not to be retained in order to support this backup. For a backup in an incremental backup chain, this is the version time of the oldest backup that exists or ever existed in the chain. For all other backups, this is the version time of the backup. This field can be used to understand what data is being retained by the backup system.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  instancePartitions[]  `

`  object ( BackupInstancePartition  ` )

Output only. The instance partition storing the backup.

This is the same as the list of the instance partitions that the database recorded at the backup's `  versionTime  ` .

## State

Indicates the current state of the backup.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The pending backup is still being created. Operations on the backup may fail with `  FAILED_PRECONDITION  ` in this state.

`  READY  `

The backup is complete and ready for use.

## BackupInstancePartition

Instance partition information for the backup.

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
  &quot;instancePartition&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instancePartition  `

`  string  `

A unique identifier for the instance partition. Values are of the form `  projects/<project>/instances/<instance>/instancePartitions/<instancePartitionId>  `

## Methods

### `             copy           `

Starts copying a Cloud Spanner Backup.

### `             create           `

Starts creating a new Cloud Spanner Backup.

### `             delete           `

Deletes a pending or completed `  Backup  ` .

### `             get           `

Gets metadata on a pending or completed `  Backup  ` .

### `             getIamPolicy           `

Gets the access control policy for a database or backup resource.

### `             list           `

Lists completed and pending backups.

### `             patch           `

Updates a pending or completed `  Backup  ` .

### `             setIamPolicy           `

Sets the access control policy on a database or backup resource.

### `             testIamPermissions           `

Returns permissions that the caller has on the specified database or backup resource.
