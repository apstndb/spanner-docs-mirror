---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules
title: 'REST Resource: projects.instances.databases.backupSchedules'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [Resource: BackupSchedule](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule.SCHEMA_REPRESENTATION)
- [BackupScheduleSpec](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupScheduleSpec)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupScheduleSpec.SCHEMA_REPRESENTATION)
- [CrontabSpec](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CrontabSpec)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CrontabSpec.SCHEMA_REPRESENTATION)
- [CreateBackupEncryptionConfig](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CreateBackupEncryptionConfig)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CreateBackupEncryptionConfig.SCHEMA_REPRESENTATION)
- [EncryptionType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#EncryptionType)
- [FullBackupSpec](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#FullBackupSpec)
- [IncrementalBackupSpec](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#IncrementalBackupSpec)
- [Methods](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#METHODS_SUMMARY)

## Resource: BackupSchedule

BackupSchedule expresses the automated backup creation specification for a Spanner database.

**JSON representation**

```
{
  "name": string,
  "spec": {
    object (BackupScheduleSpec)
  },
  "retentionDuration": string,
  "encryptionConfig": {
    object (CreateBackupEncryptionConfig)
  },
  "updateTime": string,

  // Union field backup_type_spec can be only one of the following:
  "fullBackupSpec": {
    object (FullBackupSpec)
  },
  "incrementalBackupSpec": {
    object (IncrementalBackupSpec)
  }
  // End of list of possible types for union field backup_type_spec.
}
```

| Fields                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                                                                                                                                                                                          | `string` Identifier. Output only for the [`backupSchedules.create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create#google.spanner.admin.database.v1.DatabaseAdmin.CreateBackupSchedule) operation. Required for the [`backupSchedules.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/patch#google.spanner.admin.database.v1.DatabaseAdmin.UpdateBackupSchedule) operation. A globally unique identifier for the backup schedule which cannot be changed. Values are of the form `projects/<project>/instances/<instance>/databases/<database>/backupSchedules/[a-z][a-z0-9_\-]*[a-z0-9]` The final segment of the name must be between 2 and 60 characters in length. |
| `spec`                                                                                                                                                                                          | `object ( `[`BackupScheduleSpec`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupScheduleSpec)` )` Optional. The schedule specification based on which the backup creations are triggered.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `retentionDuration`                                                                                                                                                                             | `string ( `[`Duration`](https://protobuf.dev/reference/protobuf/google.protobuf/#duration)` format)` Optional. The retention duration of a backup that must be at least 6 hours and at most 366 days. The backup is eligible to be automatically deleted once the retention period has elapsed. A duration in seconds with up to nine fractional digits, ending with ' `s` '. Example: `"3.5s"` .                                                                                                                                                                                                                                                                                                                                                                                                          |
| `encryptionConfig`                                                                                                                                                                              | `object ( `[`CreateBackupEncryptionConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CreateBackupEncryptionConfig)` )` Optional. The encryption configuration that is used to encrypt the backup. If this field is not specified, the backup uses the same encryption configuration as the database.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `updateTime`                                                                                                                                                                                    | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Output only. The timestamp at which the schedule was last updated. If the schedule has never been updated, this field contains the timestamp when the schedule was first created. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                                                                                                                                    |
| Union field `backup_type_spec` . Required. Backup type specification determines the type of backup that is created by the backup schedule. `backup_type_spec` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `fullBackupSpec`                                                                                                                                                                                | `object ( `[`FullBackupSpec`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#FullBackupSpec)` )` The schedule creates only full backups.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `incrementalBackupSpec`                                                                                                                                                                         | `object ( `[`IncrementalBackupSpec`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#IncrementalBackupSpec)` )` The schedule creates incremental backup chains.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

## BackupScheduleSpec

Defines specifications of the backup schedule.

**JSON representation**

```
{

  // Union field schedule_spec can be only one of the following:
  "cronSpec": {
    object (CrontabSpec)
  }
  // End of list of possible types for union field schedule_spec.
}
```

| Fields                                                                                    |                                                                                                                                                                                          |
|-------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `schedule_spec` . Required. `schedule_spec` can be only one of the following: |                                                                                                                                                                                          |
| `cronSpec`                                                                                | `object ( `[`CrontabSpec`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CrontabSpec)` )` Cron style schedule specification. |

## CrontabSpec

CrontabSpec can be used to specify the version time and frequency at which the backup is created.

**JSON representation**

```
{
  "text": string,
  "timeZone": string,
  "creationWindow": string
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
<td><code>text</code></td>
<td><p><code>string</code></p>
<p>Required. Textual representation of the crontab. User can customize the backup frequency and the backup version time using the cron expression. The version time must be in UTC timezone. The backup will contain an externally consistent copy of the database at the version time.</p>
<p>Full backups must be scheduled a minimum of 12 hours apart and incremental backups must be scheduled a minimum of 4 hours apart. Examples of valid cron specifications:</p>
<ul>
<li><code>0 2/12 * * *</code> : every 12 hours at (2, 14) hours past midnight in UTC.</li>
<li><code>0 2,14 * * *</code> : every 12 hours at (2, 14) hours past midnight in UTC.</li>
<li><code>0 */4 * * *</code> : (incremental backups only) every 4 hours at (0, 4, 8, 12, 16, 20) hours past midnight in UTC.</li>
<li><code>0 2 * * *</code> : once a day at 2 past midnight in UTC.</li>
<li><code>0 2 * * 0</code> : once a week every Sunday at 2 past midnight in UTC.</li>
<li><code>0 2 8 * *</code> : once a month on 8th day at 2 past midnight in UTC.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>timeZone</code></td>
<td><p><code>string</code></p>
<p>Output only. The time zone of the times in <code>CrontabSpec.text</code> . Currently, only UTC is supported.</p></td>
</tr>
<tr class="odd">
<td><code>creationWindow</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#duration"><code>Duration</code></a><code> format)</code></p>
<p>Output only. Scheduled backups contain an externally consistent copy of the database at the version time specified in <code>schedule_spec.cron_spec</code> . However, Spanner might not initiate the creation of the scheduled backups at that version time. Spanner initiates the creation of scheduled backups within the time window bounded by the versionTime specified in <code>schedule_spec.cron_spec</code> and versionTime + <code>creationWindow</code> .</p>
<p>A duration in seconds with up to nine fractional digits, ending with ' <code>s</code> '. Example: <code>"3.5s"</code> .</p></td>
</tr>
</tbody>
</table>

## CreateBackupEncryptionConfig

Encryption configuration for the backup to create.

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
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#EncryptionType"><code>EncryptionType</code></a><code> )</code></p>
<p>Required. The encryption type of the backup.</p></td>
</tr>
<tr class="even">
<td><code>kmsKeyName</code></td>
<td><p><code>string</code></p>
<p>Optional. This field is maintained for backwards compatibility. For new callers, we recommend using <code>kmsKeyNames</code> to specify the KMS key. Only use <code>kmsKeyName</code> if the location of the KMS key matches the database instance's configuration (location) exactly. For example, if the KMS location is in <code>us-central1</code> or <code>nam3</code> , then the database instance must also be in <code>us-central1</code> or <code>nam3</code> .</p>
<p>The Cloud KMS key that is used to encrypt and decrypt the restored database. Set this field only when <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CreateBackupEncryptionConfig.FIELDS.encryption_type"><code>encryptionType</code></a> is <code>CUSTOMER_MANAGED_ENCRYPTION</code> . Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p></td>
</tr>
<tr class="odd">
<td><code>kmsKeyNames[]</code></td>
<td><p><code>string</code></p>
<p>Optional. Specifies the KMS configuration for the one or more keys used to protect the backup. Values are of the form <code>projects/&lt;project&gt;/locations/&lt;location&gt;/keyRings/&lt;key_ring&gt;/cryptoKeys/&lt;kmsKeyName&gt;</code> .</p>
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

| Enums                         |                                                                                                                                                                                                                                                                                                                                                                                                      |
|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ENCRYPTION_TYPE_UNSPECIFIED` | Unspecified. Do not use.                                                                                                                                                                                                                                                                                                                                                                             |
| `USE_DATABASE_ENCRYPTION`     | Use the same encryption configuration as the database. This is the default option when [`encryptionConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#CreateBackupEncryptionConfig) is empty. For example, if the database is using `Customer_Managed_Encryption` , the backup will be using the same Cloud KMS key as the database. |
| `GOOGLE_DEFAULT_ENCRYPTION`   | Use Google default encryption.                                                                                                                                                                                                                                                                                                                                                                       |
| `CUSTOMER_MANAGED_ENCRYPTION` | Use customer managed encryption. If specified, `kmsKeyName` must contain a valid Cloud KMS key.                                                                                                                                                                                                                                                                                                      |

## FullBackupSpec

This type has no fields.

The specification for full backups. A full backup stores the entire contents of the database at a given version time.

## IncrementalBackupSpec

This type has no fields.

The specification for incremental backup chains. An incremental backup stores the delta of changes between a previous backup and the database contents at a given version time. An incremental backup chain consists of a full backup and zero or more successive incremental backups. The first backup created for an incremental backup chain is always a full backup.

| Methods                                                                                                                                              |                                                                                       |
|------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| [`create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/create)                         | Creates a new backup schedule.                                                        |
| [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/delete)                         | Deletes a backup schedule.                                                            |
| [`get`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/get)                               | Gets backup schedule for the input schedule name.                                     |
| [`getIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/getIamPolicy)             | Gets the access control policy for a database or backup resource.                     |
| [`list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/list)                             | Lists all the backup schedules for the database.                                      |
| [`patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/patch)                           | Updates a backup schedule.                                                            |
| [`setIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/setIamPolicy)             | Sets the access control policy on a database or backup resource.                      |
| [`testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules/testIamPermissions) | Returns permissions that the caller has on the specified database or backup resource. |
