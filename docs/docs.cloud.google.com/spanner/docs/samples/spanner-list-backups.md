---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-list-backups
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-list-backups
title: List backups
description: List all backups and access backup data using filters and paging.
data_source: docs.cloud.google.com
---

List all backups and access backup data using filters and paging.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Manage backups](https://docs.cloud.google.com/spanner/docs/backup/manage-backups)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void ListBackups(google::cloud::spanner_admin::DatabaseAdminClient client,
                     std::string const& project_id,
                     std::string const& instance_id) {
      google::cloud::spanner::Instance in(project_id, instance_id);
      std::cout << "All backups:\n";
      for (auto& backup : client.ListBackups(in.FullName())) {
        if (!backup) throw std::move(backup).status();
        std::cout << "Backup " << backup->name() << " on database "
                  << backup->database() << " with size : " << backup->size_bytes()
                  << " bytes.\n";
      }
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    using Google.Cloud.Spanner.Admin.Database.V1;
    using Google.Cloud.Spanner.Common.V1;
    using System;
    using System.Collections.Generic;
    using System.Linq;
    
    public class ListBackupsSample
    {
        public IEnumerable<Backup> ListBackups(string projectId, string instanceId, string databaseId, string backupId)
        {
            // Create the DatabaseAdminClient instance.
            DatabaseAdminClient databaseAdminClient = DatabaseAdminClient.Create();
    
            InstanceName parentAsInstanceName = InstanceName.FromProjectInstance(projectId, instanceId);
    
            // List all backups.
            Console.WriteLine("All backups:");
            var allBackups = databaseAdminClient.ListBackups(parentAsInstanceName);
            PrintBackups(allBackups);
    
            ListBackupsRequest request = new ListBackupsRequest
            {
                ParentAsInstanceName = parentAsInstanceName,
            };
    
            // List backups containing backup name.
            Console.WriteLine($"Backups with backup name containing {backupId}:");
            request.Filter = $"name:{backupId}";
            var backupsWithName = databaseAdminClient.ListBackups(request);
            PrintBackups(backupsWithName);
    
            // List backups on a database containing name.
            Console.WriteLine($"Backups with database name containing {databaseId}:");
            request.Filter = $"database:{databaseId}";
            var backupsWithDatabaseName = databaseAdminClient.ListBackups(request);
            PrintBackups(backupsWithDatabaseName);
    
            // List backups that expire within 30 days.
            Console.WriteLine("Backups expiring within 30 days:");
            string expireTime = DateTime.UtcNow.AddDays(30).ToString("O");
            request.Filter = $"expire_time < \"{expireTime}\"";
            var expiringBackups = databaseAdminClient.ListBackups(request);
            PrintBackups(expiringBackups);
    
            // List backups with a size greater than 100 bytes.
            Console.WriteLine("Backups with size > 100 bytes:");
            request.Filter = "size_bytes > 100";
            var backupsWithSize = databaseAdminClient.ListBackups(request);
            PrintBackups(backupsWithSize);
    
            // List backups created in the last day that are ready.
            Console.WriteLine("Backups created within last day that are ready:");
            string createTime = DateTime.UtcNow.AddDays(-1).ToString("O");
            request.Filter = $"create_time >= \"{createTime}\" AND state:READY";
            var recentReadyBackups = databaseAdminClient.ListBackups(request);
            PrintBackups(recentReadyBackups);
    
            // List backups in pages of 500 elements each
            foreach (var page in databaseAdminClient.ListBackups(parentAsInstanceName, pageSize: 500).AsRawResponses())
            {
                PrintBackups(page);
            }
    
            return allBackups;
        }
    
        private static void PrintBackups(IEnumerable<Backup> backups)
        {
            // We print the first 5 elements each time for demonstration purposes.
            // You can print all backups in the sequence by removing the call to Take(5).
            // If the sequence has been returned by a paginated operation it will lazily
            // fetch elements in pages as needed.
            foreach (Backup backup in backups.Take(5))
            {
                Console.WriteLine($"Backup Name : {backup.Name}");
            };
        }
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import (
     "context"
     "fmt"
     "io"
     "regexp"
     "time"
    
     database "cloud.google.com/go/spanner/admin/database/apiv1"
     adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
     "google.golang.org/api/iterator"
    )
    
    func listBackups(ctx context.Context, w io.Writer, db, backupID string) error {
     adminClient, err := database.NewDatabaseAdminClient(ctx)
     if err != nil {
         return err
     }
     defer adminClient.Close()
    
     matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
     if matches == nil || len(matches) != 3 {
         return fmt.Errorf("Invalid database id %s", db)
     }
     instanceName := matches[1]
    
     printBackups := func(iter *database.BackupIterator) error {
         for {
             resp, err := iter.Next()
             if err == iterator.Done {
                 return nil
             }
             if err != nil {
                 return err
             }
             fmt.Fprintf(w, "Backup %s\n", resp.Name)
         }
     }
    
     var iter *database.BackupIterator
     var filter string
     // List all backups.
     iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
         Parent: instanceName,
     })
     if err := printBackups(iter); err != nil {
         return err
     }
    
     // List all backups that contain a name.
     iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
         Parent: instanceName,
         Filter: "name:" + backupID,
     })
     if err := printBackups(iter); err != nil {
         return err
     }
    
     // List all backups that expire before a timestamp.
     expireTime := time.Now().AddDate(0, 0, 30)
     filter = fmt.Sprintf(`expire_time < "%s"`, expireTime.Format(time.RFC3339))
     iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
         Parent: instanceName,
         Filter: filter,
     })
     if err := printBackups(iter); err != nil {
         return err
     }
    
     // List all backups for a database that contains a name.
     iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
         Parent: instanceName,
         Filter: "database:" + db,
     })
     if err := printBackups(iter); err != nil {
         return err
     }
    
     // List all backups with a size greater than some bytes.
     iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
         Parent: instanceName,
         Filter: "size_bytes > 100",
     })
     if err := printBackups(iter); err != nil {
         return err
     }
    
     // List backups that were created after a timestamp that are also ready.
     createTime := time.Now().AddDate(0, 0, -1)
     filter = fmt.Sprintf(
         `create_time >= "%s" AND state:READY`,
         createTime.Format(time.RFC3339),
     )
     iter = adminClient.ListBackups(ctx, &adminpb.ListBackupsRequest{
         Parent: instanceName,
         Filter: filter,
     })
     if err := printBackups(iter); err != nil {
         return err
     }
    
     // List backups with pagination.
     request := &adminpb.ListBackupsRequest{
         Parent:   instanceName,
         PageSize: 10,
     }
     for {
         iter = adminClient.ListBackups(ctx, request)
         if err := printBackups(iter); err != nil {
             return err
         }
         pageToken := iter.PageInfo().Token
         if pageToken == "" {
             break
         } else {
             request.PageToken = pageToken
         }
     }
    
     fmt.Fprintf(w, "Backups listed.\n")
     return nil
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    static void listBackups(
        DatabaseAdminClient dbAdminClient, String projectId,
        String instanceId, String databaseId, String backupId) {
      InstanceName instanceName = InstanceName.of(projectId, instanceId);
      // List all backups.
      System.out.println("All backups:");
      for (Backup backup : dbAdminClient.listBackups(
          instanceName.toString()).iterateAll()) {
        System.out.println(backup);
      }
    
      // List all backups with a specific name.
      System.out.println(
          String.format("All backups with backup name containing \"%s\":", backupId));
      ListBackupsRequest listBackupsRequest =
          ListBackupsRequest.newBuilder().setParent(instanceName.toString())
              .setFilter(String.format("name:%s", backupId)).build();
      for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
        System.out.println(backup);
      }
    
      // List all backups for databases whose name contains a certain text.
      System.out.println(
          String.format(
              "All backups for databases with a name containing \"%s\":", databaseId));
      listBackupsRequest =
          ListBackupsRequest.newBuilder().setParent(instanceName.toString())
              .setFilter(String.format("database:%s", databaseId)).build();
      for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
        System.out.println(backup);
      }
    
      // List all backups that expire before a certain time.
      com.google.cloud.Timestamp expireTime = com.google.cloud.Timestamp.ofTimeMicroseconds(
          TimeUnit.MICROSECONDS.convert(
              System.currentTimeMillis() + TimeUnit.DAYS.toMillis(30), TimeUnit.MILLISECONDS));
    
      System.out.println(String.format("All backups that expire before %s:", expireTime));
      listBackupsRequest =
          ListBackupsRequest.newBuilder().setParent(instanceName.toString())
              .setFilter(String.format("expire_time < \"%s\"", expireTime)).build();
    
      for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
        System.out.println(backup);
      }
    
      // List all backups with size greater than a certain number of bytes.
      listBackupsRequest =
          ListBackupsRequest.newBuilder().setParent(instanceName.toString())
              .setFilter("size_bytes > 100").build();
    
      System.out.println("All backups with size greater than 100 bytes:");
      for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
        System.out.println(backup);
      }
    
      // List all backups with a create time after a certain timestamp and that are also ready.
      com.google.cloud.Timestamp createTime = com.google.cloud.Timestamp.ofTimeMicroseconds(
          TimeUnit.MICROSECONDS.convert(
              System.currentTimeMillis() - TimeUnit.DAYS.toMillis(1), TimeUnit.MILLISECONDS));
    
      System.out.println(
          String.format(
              "All databases created after %s and that are ready:", createTime.toString()));
      listBackupsRequest =
          ListBackupsRequest.newBuilder().setParent(instanceName.toString())
              .setFilter(String.format(
                  "create_time >= \"%s\" AND state:READY", createTime.toString())).build();
      for (Backup backup : dbAdminClient.listBackups(listBackupsRequest).iterateAll()) {
        System.out.println(backup);
      }
    
      // List backups using pagination.
      System.out.println("All backups, listed using pagination:");
      listBackupsRequest =
          ListBackupsRequest.newBuilder().setParent(instanceName.toString()).setPageSize(10).build();
      while (true) {
        ListBackupsPagedResponse response = dbAdminClient.listBackups(listBackupsRequest);
        for (Backup backup : response.getPage().iterateAll()) {
          System.out.println(backup);
        }
        String nextPageToken = response.getNextPageToken();
        if (!Strings.isNullOrEmpty(nextPageToken)) {
          listBackupsRequest = listBackupsRequest.toBuilder().setPageToken(nextPageToken).build();
        } else {
          break;
        }
      }
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * List backups in an instance.
     * Example:
     * ```
     * list_backups($instanceId, $databaseId);
     * ```
     *
     * @param string $instanceId The Spanner instance ID.
     */
    function list_backups(string $instanceId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
    
        // List all backups.
        print('All backups:' . PHP_EOL);
        foreach ($instance->backups() as $backup) {
            print('  ' . basename($backup->name()) . PHP_EOL);
        }
    
        // List all backups that contain a name.
        $backupName = 'backup-test-';
        print("All backups with name containing \"$backupName\":" . PHP_EOL);
        $filter = "name:$backupName";
        foreach ($instance->backups(['filter' => $filter]) as $backup) {
            print('  ' . basename($backup->name()) . PHP_EOL);
        }
    
        // List all backups for a database that contains a name.
        $databaseId = 'test-';
        print("All backups for a database which name contains \"$databaseId\":" . PHP_EOL);
        $filter = "database:$databaseId";
        foreach ($instance->backups(['filter' => $filter]) as $backup) {
            print('  ' . basename($backup->name()) . PHP_EOL);
        }
    
        // List all backups that expire before a timestamp.
        $expireTime = $spanner->timestamp(new \DateTime('+30 days'));
        print("All backups that expire before $expireTime:" . PHP_EOL);
        $filter = "expire_time < \"$expireTime\"";
        foreach ($instance->backups(['filter' => $filter]) as $backup) {
            print('  ' . basename($backup->name()) . PHP_EOL);
        }
    
        // List all backups with a size greater than some bytes.
        $size = 500;
        print("All backups with size greater than $size bytes:" . PHP_EOL);
        $filter = "size_bytes > $size";
        foreach ($instance->backups(['filter' => $filter]) as $backup) {
            print('  ' . basename($backup->name()) . PHP_EOL);
        }
    
        // List backups that were created after a timestamp that are also ready.
        $createTime = $spanner->timestamp(new \DateTime('-1 day'));
        print("All backups created after $createTime:" . PHP_EOL);
        $filter = "create_time >= \"$createTime\" AND state:READY";
        foreach ($instance->backups(['filter' => $filter]) as $backup) {
            print('  ' . basename($backup->name()) . PHP_EOL);
        }
    
        // List backups with pagination.
        print('All backups with pagination:' . PHP_EOL);
        $pages = $instance->backups(['pageSize' => 2])->iterateByPage();
        foreach ($pages as $pageNumber => $page) {
            print("All backups, page $pageNumber:" . PHP_EOL);
            foreach ($page as $backup) {
                print('  ' . basename($backup->name()) . PHP_EOL);
            }
        }
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def list_backups(instance_id, database_id, backup_id):
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
    
        # List all backups.
        print("All backups:")
        for backup in instance.list_backups():
            print(backup.name)
    
        # List all backups that contain a name.
        print('All backups with backup name containing "{}":'.format(backup_id))
        for backup in instance.list_backups(filter_="name:{}".format(backup_id)):
            print(backup.name)
    
        # List all backups for a database that contains a name.
        print('All backups with database name containing "{}":'.format(database_id))
        for backup in instance.list_backups(filter_="database:{}".format(database_id)):
            print(backup.name)
    
        # List all backups that expire before a timestamp.
        expire_time = datetime.utcnow().replace(microsecond=0) + timedelta(days=30)
        print(
            'All backups with expire_time before "{}-{}-{}T{}:{}:{}Z":'.format(
                *expire_time.timetuple()
            )
        )
        for backup in instance.list_backups(
            filter_='expire_time < "{}-{}-{}T{}:{}:{}Z"'.format(*expire_time.timetuple())
        ):
            print(backup.name)
    
        # List all backups with a size greater than some bytes.
        print("All backups with backup size more than 100 bytes:")
        for backup in instance.list_backups(filter_="size_bytes > 100"):
            print(backup.name)
    
        # List backups that were created after a timestamp that are also ready.
        create_time = datetime.utcnow().replace(microsecond=0) - timedelta(days=1)
        print(
            'All backups created after "{}-{}-{}T{}:{}:{}Z" and are READY:'.format(
                *create_time.timetuple()
            )
        )
        for backup in instance.list_backups(
            filter_='create_time >= "{}-{}-{}T{}:{}:{}Z" AND state:READY'.format(
                *create_time.timetuple()
            )
        ):
            print(backup.name)
    
        print("All backups with pagination")
        # If there are multiple pages, additional ``ListBackup``
        # requests will be made as needed while iterating.
        paged_backups = set()
        for backup in instance.list_backups(page_size=2):
            paged_backups.add(backup.name)
        for backup in paged_backups:
            print(backup)

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    # project_id  = "Your Google Cloud project ID"
    # instance_id = "Your Spanner instance ID"
    # backup_id = "Your Spanner database backup ID"
    # database_id = "Your Spanner databaseID"
    
    require "google/cloud/spanner"
    require "google/cloud/spanner/admin/database"
    
    database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin
    instance_path = database_admin_client.instance_path project: project_id, instance: instance_id
    
    puts "All backups"
    database_admin_client.list_backups(parent: instance_path).each do |backup|
      puts backup.name
    end
    
    puts "All backups with backup name containing \"#{backup_id}\":"
    database_admin_client.list_backups(parent: instance_path, filter: "name:#{backup_id}").each do |backup|
      puts backup.name
    end
    
    puts "All backups for databases with a name containing \"#{database_id}\":"
    database_admin_client.list_backups(parent: instance_path, filter: "database:#{database_id}").each do |backup|
      puts backup.name
    end
    
    puts "All backups that expire before a timestamp:"
    expire_time = Time.now + (30 * 24 * 3600) # 30 days from now
    database_admin_client.list_backups(parent: instance_path, filter: "expire_time < \"#{expire_time.iso8601}\"").each do |backup|
      puts backup.name
    end
    
    puts "All backups with a size greater than 500 bytes:"
    database_admin_client.list_backups(parent: instance_path, filter: "size_bytes >= 500").each do |backup|
      puts backup.name
    end
    
    puts "All backups that were created after a timestamp that are also ready:"
    create_time = Time.now - (24 * 3600) # From 1 day ago
    database_admin_client.list_backups(parent: instance_path, filter: "create_time >= \"#{create_time.iso8601}\" AND state:READY").each do |backup|
      puts backup.name
    end
    
    puts "All backups with pagination:"
    list = database_admin_client.list_backups parent: instance_path, page_size: 5
    list.each do |backup|
      puts backup.name
    end

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
