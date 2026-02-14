  - [Resource: BackupSchedule](#BackupSchedule)
      - [JSON representation](#BackupSchedule.SCHEMA_REPRESENTATION)
  - [BackupScheduleSpec](#BackupScheduleSpec)
      - [JSON representation](#BackupScheduleSpec.SCHEMA_REPRESENTATION)
  - [CrontabSpec](#CrontabSpec)
      - [JSON representation](#CrontabSpec.SCHEMA_REPRESENTATION)
  - [CreateBackupEncryptionConfig](#CreateBackupEncryptionConfig)
      - [JSON representation](#CreateBackupEncryptionConfig.SCHEMA_REPRESENTATION)
  - [EncryptionType](#EncryptionType)
  - [FullBackupSpec](#FullBackupSpec)
  - [IncrementalBackupSpec](#IncrementalBackupSpec)
  - [Methods](#METHODS_SUMMARY)

## Resource: BackupSchedule

BackupSchedule expresses the automated backup creation specification for a Spanner database.

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
  &quot;spec&quot;: {
    object (BackupScheduleSpec)
  },
  &quot;retentionDuration&quot;: string,
  &quot;encryptionConfig&quot;: {
    object (CreateBackupEncryptionConfig)
  },
  &quot;updateTime&quot;: string,

  // Union field backup_type_spec can be only one of the following:
  &quot;fullBackupSpec&quot;: {
    object (FullBackupSpec)
  },
  &quot;incrementalBackupSpec&quot;: {
    object (IncrementalBackupSpec)
  }
  // End of list of possible types for union field backup_type_spec.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Identifier. Output only for the `  backupSchedules.create  ` operation. Required for the `  backupSchedules.patch  ` operation. A globally unique identifier for the backup schedule which cannot be changed. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>/backupSchedules/[a-z][a-z0-9_\-]*[a-z0-9]  ` The final segment of the name must be between 2 and 60 characters in length.

`  spec  `

`  object ( BackupScheduleSpec  ` )

Optional. The schedule specification based on which the backup creations are triggered.

`  retentionDuration  `

`  string ( Duration  ` format)

Optional. The retention duration of a backup that must be at least 6 hours and at most 366 days. The backup is eligible to be automatically deleted once the retention period has elapsed.

A duration in seconds with up to nine fractional digits, ending with ' `  s  ` '. Example: `  "3.5s"  ` .

`  encryptionConfig  `

`  object ( CreateBackupEncryptionConfig  ` )

Optional. The encryption configuration that is used to encrypt the backup. If this field is not specified, the backup uses the same encryption configuration as the database.

`  updateTime  `

`  string ( Timestamp  ` format)

Output only. The timestamp at which the schedule was last updated. If the schedule has never been updated, this field contains the timestamp when the schedule was first created.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

Union field `  backup_type_spec  ` . Required. Backup type specification determines the type of backup that is created by the backup schedule. `  backup_type_spec  ` can be only one of the following:

`  fullBackupSpec  `

`  object ( FullBackupSpec  ` )

The schedule creates only full backups.

`  incrementalBackupSpec  `

`  object ( IncrementalBackupSpec  ` )

The schedule creates incremental backup chains.

## BackupScheduleSpec

Defines specifications of the backup schedule.

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

  // Union field schedule_spec can be only one of the following:
  &quot;cronSpec&quot;: {
    object (CrontabSpec)
  }
  // End of list of possible types for union field schedule_spec.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  schedule_spec  ` . Required. `  schedule_spec  ` can be only one of the following:

`  cronSpec  `

`  object ( CrontabSpec  ` )

Cron style schedule specification.

## CrontabSpec

CrontabSpec can be used to specify the version time and frequency at which the backup is created.

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
  &quot;text&quot;: string,
  &quot;timeZone&quot;: string,
  &quot;creationWindow&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  text  `

`  string  `

Required. Textual representation of the crontab. User can customize the backup frequency and the backup version time using the cron expression. The version time must be in UTC timezone. The backup will contain an externally consistent copy of the database at the version time.

Full backups must be scheduled a minimum of 12 hours apart and incremental backups must be scheduled a minimum of 4 hours apart. Examples of valid cron specifications:

  - `  0 2/12 * * *  ` : every 12 hours at (2, 14) hours past midnight in UTC.
  - `  0 2,14 * * *  ` : every 12 hours at (2, 14) hours past midnight in UTC.
  - `  0 */4 * * *  ` : (incremental backups only) every 4 hours at (0, 4, 8, 12, 16, 20) hours past midnight in UTC.
  - `  0 2 * * *  ` : once a day at 2 past midnight in UTC.
  - `  0 2 * * 0  ` : once a week every Sunday at 2 past midnight in UTC.
  - `  0 2 8 * *  ` : once a month on 8th day at 2 past midnight in UTC.

`  timeZone  `

`  string  `

Output only. The time zone of the times in `  CrontabSpec.text  ` . Currently, only UTC is supported.

`  creationWindow  `

`  string ( Duration  ` format)

Output only. Scheduled backups contain an externally consistent copy of the database at the version time specified in `  schedule_spec.cron_spec  ` . However, Spanner might not initiate the creation of the scheduled backups at that version time. Spanner initiates the creation of scheduled backups within the time window bounded by the versionTime specified in `  schedule_spec.cron_spec  ` and versionTime + `  creationWindow  ` .

A duration in seconds with up to nine fractional digits, ending with ' `  s  ` '. Example: `  "3.5s"  ` .

## CreateBackupEncryptionConfig

Encryption configuration for the backup to create.

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

Optional. Specifies the KMS configuration for the one or more keys used to protect the backup. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kmsKeyName>  ` .

The keys referenced by `  kmsKeyNames  ` must fully cover all regions of the backup's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

## EncryptionType

Encryption types for the backup.

Enums

`  ENCRYPTION_TYPE_UNSPECIFIED  `

Unspecified. Do not use.

`  USE_DATABASE_ENCRYPTION  `

Use the same encryption configuration as the database. This is the default option when `  encryptionConfig  ` is empty. For example, if the database is using `  Customer_Managed_Encryption  ` , the backup will be using the same Cloud KMS key as the database.

`  GOOGLE_DEFAULT_ENCRYPTION  `

Use Google default encryption.

`  CUSTOMER_MANAGED_ENCRYPTION  `

Use customer managed encryption. If specified, `  kmsKeyName  ` must contain a valid Cloud KMS key.

## FullBackupSpec

This type has no fields.

The specification for full backups. A full backup stores the entire contents of the database at a given version time.

## IncrementalBackupSpec

This type has no fields.

The specification for incremental backup chains. An incremental backup stores the delta of changes between a previous backup and the database contents at a given version time. An incremental backup chain consists of a full backup and zero or more successive incremental backups. The first backup created for an incremental backup chain is always a full backup.

## Methods

### `             create           `

Creates a new backup schedule.

### `             delete           `

Deletes a backup schedule.

### `             get           `

Gets backup schedule for the input schedule name.

### `             getIamPolicy           `

Gets the access control policy for a database or backup resource.

### `             list           `

Lists all the backup schedules for the database.

### `             patch           `

Updates a backup schedule.

### `             setIamPolicy           `

Sets the access control policy on a database or backup resource.

### `             testIamPermissions           `

Returns permissions that the caller has on the specified database or backup resource.
