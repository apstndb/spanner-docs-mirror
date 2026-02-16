This page describes Spanner backup schedule operations and explains how to set up a backup schedule for your database.

## Before you begin

  - To get the permissions that you need to create and manage backup schedules, ask your administrator to grant you the following IAM roles on the instance:
    
      - Create, view, update, and delete backup schedules: [Cloud Spanner Backup Admin](/iam/docs/roles-permissions/spanner#spanner.backupAdmin) ( `  roles/spanner.backupAdmin  ` )
      - Create and view backup schedules: [Cloud Spanner Backup Writer](/iam/docs/roles-permissions/spanner#spanner.backupWriter) ( `  roles/spanner.backupWriter  ` )
    
    **Note:** [IAM basic roles](/iam/docs/roles-overview#basic) might also contain permissions to create and manage backup schedules. You shouldn't grant basic roles in a production environment, but you can grant them in a development or test environment.

## Create a backup schedule

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

<!-- end list -->

1.  Click the instance containing the database.

2.  Click the database.

3.  In the navigation menu, click the **Backup/Restore** .

4.  Click **Create backup schedule** .

5.  Fill out the form, then click **Create** .

### gcloud

Before using any of the command data below, make the following replacements:

  - SCHEDULE\_ID : the backup schedule ID.
  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the ID of the instance where you want to create the backup schedule.
  - DATABASE\_ID : the ID of the database where you want to create the backup schedule.
  - RETENTION\_DURATION : the retention duration of the backups created by the schedule. For example, if you want the retention duration to be one day, you can use `  86400s  ` .
  - CRONTAB\_EXPRESSION : the crontab expression for the backup schedule frequency. For example, if you want the backup schedule frequency to be every 12 hours, you can use `  0 12 * * *  ` .
  - BACKUP\_TYPE : whether it's a full backup schedule or an incremental backup schedule. Possible values are `  full-backup  ` or `  incremental-backup  ` .
  - ENCRYPTION\_TYPE : the encryption type of backups created by the backup schedule. Valid values are `  USE_DATABASE_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` or `  kmsKeyNames  ` .

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules create SCHEDULE_ID \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID \
  --retention-duration=RETENTION_DURATION \
  --cron="CRONTAB_EXPRESSION" \
  --backup-type=BACKUP_TYPE \
  --encryption-type=ENCRYPTION_TYPE
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules create SCHEDULE_ID `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID `
  --retention-duration=RETENTION_DURATION `
  --cron="CRONTAB_EXPRESSION" `
  --backup-type=BACKUP_TYPE `
  --encryption-type=ENCRYPTION_TYPE
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules create SCHEDULE_ID ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID ^
  --retention-duration=RETENTION_DURATION ^
  --cron="CRONTAB_EXPRESSION" ^
  --backup-type=BACKUP_TYPE ^
  --encryption-type=ENCRYPTION_TYPE
```

### REST v1

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the ID of the instance where you want to create the backup schedule.
  - DATABASE\_ID : the ID of the database where you want to create the backup schedule.
  - SCHEDULE\_ID : the backup schedule ID.
  - BACKUP\_TYPE : whether it's a full backup schedule or an incremental backup schedule. Possible values are `  fullBackupSpec  ` or `  incrementalBackupSpec  ` .
  - ENCRYPTION\_TYPE : the encryption type of backups created by the backup schedule. Possible values are `  USE_DATABASE_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` or `  kmsKeyNames  ` .
  - RETENTION\_DURATION : the retention duration of the backups created by the schedule.

HTTP method and URL:

``` text
POST https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules?backup_schedule_id=SCHEDULE_ID
```

Request JSON body:

``` text
{
  "retentionDuration": "RETENTION_DURATION",
  "spec": {
    "cronSpec": {
      "text": "0 2 * * *"
    }
  },
  "encryptionConfig": {
    "encryptionType": "ENCRYPTION_TYPE"
  },
  "BACKUP_TYPE": {}
}
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Save the request body in a file named `  request.json  ` , and execute the following command:

``` text
curl -X POST \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "Content-Type: application/json; charset=utf-8" \
     -d @request.json \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules?backup_schedule_id=SCHEDULE_ID"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Save the request body in a file named `  request.json  ` , and execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method POST `
    -Headers $headers `
    -ContentType: "application/json; charset=utf-8" `
    -InFile request.json `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules?backup_schedule_id=SCHEDULE_ID" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
  "name": "projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID",
  "retentionDuration": "86400s",
  "encryptionConfig": {
    "encryptionType": "USE_DATABASE_ENCRYPTION"
  },
  "spec": {
    "cronSpec": {
      "text": "0 2 * * *",
      "timeZone": "UTC",
      "creationWindow": "14400s"
    }
  },
  "BACKUP_TYPE": {},
  "updateTime": "2024-05-22T11:13:51.835590Z"
}
```

### Client libraries

### C\#

To create a full backup schedule, see the following sample code:

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Protobuf.WellKnownTypes;
using System;

public class CreateFullBackupScheduleSample
{
    public BackupSchedule CreateFullBackupSchedule(string projectId, string instanceId, string databaseId, string scheduleId)
    {
        DatabaseAdminClient client = DatabaseAdminClient.Create();

        BackupSchedule response = client.CreateBackupSchedule(
            new CreateBackupScheduleRequest
            {
                ParentAsDatabaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId),
                BackupScheduleId = scheduleId,
                BackupSchedule = new BackupSchedule
                {
                    Spec = new BackupScheduleSpec
                    {
                        CronSpec = new CrontabSpec
                        {
                            Text = "30 12 * * *",
                        }
                    },
                    RetentionDuration = new Duration
                    {
                        Seconds = 86400,
                    },
                    EncryptionConfig = new CreateBackupEncryptionConfig
                    {
                        EncryptionType = CreateBackupEncryptionConfig.Types.EncryptionType.UseDatabaseEncryption,
                    },
                    FullBackupSpec = new FullBackupSpec { },
                }
            });

        Console.WriteLine($"Created full backup schedule: {response}");
        return response;
    }
}
```

To create an incremental backup schedule, see the following sample code:

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using Google.Protobuf.WellKnownTypes;
using System;

public class CreateIncrementalBackupScheduleSample
{
    public BackupSchedule CreateIncrementalBackupSchedule(string projectId, string instanceId, string databaseId, string scheduleId)
    {
        DatabaseAdminClient client = DatabaseAdminClient.Create();

        BackupSchedule response = client.CreateBackupSchedule(
            new CreateBackupScheduleRequest
            {
                ParentAsDatabaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId),
                BackupScheduleId = scheduleId,
                BackupSchedule = new BackupSchedule
                {
                    Spec = new BackupScheduleSpec
                    {
                        CronSpec = new CrontabSpec
                        {
                            Text = "30 12 * * *",
                        }
                    },
                    RetentionDuration = new Duration
                    {
                        Seconds = 86400,
                    },
                    EncryptionConfig = new CreateBackupEncryptionConfig
                    {
                        EncryptionType = CreateBackupEncryptionConfig.Types.EncryptionType.GoogleDefaultEncryption,
                    },
                    IncrementalBackupSpec = new IncrementalBackupSpec { },
                }
            });

        Console.WriteLine($"Created incremental backup schedule: {response}");
        return response;
    }
}
```

### C++

To create a full backup schedule, see the following sample code:

``` cpp
void CreateFullBackupSchedule(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& backup_schedule_id) {
  google::spanner::admin::database::v1::BackupSchedule backup_schedule;
  *backup_schedule.mutable_full_backup_spec() = {};
  backup_schedule.mutable_spec()->mutable_cron_spec()->set_text("30 12 * * *");
  backup_schedule.mutable_retention_duration()->set_seconds(3600 * 24);

  google::spanner::admin::database::v1::CreateBackupEncryptionConfig
      encryption_config;
  encryption_config.set_encryption_type(
      google::spanner::admin::database::v1::CreateBackupEncryptionConfig::
          USE_DATABASE_ENCRYPTION);
  *backup_schedule.mutable_encryption_config() = std::move(encryption_config);

  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  google::spanner::admin::database::v1::CreateBackupScheduleRequest request;
  request.set_parent(db.FullName());
  request.set_backup_schedule_id(backup_schedule_id);
  *request.mutable_backup_schedule() = std::move(backup_schedule);

  auto created_backup_schedule = client.CreateBackupSchedule(request);
  if (!created_backup_schedule) {
    throw std::move(created_backup_schedule).status();
  }
  std::cout << "Backup schedule " << created_backup_schedule->name()
            << " created at "
            << *google::cloud::spanner::MakeTimestamp(
                   created_backup_schedule->update_time());
}
```

To create an incremental backup schedule, see the following sample code:

``` cpp
void CreateIncrementalBackupSchedule(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& backup_schedule_id) {
  google::spanner::admin::database::v1::BackupSchedule backup_schedule;
  *backup_schedule.mutable_incremental_backup_spec() = {};
  backup_schedule.mutable_spec()->mutable_cron_spec()->set_text("30 12 * * *");
  backup_schedule.mutable_retention_duration()->set_seconds(3600 * 24);

  google::spanner::admin::database::v1::CreateBackupEncryptionConfig
      encryption_config;
  encryption_config.set_encryption_type(
      google::spanner::admin::database::v1::CreateBackupEncryptionConfig::
          GOOGLE_DEFAULT_ENCRYPTION);
  *backup_schedule.mutable_encryption_config() = std::move(encryption_config);

  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  google::spanner::admin::database::v1::CreateBackupScheduleRequest request;
  request.set_parent(db.FullName());
  request.set_backup_schedule_id(backup_schedule_id);
  *request.mutable_backup_schedule() = std::move(backup_schedule);

  auto created_backup_schedule = client.CreateBackupSchedule(request);
  if (!created_backup_schedule) {
    throw std::move(created_backup_schedule).status();
  }
  std::cout << "Incremental backup schedule " << created_backup_schedule->name()
            << " created at "
            << *google::cloud::spanner::MakeTimestamp(
                   created_backup_schedule->update_time());
}
```

### Go

To create a full backup schedule, see the following sample code:

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/protobuf/types/known/durationpb"
)

func createFullBackupSchedule(w io.Writer, dbName string, scheduleId string) error {
 ctx := context.Background()

 client, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 // Create a schedule to create full backups daily at 12:30 AM, using the
 // database's encryption config, and retained for 24 hours.
 req := databasepb.CreateBackupScheduleRequest{
     Parent:           dbName,
     BackupScheduleId: scheduleId,
     BackupSchedule: &databasepb.BackupSchedule{
         Spec: &databasepb.BackupScheduleSpec{
             ScheduleSpec: &databasepb.BackupScheduleSpec_CronSpec{
                 CronSpec: &databasepb.CrontabSpec{
                     Text: "30 12 * * *",
                 },
             },
         },
         RetentionDuration: durationpb.New(24 * time.Hour),
         EncryptionConfig: &databasepb.CreateBackupEncryptionConfig{
             EncryptionType: databasepb.CreateBackupEncryptionConfig_USE_DATABASE_ENCRYPTION,
         },
         BackupTypeSpec: &databasepb.BackupSchedule_FullBackupSpec{},
     },
 }

 res, err := client.CreateBackupSchedule(ctx, &req)
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Created full backup schedule: %s", res)
 return nil
}
```

To create an incremental backup schedule, see the following sample code:

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/protobuf/types/known/durationpb"
)

func createIncrementalBackupSchedule(w io.Writer, dbName string, scheduleId string) error {
 ctx := context.Background()

 client, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 // Create a schedule to create incremental backups daily at 12:30 AM, using
 // Google-managed encryption, and retained for 24 hours.
 req := databasepb.CreateBackupScheduleRequest{
     Parent:           dbName,
     BackupScheduleId: scheduleId,
     BackupSchedule: &databasepb.BackupSchedule{
         Spec: &databasepb.BackupScheduleSpec{
             ScheduleSpec: &databasepb.BackupScheduleSpec_CronSpec{
                 CronSpec: &databasepb.CrontabSpec{
                     Text: "30 12 * * *",
                 },
             },
         },
         RetentionDuration: durationpb.New(24 * time.Hour),
         EncryptionConfig: &databasepb.CreateBackupEncryptionConfig{
             EncryptionType: databasepb.CreateBackupEncryptionConfig_GOOGLE_DEFAULT_ENCRYPTION,
         },
         BackupTypeSpec: &databasepb.BackupSchedule_IncrementalBackupSpec{},
     },
 }

 res, err := client.CreateBackupSchedule(ctx, &req)
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Created incremental backup schedule: %s", res)
 return nil
}
```

### Java

To create a full backup schedule, see the following sample code:

``` java
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.protobuf.Duration;
import com.google.spanner.admin.database.v1.BackupSchedule;
import com.google.spanner.admin.database.v1.BackupScheduleSpec;
import com.google.spanner.admin.database.v1.CreateBackupEncryptionConfig;
import com.google.spanner.admin.database.v1.CreateBackupScheduleRequest;
import com.google.spanner.admin.database.v1.CrontabSpec;
import com.google.spanner.admin.database.v1.DatabaseName;
import com.google.spanner.admin.database.v1.FullBackupSpec;
import java.io.IOException;

class CreateFullBackupScheduleSample {

  static void createFullBackupSchedule() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String backupScheduleId = "my-backup-schedule";
    createFullBackupSchedule(projectId, instanceId, databaseId, backupScheduleId);
  }

  static void createFullBackupSchedule(
      String projectId, String instanceId, String databaseId, String backupScheduleId)
      throws IOException {
    final CreateBackupEncryptionConfig encryptionConfig =
        CreateBackupEncryptionConfig.newBuilder()
            .setEncryptionType(CreateBackupEncryptionConfig.EncryptionType.USE_DATABASE_ENCRYPTION)
            .build();
    final BackupSchedule backupSchedule =
        BackupSchedule.newBuilder()
            .setFullBackupSpec(FullBackupSpec.newBuilder().build())
            .setRetentionDuration(Duration.newBuilder().setSeconds(3600 * 24).build())
            .setSpec(
                BackupScheduleSpec.newBuilder()
                    .setCronSpec(CrontabSpec.newBuilder().setText("30 12 * * *").build())
                    .build())
            .setEncryptionConfig(encryptionConfig)
            .build();

    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      DatabaseName databaseName = DatabaseName.of(projectId, instanceId, databaseId);
      final BackupSchedule createdBackupSchedule =
          databaseAdminClient.createBackupSchedule(
              CreateBackupScheduleRequest.newBuilder()
                  .setParent(databaseName.toString())
                  .setBackupScheduleId(backupScheduleId)
                  .setBackupSchedule(backupSchedule)
                  .build());
      System.out.println(
          String.format(
              "Created backup schedule: %s\n%s",
              createdBackupSchedule.getName(), createdBackupSchedule.toString()));
    }
  }
}
```

To create an incremental backup schedule, see the following sample code:

``` java
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.protobuf.Duration;
import com.google.spanner.admin.database.v1.BackupSchedule;
import com.google.spanner.admin.database.v1.BackupScheduleSpec;
import com.google.spanner.admin.database.v1.CreateBackupEncryptionConfig;
import com.google.spanner.admin.database.v1.CreateBackupScheduleRequest;
import com.google.spanner.admin.database.v1.CrontabSpec;
import com.google.spanner.admin.database.v1.DatabaseName;
import com.google.spanner.admin.database.v1.IncrementalBackupSpec;
import java.io.IOException;

class CreateIncrementalBackupScheduleSample {

  static void createIncrementalBackupSchedule() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String backupScheduleId = "my-backup-schedule";
    createIncrementalBackupSchedule(projectId, instanceId, databaseId, backupScheduleId);
  }

  static void createIncrementalBackupSchedule(
      String projectId, String instanceId, String databaseId, String backupScheduleId)
      throws IOException {
    final CreateBackupEncryptionConfig encryptionConfig =
        CreateBackupEncryptionConfig.newBuilder()
            .setEncryptionType(
                CreateBackupEncryptionConfig.EncryptionType.GOOGLE_DEFAULT_ENCRYPTION)
            .build();
    final BackupSchedule backupSchedule =
        BackupSchedule.newBuilder()
            .setIncrementalBackupSpec(IncrementalBackupSpec.newBuilder().build())
            .setRetentionDuration(Duration.newBuilder().setSeconds(3600 * 24).build())
            .setSpec(
                BackupScheduleSpec.newBuilder()
                    .setCronSpec(CrontabSpec.newBuilder().setText("30 12 * * *").build())
                    .build())
            .setEncryptionConfig(encryptionConfig)
            .build();

    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      DatabaseName databaseName = DatabaseName.of(projectId, instanceId, databaseId);
      final BackupSchedule createdBackupSchedule =
          databaseAdminClient.createBackupSchedule(
              CreateBackupScheduleRequest.newBuilder()
                  .setParent(databaseName.toString())
                  .setBackupScheduleId(backupScheduleId)
                  .setBackupSchedule(backupSchedule)
                  .build());
      System.out.println(
          String.format(
              "Created incremental backup schedule: %s\n%s",
              createdBackupSchedule.getName(), createdBackupSchedule.toString()));
    }
  }
}
```

### Node.js

To create a full backup schedule, see the following sample code:

``` javascript
// Import the Google Cloud client library for Spanner.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const scheduleId = 'my-schedule-id';

// Create a Spanner database admin client.
const spanner = new Spanner({projectId});
const client = spanner.getDatabaseAdminClient();

try {
  // Create a schedule to create full backups daily at 12:30 AM, using the
  // database's encryption config, and retained for 24 hours.
  const [response] = await client.createBackupSchedule({
    parent: client.databasePath(projectId, instanceId, databaseId),
    backupScheduleId: scheduleId,
    backupSchedule: {
      spec: {
        cronSpec: {
          text: '30 12 * * *',
        },
      },
      retentionDuration: {
        seconds: 86400,
      },
      encryptionConfig: {
        encryptionType: 'USE_DATABASE_ENCRYPTION',
      },
      fullBackupSpec: {},
    },
  });
  console.log('Created full backup schedule:', response);
} catch (err) {
  console.error('ERROR:', err);
}
```

To create an incremental backup schedule, see the following sample code:

``` javascript
// Import the Google Cloud client library for Spanner.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const scheduleId = 'my-schedule-id';

// Create a Spanner database admin client.
const spanner = new Spanner({projectId});
const client = spanner.getDatabaseAdminClient();

try {
  // Create a schedule to create incremental backups daily at 12:30 AM,
  // using Google-managed encryption, and retained for 24 hours.
  const [response] = await client.createBackupSchedule({
    parent: client.databasePath(projectId, instanceId, databaseId),
    backupScheduleId: scheduleId,
    backupSchedule: {
      spec: {
        cronSpec: {
          text: '30 12 * * *',
        },
      },
      retentionDuration: {
        seconds: 86400,
      },
      encryptionConfig: {
        encryptionType: 'GOOGLE_DEFAULT_ENCRYPTION',
      },
      incrementalBackupSpec: {},
    },
  });
  console.log('Created incremental backup schedule:', response);
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

To create a full backup schedule, see the following sample code:

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupScheduleRequest;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupEncryptionConfig;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupEncryptionConfig\EncryptionType;
use Google\Cloud\Spanner\Admin\Database\V1\BackupSchedule;
use Google\Cloud\Spanner\Admin\Database\V1\FullBackupSpec;
use Google\Cloud\Spanner\Admin\Database\V1\BackupScheduleSpec;
use Google\Cloud\Spanner\Admin\Database\V1\CrontabSpec;
use Google\Protobuf\Duration;

/**
 * Create a backup schedule.
 * Example:
 * ```
 * create_backup_schedule($projectId, $instanceId, $databaseId, $backupScheduleId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupScheduleId The ID of the backup schedule to be created.
 * at https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule.FIELDS
 */
function create_backup_schedule(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupScheduleId,
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseFullName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    printf('%s', $databaseFullName);

    $encryptionConfig = (new CreateBackupEncryptionConfig())
        ->setEncryptionType(EncryptionType::USE_DATABASE_ENCRYPTION);
    $backupSchedule = new BackupSchedule([
        'full_backup_spec' => new FullBackupSpec(),
        'retention_duration' => (new Duration())
            ->setSeconds(24 * 60 * 60),
        'spec' => new BackupScheduleSpec([
            'cron_spec' => new CrontabSpec([
                'text' => '30 12 * * *'
            ]),
        ]),
        'encryption_config' => $encryptionConfig,
    ]);
    $request = new CreateBackupScheduleRequest([
        'parent' => $databaseFullName,
        'backup_schedule_id' => $backupScheduleId,
        'backup_schedule' => $backupSchedule,
    ]);

    $created_backup_schedule = $databaseAdminClient->createBackupSchedule($request);

    printf('Created backup scehedule %s' . PHP_EOL, $created_backup_schedule->getName());
}
````

### Python

To create a full backup schedule, see the following sample code:

``` python
def create_full_backup_schedule(
    instance_id: str,
    database_id: str,
    schedule_id: str,
) -> None:
    from datetime import timedelta
    from google.cloud import spanner
    from google.cloud.spanner_admin_database_v1.types import (
        backup_schedule as backup_schedule_pb,
    )
    from google.cloud.spanner_admin_database_v1.types import (
        CreateBackupEncryptionConfig,
        FullBackupSpec,
    )

    client = spanner.Client()
    database_admin_api = client.database_admin_api

    request = backup_schedule_pb.CreateBackupScheduleRequest(
        parent=database_admin_api.database_path(
            client.project, instance_id, database_id
        ),
        backup_schedule_id=schedule_id,
        backup_schedule=backup_schedule_pb.BackupSchedule(
            spec=backup_schedule_pb.BackupScheduleSpec(
                cron_spec=backup_schedule_pb.CrontabSpec(
                    text="30 12 * * *",
                ),
            ),
            retention_duration=timedelta(hours=24),
            encryption_config=CreateBackupEncryptionConfig(
                encryption_type=CreateBackupEncryptionConfig.EncryptionType.USE_DATABASE_ENCRYPTION,
            ),
            full_backup_spec=FullBackupSpec(),
        ),
    )

    response = database_admin_api.create_backup_schedule(request)
    print(f"Created full backup schedule: {response}")
```

To create an incremental backup schedule, see the following sample code:

``` python
def create_incremental_backup_schedule(
    instance_id: str,
    database_id: str,
    schedule_id: str,
) -> None:
    from datetime import timedelta
    from google.cloud import spanner
    from google.cloud.spanner_admin_database_v1.types import (
        backup_schedule as backup_schedule_pb,
    )
    from google.cloud.spanner_admin_database_v1.types import (
        CreateBackupEncryptionConfig,
        IncrementalBackupSpec,
    )

    client = spanner.Client()
    database_admin_api = client.database_admin_api

    request = backup_schedule_pb.CreateBackupScheduleRequest(
        parent=database_admin_api.database_path(
            client.project, instance_id, database_id
        ),
        backup_schedule_id=schedule_id,
        backup_schedule=backup_schedule_pb.BackupSchedule(
            spec=backup_schedule_pb.BackupScheduleSpec(
                cron_spec=backup_schedule_pb.CrontabSpec(
                    text="30 12 * * *",
                ),
            ),
            retention_duration=timedelta(hours=24),
            encryption_config=CreateBackupEncryptionConfig(
                encryption_type=CreateBackupEncryptionConfig.EncryptionType.GOOGLE_DEFAULT_ENCRYPTION,
            ),
            incremental_backup_spec=IncrementalBackupSpec(),
        ),
    )

    response = database_admin_api.create_backup_schedule(request)
    print(f"Created incremental backup schedule: {response}")
```

## Get a backup schedule

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

<!-- end list -->

1.  Click the instance containing the database.

2.  Click the database.

3.  In the navigation menu, click the **Backup/Restore** .

4.  Click the **Schedules** tab to view all the backup schedules and their information.

### gcloud

Before using any of the command data below, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.
  - SCHEDULE\_ID : the backup schedule ID.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules describe SCHEDULE_ID \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules describe SCHEDULE_ID `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules describe SCHEDULE_ID ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID
```

You should receive a response similar to the following:

``` text
{
  encryptionConfig:
    encryptionType: USE_DATABASE_ENCRYPTION
  incrementalBackupSpec: {}
  name: projects/my-project/instances/my-instance/databases/my-database/backupSchedules/my-schedule
  retentionDuration: 2592000s
  spec:
    cronSpec:
      creationWindow: 14400s
      text: 0 */4 * * *
      timeZone: UTC
  updateTime: '2024-09-13T10:24:18.754839Z'
}
```

### REST v1

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.
  - SCHEDULE\_ID : the backup schedule ID.

HTTP method and URL:

``` text
GET https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
curl -X GET \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method GET `
    -Headers $headers `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
  "name": "projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID",
  "retentionDuration": "100000s",
  "encryptionConfig": {
    "encryptionType": "USE_DATABASE_ENCRYPTION"
  },
  "spec": {
    "cronSpec": {
      "text": "0 2 * * *",
      "timeZone": "UTC",
      "creationWindow": "14400s"
    }
  },
  "fullBackupSpec": {},
  "updateTime": "2024-05-22T11:13:51.835590Z"
}
```

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using System;

public class GetBackupScheduleSample
{
    public BackupSchedule GetBackupSchedule(string projectId, string instanceId, string databaseId, string scheduleId)
    {
        DatabaseAdminClient client = DatabaseAdminClient.Create();

        BackupSchedule response = client.GetBackupSchedule(BackupScheduleName.FromProjectInstanceDatabaseSchedule(projectId, instanceId, databaseId, scheduleId));

        Console.WriteLine($"Backup schedule: {response}");
        return response;
    }
}
```

### C++

``` cpp
void GetBackupSchedule(google::cloud::spanner_admin::DatabaseAdminClient client,
                       std::string const& project_id,
                       std::string const& instance_id,
                       std::string const& database_id,
                       std::string const& backup_schedule_id) {
  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  std::string backup_schedule_name =
      db.FullName() + "/backupSchedules/" + backup_schedule_id;

  google::spanner::admin::database::v1::GetBackupScheduleRequest request;
  request.set_name(backup_schedule_name);

  auto backup_schedule = client.GetBackupSchedule(request);
  if (!backup_schedule) throw std::move(backup_schedule).status();
  std::cout << "Retrieved backup schedule:\n" << backup_schedule->DebugString();
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func getBackupSchedule(w io.Writer, dbName string, scheduleId string) error {
 ctx := context.Background()

 client, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 req := databasepb.GetBackupScheduleRequest{
     Name: fmt.Sprintf("%s/backupSchedules/%s", dbName, scheduleId),
 }

 res, err := client.GetBackupSchedule(ctx, &req)
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Backup schedule: %s", res)
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.spanner.admin.database.v1.BackupSchedule;
import com.google.spanner.admin.database.v1.BackupScheduleName;
import com.google.spanner.admin.database.v1.GetBackupScheduleRequest;
import java.io.IOException;

class GetBackupScheduleSample {

  static void getBackupSchedule() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String backupScheduleId = "my-backup-schedule";
    getBackupSchedule(projectId, instanceId, databaseId, backupScheduleId);
  }

  static void getBackupSchedule(
      String projectId, String instanceId, String databaseId, String backupScheduleId)
      throws IOException {
    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      BackupScheduleName backupScheduleName =
          BackupScheduleName.of(projectId, instanceId, databaseId, backupScheduleId);
      final BackupSchedule backupSchedule =
          databaseAdminClient.getBackupSchedule(
              GetBackupScheduleRequest.newBuilder().setName(backupScheduleName.toString()).build());
      System.out.println(
          String.format(
              "Backup schedule: %s\n%s", backupSchedule.getName(), backupSchedule.toString()));
    }
  }
}
```

### Node.js

``` javascript
// Import the Google Cloud client library for Spanner.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const scheduleId = 'my-schedule-id';

// Create a Spanner database admin client.
const spanner = new Spanner({projectId});
const client = spanner.getDatabaseAdminClient();

try {
  // Get the backup schedule.
  const [response] = await client.getBackupSchedule({
    name: client.backupSchedulePath(
      projectId,
      instanceId,
      databaseId,
      scheduleId,
    ),
  });
  console.log('Backup schedule:', response);
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\GetBackupScheduleRequest;

/**
 * Get a backup schedule.
 * Example:
 * ```
 * get_backup_schedule($projectId, $instanceId, $databaseId, $backupScheduleId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupScheduleId The ID of the backup schedule to be created.
 * at https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule.FIELDS
 */
function get_backup_schedule(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupScheduleId,
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $backupScheduleName = sprintf(
        'projects/%s/instances/%s/databases/%s/backupSchedules/%s',
        $projectId,
        $instanceId,
        $databaseId,
        $backupScheduleId
    );
    $request = new GetBackupScheduleRequest([
        'name' => $backupScheduleName,
    ]);

    $backup_schedule = $databaseAdminClient->getBackupSchedule($request);

    printf('Fetched backup scehedule %s' . PHP_EOL, $backup_schedule->getName());
}
````

### Python

``` python
def get_backup_schedule(
    instance_id: str,
    database_id: str,
    schedule_id: str,
) -> None:
    from google.cloud import spanner
    from google.cloud.spanner_admin_database_v1.types import (
        backup_schedule as backup_schedule_pb,
    )

    client = spanner.Client()
    database_admin_api = client.database_admin_api

    request = backup_schedule_pb.GetBackupScheduleRequest(
        name=database_admin_api.backup_schedule_path(
            client.project,
            instance_id,
            database_id,
            schedule_id,
        ),
    )

    response = database_admin_api.get_backup_schedule(request)
    print(f"Backup schedule: {response}")
```

## List backup schedules

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

<!-- end list -->

1.  Click the instance containing the database.

2.  Click the database.

3.  In the navigation menu, click the **Backup/Restore** .

4.  Click the **Schedules** tab to view all the backup schedules and their information.

### gcloud

Before using any of the command data below, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules list \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules list `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules list ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID
```

You should receive a response similar to the following:

``` text
  
    Name
```

Backup type

Cron

Retention duration

Encryption type

my-full-schedule

FULL

30 12 \* \* \*

2592000s

USE\_DATABASE\_ENCRYPTION

my-incr-schedule

INCREMENTAL

0 \*/4 \* \* \*

2592000s

USE\_DATABASE\_ENCRYPTION

### REST v1

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.

HTTP method and URL:

``` text
GET https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
curl -X GET \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method GET `
    -Headers $headers `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
  "backupSchedules": [
    {
      "name": "projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID",
      "retentionDuration": "172800s",
      "encryptionConfig": {
        "encryptionType": "USE_DATABASE_ENCRYPTION"
      },
      "spec": {
        "cronSpec": {
          "text": "0 */12 * * *",
          "timeZone": "UTC",
          "creationWindow": "14400s"
        }
      },
      "fullBackupSpec": {},
      "updateTime": "2024-09-09T07:21:43.946180Z"
    },
    {
      "name": "projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID",
      "retentionDuration": "2592000s",
      "encryptionConfig": {
        "encryptionType": "CUSTOMER_MANAGED_ENCRYPTION",
        "kmsKeyName": "projects/PROJECT_ID/locations/us-central1/keyRings/cmek-demo/cryptoKeys/test-key"
      },
      "spec": {
        "cronSpec": {
          "text": "30 12 * * *",
          "timeZone": "UTC",
          "creationWindow": "14400s"
        }
      },
      "fullBackupSpec": {},
      "updateTime": "2024-09-17T18:27:53.868741Z"
    }
  ]
}
```

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Cloud.Spanner.Common.V1;
using System;
using System.Collections.Generic;

public class ListBackupSchedulesSample
{
    public IEnumerable<BackupSchedule> ListBackupSchedules(string projectId, string instanceId, string databaseId)
    {
        DatabaseAdminClient client = DatabaseAdminClient.Create();

        var backupSchedules = client.ListBackupSchedules(
            new ListBackupSchedulesRequest
            {
                ParentAsDatabaseName = DatabaseName.FromProjectInstanceDatabase(projectId, instanceId, databaseId),
            });

        foreach (BackupSchedule backupSchedule in backupSchedules)
        {
            Console.WriteLine($"Backup schedule: {backupSchedule}");
        }
        return backupSchedules;
    }
}
```

### C++

``` cpp
void ListBackupSchedules(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id) {
  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  google::spanner::admin::database::v1::ListBackupSchedulesRequest request;
  request.set_parent(db.FullName());

  int count = 0;
  for (auto& backup_schedule : client.ListBackupSchedules(request)) {
    if (!backup_schedule) throw std::move(backup_schedule).status();

    std::cout << "Backup Schedule [" << ++count << "]:\n"
              << backup_schedule->DebugString();
  }
  if (count == 0) {
    std::cout << "No backup schedules found for database " << db.FullName();
  }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/api/iterator"
)

func listBackupSchedules(w io.Writer, dbName string) error {
 ctx := context.Background()

 client, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 req := databasepb.ListBackupSchedulesRequest{
     Parent: dbName,
 }
 iter := client.ListBackupSchedules(ctx, &req)

 for {
     backupSchedule, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "Backup schedule: %s\n", backupSchedule)
 }

 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.spanner.admin.database.v1.BackupSchedule;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.io.IOException;

class ListBackupSchedulesSample {

  static void listBackupSchedules() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    listBackupSchedules(projectId, instanceId, databaseId);
  }

  static void listBackupSchedules(String projectId, String instanceId, String databaseId)
      throws IOException {
    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      DatabaseName databaseName = DatabaseName.of(projectId, instanceId, databaseId);

      System.out.println(
          String.format("Backup schedules for database '%s'", databaseName.toString()));
      for (BackupSchedule backupSchedule :
          databaseAdminClient.listBackupSchedules(databaseName).iterateAll()) {
        System.out.println(
            String.format(
                "Backup schedule: %s\n%s", backupSchedule.getName(), backupSchedule.toString()));
      }
    }
  }
}
```

### Node.js

``` javascript
// Import the Google Cloud client library for Spanner.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';

// Create a Spanner database admin client.
const spanner = new Spanner({projectId});
const client = spanner.getDatabaseAdminClient();

try {
  // List backup schedules of a database.
  const [schedules] = await client.listBackupSchedules({
    parent: client.databasePath(projectId, instanceId, databaseId),
  });
  schedules.forEach(schedule => {
    console.log('Backup schedule:', schedule);
  });
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\ListBackupSchedulesRequest;

/**
 * Get list of all backup schedules for a given database.
 * Example:
 * ```
 * list_backup_schedules($projectId, $instanceId, $databaseId, $backupScheduleId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * at https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule.FIELDS
 */
function list_backup_schedules(
    string $projectId,
    string $instanceId,
    string $databaseId,
): void {
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseFullName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new ListBackupSchedulesRequest([
        'parent' => $databaseFullName,
    ]);
    $backup_schedules = $databaseAdminClient->listBackupSchedules($request);

    printf('Backup schedules for database %s' . PHP_EOL, $databaseFullName);
    foreach ($backup_schedules as $schedule) {
        printf('Backup schedule: %s' . PHP_EOL, $schedule->getName());
    }
}
````

### Python

``` python
def list_backup_schedules(instance_id: str, database_id: str) -> None:
    from google.cloud import spanner
    from google.cloud.spanner_admin_database_v1.types import (
        backup_schedule as backup_schedule_pb,
    )

    client = spanner.Client()
    database_admin_api = client.database_admin_api

    request = backup_schedule_pb.ListBackupSchedulesRequest(
        parent=database_admin_api.database_path(
            client.project,
            instance_id,
            database_id,
        ),
    )

    for backup_schedule in database_admin_api.list_backup_schedules(request):
        print(f"Backup schedule: {backup_schedule}")
```

## Update a backup schedule

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

<!-- end list -->

1.  Click the instance containing the database.

2.  Click the database.

3.  In the navigation menu, click the **Backup/Restore** .

4.  Click the **Schedules** tab.

5.  Select **More actions** for your backup schedule and click **Update** .

6.  You can update the frequency and retention time of the backup schedule. If it's a full backup schedule, you can also update the encryption type.

7.  Click **Save** .

### gcloud

Before using any of the command data below, make the following replacements:

  - SCHEDULE\_ID : the backup schedule ID.
  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the ID of the instance where you want to modify the backup schedule.
  - DATABASE\_ID : the ID of the database where you want to modify the backup schedule.
  - RETENTION\_DURATION : the retention duration of the backups created by the schedule.
  - CRONTAB\_EXPRESSION : the crontab expression for the backup schedule frequency. For example, if you want the backup schedule frequency to be every 12 hours, you can use `  0 12 * * *  ` .
  - ENCRYPTION\_TYPE : the encryption type of backups created by the backup schedule. Valid values are `  USE_DATABASE_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` or `  kmsKeyNames  ` .

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules update SCHEDULE_ID \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID \
  --retention-duration=RETENTION_DURATION \
  --cron="CRONTAB_EXPRESSION" \
  --encryption-type=ENCRYPTION_TYPE
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules update SCHEDULE_ID `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID `
  --retention-duration=RETENTION_DURATION `
  --cron="CRONTAB_EXPRESSION" `
  --encryption-type=ENCRYPTION_TYPE
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules update SCHEDULE_ID ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID ^
  --retention-duration=RETENTION_DURATION ^
  --cron="CRONTAB_EXPRESSION" ^
  --encryption-type=ENCRYPTION_TYPE
```

### REST v1

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the ID of the instance where you want to modify the backup schedule.
  - DATABASE\_ID : the ID of the database where you want to modify the backup schedule.
  - SCHEDULE\_ID : the backup schedule ID.
  - RETENTION\_DURATION : the retention duration of the backups created by the schedule. For example, if you want the retention duration to be one day, specify `  86400s  ` .
  - ENCRYPTION\_TYPE : the encryption type of backups created by the backup schedule. Possible values are `  USE_DATABASE_ENCRYPTION  ` , `  GOOGLE_DEFAULT_ENCRYPTION  ` , or `  CUSTOMER_MANAGED_ENCRYPTION  ` . If you use `  CUSTOMER_MANAGED_ENCRYPTION  ` , you must specify a `  kmsKeyName  ` or `  kmsKeyNames  ` .

HTTP method and URL:

``` text
PATCH https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID?updateMask=retention_duration,encryption_config,spec.cron_spec.text
```

Request JSON body:

``` text
{
  "retentionDuration": "RETENTION_DURATION",
  "spec": {
    "cronSpec": {
      "text": "0 2 * * *"
    }
    "encryptionConfig": {
      "encryptionType": "ENCRYPTION_TYPE"
    },
  },
}
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Save the request body in a file named `  request.json  ` , and execute the following command:

``` text
curl -X PATCH \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "Content-Type: application/json; charset=utf-8" \
     -d @request.json \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID?updateMask=retention_duration,encryption_config,spec.cron_spec.text"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Save the request body in a file named `  request.json  ` , and execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method PATCH `
    -Headers $headers `
    -ContentType: "application/json; charset=utf-8" `
    -InFile request.json `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID?updateMask=retention_duration,encryption_config,spec.cron_spec.text" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
  {
    "name": "projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID",
    "retentionDuration": "86400s",
    "encryptionConfig": {
      "encryptionType": "USE_DATABASE_ENCRYPTION"
    },
    "spec": {
      "cronSpec": {
        "text": "0 2 * * *",
        "timeZone": "UTC",
        "creationWindow": "14400s"
      }
    },
    "fullBackupSpec": {},
    "updateTime": "2024-05-22T11:13:51.835590Z"
  }
```

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using Google.Protobuf.WellKnownTypes;
using System;

public class UpdateBackupScheduleSample
{
    public BackupSchedule UpdateBackupSchedule(string projectId, string instanceId, string databaseId, string scheduleId)
    {
        DatabaseAdminClient client = DatabaseAdminClient.Create();

        BackupSchedule response = client.UpdateBackupSchedule(
            new UpdateBackupScheduleRequest
            {
                BackupSchedule = new BackupSchedule
                {
                    BackupScheduleName = BackupScheduleName.FromProjectInstanceDatabaseSchedule(projectId, instanceId, databaseId, scheduleId),
                    Spec = new BackupScheduleSpec
                    {
                        CronSpec = new CrontabSpec
                        {
                            Text = "45 15 * * *",
                        },
                    },
                    RetentionDuration = new Duration
                    {
                        Seconds = 172800,
                    },
                    EncryptionConfig = new CreateBackupEncryptionConfig
                    {
                        EncryptionType = CreateBackupEncryptionConfig.Types.EncryptionType.UseDatabaseEncryption,
                    },
                },
                UpdateMask = new FieldMask
                {
                    Paths = { "spec.cron_spec.text", "retention_duration", "encryption_config" },
                },
            });

        Console.WriteLine($"Updated backup schedule: {response}");
        return response;
    }
}
```

### C++

``` cpp
void UpdateBackupSchedule(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& backup_schedule_id) {
  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  std::string backup_schedule_name =
      db.FullName() + "/backupSchedules/" + backup_schedule_id;

  google::spanner::admin::database::v1::BackupSchedule backup_schedule;
  backup_schedule.set_name(backup_schedule_name);
  backup_schedule.mutable_spec()->mutable_cron_spec()->set_text("45 15 * * *");
  backup_schedule.mutable_retention_duration()->set_seconds(3600 * 48);

  google::spanner::admin::database::v1::CreateBackupEncryptionConfig
      encryption_config;
  encryption_config.set_encryption_type(
      google::spanner::admin::database::v1::CreateBackupEncryptionConfig::
          USE_DATABASE_ENCRYPTION);
  *backup_schedule.mutable_encryption_config() = std::move(encryption_config);

  google::protobuf::FieldMask update_mask;
  update_mask.add_paths("spec.cron_spec.text");
  update_mask.add_paths("retention_duration");
  update_mask.add_paths("encryption_config");

  google::spanner::admin::database::v1::UpdateBackupScheduleRequest request;
  *request.mutable_backup_schedule() = std::move(backup_schedule);
  *request.mutable_update_mask() = std::move(update_mask);

  auto updated_backup_schedule = client.UpdateBackupSchedule(request);
  if (!updated_backup_schedule) {
    throw std::move(updated_backup_schedule).status();
  }
  std::cout << "Backup schedule " << updated_backup_schedule->name()
            << " updated at "
            << *google::cloud::spanner::MakeTimestamp(
                   updated_backup_schedule->update_time());
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "time"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
 "google.golang.org/protobuf/types/known/durationpb"
 "google.golang.org/protobuf/types/known/fieldmaskpb"
)

func updateBackupSchedule(w io.Writer, dbName string, scheduleId string) error {
 ctx := context.Background()

 client, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 // Update a schedule to create backups daily at 3:45 PM, using the database's
 // encryption config, and retained for 48 hours.
 req := databasepb.UpdateBackupScheduleRequest{
     BackupSchedule: &databasepb.BackupSchedule{
         Name: fmt.Sprintf("%s/backupSchedules/%s", dbName, scheduleId),
         Spec: &databasepb.BackupScheduleSpec{
             ScheduleSpec: &databasepb.BackupScheduleSpec_CronSpec{
                 CronSpec: &databasepb.CrontabSpec{
                     Text: "45 15 * * *",
                 },
             },
         },
         RetentionDuration: durationpb.New(48 * time.Hour),
         EncryptionConfig: &databasepb.CreateBackupEncryptionConfig{
             EncryptionType: databasepb.CreateBackupEncryptionConfig_USE_DATABASE_ENCRYPTION,
         },
     },
     UpdateMask: &fieldmaskpb.FieldMask{
         Paths: []string{
             "spec.cron_spec.text",
             "retention_duration",
             "encryption_config",
         },
     },
 }

 res, err := client.UpdateBackupSchedule(ctx, &req)
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Updated backup schedule: %s", res)
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.protobuf.Duration;
import com.google.protobuf.FieldMask;
import com.google.spanner.admin.database.v1.BackupSchedule;
import com.google.spanner.admin.database.v1.BackupScheduleName;
import com.google.spanner.admin.database.v1.BackupScheduleSpec;
import com.google.spanner.admin.database.v1.CreateBackupEncryptionConfig;
import com.google.spanner.admin.database.v1.CrontabSpec;
import com.google.spanner.admin.database.v1.UpdateBackupScheduleRequest;
import java.io.IOException;

class UpdateBackupScheduleSample {

  static void updateBackupSchedule() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String backupScheduleId = "my-backup-schedule";
    updateBackupSchedule(projectId, instanceId, databaseId, backupScheduleId);
  }

  static void updateBackupSchedule(
      String projectId, String instanceId, String databaseId, String backupScheduleId)
      throws IOException {
    BackupScheduleName backupScheduleName =
        BackupScheduleName.of(projectId, instanceId, databaseId, backupScheduleId);
    final CreateBackupEncryptionConfig encryptionConfig =
        CreateBackupEncryptionConfig.newBuilder()
            .setEncryptionType(CreateBackupEncryptionConfig.EncryptionType.USE_DATABASE_ENCRYPTION)
            .build();
    final BackupSchedule backupSchedule =
        BackupSchedule.newBuilder()
            .setName(backupScheduleName.toString())
            .setRetentionDuration(Duration.newBuilder().setSeconds(3600 * 48))
            .setSpec(
                BackupScheduleSpec.newBuilder()
                    .setCronSpec(CrontabSpec.newBuilder().setText("45 15 * * *").build())
                    .build())
            .setEncryptionConfig(encryptionConfig)
            .build();

    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      final FieldMask fieldMask =
          FieldMask.newBuilder()
              .addPaths("retention_duration")
              .addPaths("spec.cron_spec.text")
              .addPaths("encryption_config")
              .build();
      final BackupSchedule updatedBackupSchedule =
          databaseAdminClient.updateBackupSchedule(
              UpdateBackupScheduleRequest.newBuilder()
                  .setBackupSchedule(backupSchedule)
                  .setUpdateMask(fieldMask)
                  .build());
      System.out.println(
          String.format(
              "Updated backup schedule: %s\n%s",
              updatedBackupSchedule.getName(), updatedBackupSchedule.toString()));
    }
  }
}
```

### Node.js

``` javascript
// Import the Google Cloud client library for Spanner.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const scheduleId = 'my-schedule-id';

// Create a Spanner database admin client.
const spanner = new Spanner({projectId});
const client = spanner.getDatabaseAdminClient();

try {
  // Update the schedule to create backups daily at 3:45 PM, using the
  // database's encryption config, and retained for 48 hours.
  const [response] = await client.updateBackupSchedule({
    backupSchedule: {
      name: client.backupSchedulePath(
        projectId,
        instanceId,
        databaseId,
        scheduleId,
      ),
      spec: {
        cronSpec: {
          text: '45 15 * * *',
        },
      },
      retentionDuration: {
        seconds: 172800,
      },
      encryptionConfig: {
        encryptionType: 'USE_DATABASE_ENCRYPTION',
      },
    },
    updateMask: {
      paths: [
        'spec.cron_spec.text',
        'retention_duration',
        'encryption_config',
      ],
    },
  });
  console.log('Updated backup schedule:', response);
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateBackupScheduleRequest;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupEncryptionConfig;
use Google\Cloud\Spanner\Admin\Database\V1\CreateBackupEncryptionConfig\EncryptionType;
use Google\Cloud\Spanner\Admin\Database\V1\BackupSchedule;
use Google\Cloud\Spanner\Admin\Database\V1\FullBackupSpec;
use Google\Cloud\Spanner\Admin\Database\V1\BackupScheduleSpec;
use Google\Cloud\Spanner\Admin\Database\V1\CrontabSpec;
use Google\Protobuf\Duration;
use Google\Protobuf\FieldMask;

/**
 * Update an existing backup schedule.
 * Example:
 * ```
 * update_backup_schedule($projectId, $instanceId, $databaseId, $backupScheduleId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupScheduleId The ID of the backup schedule to be created.
 * at https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule.FIELDS
 */
function update_backup_schedule(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupScheduleId,
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $encryptionConfig = new CreateBackupEncryptionConfig([
        'encryption_type' => EncryptionType::USE_DATABASE_ENCRYPTION,
    ]);
    $backupScheduleName = sprintf(
        'projects/%s/instances/%s/databases/%s/backupSchedules/%s',
        $projectId,
        $instanceId,
        $databaseId,
        $backupScheduleId
    );
    $backupSchedule = new BackupSchedule([
        'name' => $backupScheduleName,
        'full_backup_spec' => new FullBackupSpec(),
        'retention_duration' => (new Duration())
            ->setSeconds(48 * 60 * 60),
        'spec' => new BackupScheduleSpec([
            'cron_spec' => new CrontabSpec([
                'text' => '45 15 * * *'
            ]),
        ]),
        'encryption_config' => $encryptionConfig,
    ]);
    $fieldMask = (new FieldMask())
        ->setPaths([
            'retention_duration',
            'spec.cron_spec.text',
            'encryption_config',
        ]);

    $request = new UpdateBackupScheduleRequest([
        'backup_schedule' => $backupSchedule,
        'update_mask' => $fieldMask,
    ]);

    $updated_backup_schedule = $databaseAdminClient->updateBackupSchedule($request);

    printf('Updated backup scehedule %s' . PHP_EOL, $updated_backup_schedule->getName());
}
````

### Python

``` python
def update_backup_schedule(
    instance_id: str,
    database_id: str,
    schedule_id: str,
) -> None:
    from datetime import timedelta
    from google.cloud import spanner
    from google.cloud.spanner_admin_database_v1.types import (
        backup_schedule as backup_schedule_pb,
    )
    from google.cloud.spanner_admin_database_v1.types import (
        CreateBackupEncryptionConfig,
    )
    from google.protobuf.field_mask_pb2 import FieldMask

    client = spanner.Client()
    database_admin_api = client.database_admin_api

    request = backup_schedule_pb.UpdateBackupScheduleRequest(
        backup_schedule=backup_schedule_pb.BackupSchedule(
            name=database_admin_api.backup_schedule_path(
                client.project,
                instance_id,
                database_id,
                schedule_id,
            ),
            spec=backup_schedule_pb.BackupScheduleSpec(
                cron_spec=backup_schedule_pb.CrontabSpec(
                    text="45 15 * * *",
                ),
            ),
            retention_duration=timedelta(hours=48),
            encryption_config=CreateBackupEncryptionConfig(
                encryption_type=CreateBackupEncryptionConfig.EncryptionType.USE_DATABASE_ENCRYPTION,
            ),
        ),
        update_mask=FieldMask(
            paths=[
                "spec.cron_spec.text",
                "retention_duration",
                "encryption_config",
            ],
        ),
    )

    response = database_admin_api.update_backup_schedule(request)
    print(f"Updated backup schedule: {response}")
```

## Delete a backup schedule

### Console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

<!-- end list -->

1.  Click the instance containing the database.

2.  Click the database.

3.  In the navigation menu, click the **Backup/Restore** .

4.  Select **More actions** for your backup schedule and click **Delete** .

5.  Type the backup schedule name.

6.  Click **Confirm** .

### gcloud

Before using any of the command data below, make the following replacements:

  - SCHEDULE\_ID : the backup schedule ID.
  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the ID of the instance where you want to delete the backup schedule.
  - DATABASE\_ID : the ID of the database where you want to delete the backup schedule.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules delete SCHEDULE_ID \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules delete SCHEDULE_ID `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules delete SCHEDULE_ID ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID
```

### REST v1

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the ID of the instance where you want to delete the backup schedule.
  - DATABASE\_ID : the ID of the database where you want to delete the backup schedule.
  - SCHEDULE\_ID : the backup schedule ID.

HTTP method and URL:

``` text
DELETE https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
curl -X DELETE \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method DELETE `
    -Headers $headers `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID" | Select-Object -Expand Content
```

You should receive a successful status code (2xx) and an empty response.

### Client libraries

### C\#

``` csharp
using Google.Cloud.Spanner.Admin.Database.V1;
using System;

public class DeleteBackupScheduleSample
{
    public void DeleteBackupSchedule(string projectId, string instanceId, string databaseId, string scheduleId)
    {
        DatabaseAdminClient client = DatabaseAdminClient.Create();

        client.DeleteBackupSchedule(BackupScheduleName.FromProjectInstanceDatabaseSchedule(projectId, instanceId, databaseId, scheduleId));

        Console.WriteLine("Deleted backup schedule");
    }
}
```

### C++

``` cpp
void DeleteBackupSchedule(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& backup_schedule_id) {
  google::cloud::spanner::Database db(project_id, instance_id, database_id);
  std::string backup_schedule_name =
      db.FullName() + "/backupSchedules/" + backup_schedule_id;

  google::spanner::admin::database::v1::DeleteBackupScheduleRequest request;
  request.set_name(backup_schedule_name);

  auto status = client.DeleteBackupSchedule(request);
  if (!status.ok()) throw std::move(status);
  std::cout << "Backup schedule " << backup_schedule_name << " deleted";
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func deleteBackupSchedule(w io.Writer, dbName string, scheduleId string) error {
 ctx := context.Background()

 client, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer client.Close()

 req := databasepb.DeleteBackupScheduleRequest{
     Name: fmt.Sprintf("%s/backupSchedules/%s", dbName, scheduleId),
 }

 err = client.DeleteBackupSchedule(ctx, &req)
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Deleted backup schedule")
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.spanner.admin.database.v1.BackupScheduleName;
import com.google.spanner.admin.database.v1.DeleteBackupScheduleRequest;
import java.io.IOException;

class DeleteBackupScheduleSample {

  static void deleteBackupSchedule() throws IOException {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String backupScheduleId = "my-backup-schedule";
    deleteBackupSchedule(projectId, instanceId, databaseId, backupScheduleId);
  }

  static void deleteBackupSchedule(
      String projectId, String instanceId, String databaseId, String backupScheduleId)
      throws IOException {
    try (DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.create()) {
      BackupScheduleName backupScheduleName =
          BackupScheduleName.of(projectId, instanceId, databaseId, backupScheduleId);
      databaseAdminClient.deleteBackupSchedule(
          DeleteBackupScheduleRequest.newBuilder().setName(backupScheduleName.toString()).build());
      System.out.println(
          String.format("Deleted backup schedule: %s", backupScheduleName.toString()));
    }
  }
}
```

### Node.js

``` javascript
// Import the Google Cloud client library for Spanner.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance-id';
// const databaseId = 'my-database-id';
// const scheduleId = 'my-schedule-id';

// Create a Spanner database admin client.
const spanner = new Spanner({projectId});
const client = spanner.getDatabaseAdminClient();

try {
  // Delete the backup schedule.
  await client.deleteBackupSchedule({
    name: client.backupSchedulePath(
      projectId,
      instanceId,
      databaseId,
      scheduleId,
    ),
  });
  console.log('Deleted backup schedule');
} catch (err) {
  console.error('ERROR:', err);
}
```

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\DeleteBackupScheduleRequest;

/**
 * Delete a backup schedule.
 * Example:
 * ```
 * delete_backup_schedule($projectId, $instanceId, $databaseId, $backupScheduleId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $backupScheduleId The ID of the backup schedule to be created.
 * at https://cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules#BackupSchedule.FIELDS
 */
function delete_backup_schedule(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $backupScheduleId,
): void {
    $databaseAdminClient = new DatabaseAdminClient();

    $backupScheduleName = sprintf(
        'projects/%s/instances/%s/databases/%s/backupSchedules/%s',
        $projectId,
        $instanceId,
        $databaseId,
        $backupScheduleId
    );
    $request = new DeleteBackupScheduleRequest([
        'name' => strval($backupScheduleName),
    ]);

    $databaseAdminClient->deleteBackupSchedule($request);
    printf('Deleted backup scehedule %s' . PHP_EOL, $backupScheduleName);
}
````

### Python

``` python
def delete_backup_schedule(
    instance_id: str,
    database_id: str,
    schedule_id: str,
) -> None:
    from google.cloud import spanner
    from google.cloud.spanner_admin_database_v1.types import (
        backup_schedule as backup_schedule_pb,
    )

    client = spanner.Client()
    database_admin_api = client.database_admin_api

    request = backup_schedule_pb.DeleteBackupScheduleRequest(
        name=database_admin_api.backup_schedule_path(
            client.project,
            instance_id,
            database_id,
            schedule_id,
        ),
    )

    database_admin_api.delete_backup_schedule(request)
    print("Deleted backup schedule")
```

## Set IAM access control policy

You can set an Identity and Access Management (IAM) access policy to specify who has access to the backup schedule.

### gcloud

Before using any of the command data below, make the following replacements:

  - SCHEDULE\_ID : the backup schedule ID.
  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules set-iam-policy SCHEDULE_ID \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID \
  policy.json
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules set-iam-policy SCHEDULE_ID `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID `
  policy.json
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules set-iam-policy SCHEDULE_ID ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID ^
  policy.json
```

You should receive a response similar to the following:

``` text
Updated IAM policy for backupSchedule [SCHEDULE_ID].
bindings:
- members:
  - user:test@google.com
  role: roles/editor
etag: BwYi82k-fho=
version: 1
```

The following is an example `  policy.json  ` file that you can use along with this Google Cloud CLI command:

``` text
{
"version": 1,
"etag": "BwYi8ypICC0=",
"bindings": [
  {
    "role": "roles/editor",
    "members": [
      "user:test@gmail.com"
    ]
  }
]
}
```

### REST v1

Before using any of the request data, make the following replacements:

  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.
  - SCHEDULE\_ID : the backup schedule ID.

HTTP method and URL:

``` text
POST https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID:setIamPolicy?alt=json
```

Request JSON body:

``` text
{
  "policy": {
    "version": 1,
    "etag": "BwYi8ypICC0=",
    "bindings": [
      {
        "role": "roles/editor",
        "members": [
          "user:test@gmail.com"
        ]
      }
    ]
  }
}
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Save the request body in a file named `  request.json  ` , and execute the following command:

``` text
curl -X POST \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "Content-Type: application/json; charset=utf-8" \
     -d @request.json \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID:setIamPolicy?alt=json"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Save the request body in a file named `  request.json  ` , and execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method POST `
    -Headers $headers `
    -ContentType: "application/json; charset=utf-8" `
    -InFile request.json `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID:setIamPolicy?alt=json" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
  {
    "version": 1,
    "etag": "etag",
    "bindings": [
      {
        "role": "roles/non-primitive",
        "members": [
          "user:test@gmail.com"
        ]
      }
    ]
  }
}
```

## Get IAM access control policy

### gcloud

Before using any of the command data below, make the following replacements:

  - SCHEDULE\_ID : the backup schedule ID.
  - PROJECT\_ID : the project ID.
  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.

Execute the following command:

#### Linux, macOS, or Cloud Shell

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules get-iam-policy SCHEDULE_ID \
  --project=PROJECT_ID \
  --instance=INSTANCE_ID \
  --database=DATABASE_ID
```

#### Windows (PowerShell)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules get-iam-policy SCHEDULE_ID `
  --project=PROJECT_ID `
  --instance=INSTANCE_ID `
  --database=DATABASE_ID
```

#### Windows (cmd.exe)

**Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](/sdk/gcloud/reference/init) ; or [gcloud auth login](/sdk/gcloud/reference/auth/login) and [gcloud config set project](/sdk/gcloud/reference/config/set) .

``` text
gcloud spanner backup-schedules get-iam-policy SCHEDULE_ID ^
  --project=PROJECT_ID ^
  --instance=INSTANCE_ID ^
  --database=DATABASE_ID
```

You should receive a response similar to the following:

``` text
bindings:
- members:
  - user:test@gmail.com
  role: roles/editor
etag: BwYi82k-fho=
version: 1
```

### REST v1

Before using any of the request data, make the following replacements:

  - INSTANCE\_ID : the instance ID.
  - DATABASE\_ID : the database ID.
  - SCHEDULE\_ID : the backup schedule ID.
  - PROJECT\_ID : the project ID.

HTTP method and URL:

``` text
POST https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID:getIamPolicy?alt=json
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
curl -X POST \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "Content-Type: application/json; charset=utf-8" \
     -d "" \
     "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID:getIamPolicy?alt=json"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method POST `
    -Headers $headers `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID/backupSchedules/SCHEDULE_ID:getIamPolicy?alt=json" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
  {
    "version": 1,
    "etag": "BwYbyZ9pc4o=",
    "bindings": [
      {
        "role": "roles/editor",
        "members": [
          "user:test@gmail.com"
        ]
      }
    ]
  }
}
```

## What's next

  - [Backups overview](/spanner/docs/backup)
