> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

[Identity and Access Management (IAM)](https://docs.cloud.google.com/iam/docs/overview) lets you control user access to Spanner Omni resources. For example, a user can have full control of a specific database, but cannot create or modify other databases in your deployment. Using IAM lets you grant a permission to a user without having to modify each Spanner Omni database permission individually.

This document focuses on the IAM *permissions* relevant to Spanner Omni and the IAM *roles* that grant those permissions. For a detailed description of IAM and its features, see the [Identity and Access Management](https://docs.cloud.google.com/iam/docs/overview) developer's guide.

> The following are differences between Spanner IAM and Spanner Omni IAM:
> 
>   - Spanner Omni doesn't support custom roles.
>   - Roles in Spanner Omni don't contain permissions outside the Spanner namespace, such as those not prefixed by `spanner.` .
>   - Spanner Omni shares some, but not all, of the permissions used by Spanner.
>   - Spanner Omni includes unique permissions not present in Spanner.

## Permissions

Permissions allow users to perform specific actions on Spanner Omni resources. For example, the `spanner.databases.read` permission allows a user to read from a database using Spanner Omni's read API, while `spanner.databases.export` lets a user export a Spanner Omni database. You don't directly give users permissions; instead, you grant them [predefined roles](https://docs.cloud.google.com/spanner-omni/iam#roles) , which have one or more permissions bundled within them.

The following tables list the IAM permissions that are associated with Spanner Omni. Some permissions are shared with Spanner and some are used by only Spanner Omni.

### Databases

The following permissions apply to Spanner Omni databases.

| Database permission name           | Description                                                                                              | Spanner and Spanner Omni | Spanner Omni only |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------- | :----------------------: | ----------------- |
| `spanner.databases.create`         | Create a database.                                                                                       |            ✔             |                   |
| `spanner.databases.createBackup`   | Create a backup from the database. Also requires `spanner.backups.create` to create the backup resource. |            ✔             |                   |
| `spanner.databases.get`            | Get a database's metadata.                                                                               |            ✔             |                   |
| `spanner.databases.getIamPolicy`   | Get a database's IAM policy.                                                                             |            ✔             |                   |
| `spanner.databases.list`           | List databases.                                                                                          |            ✔             |                   |
| `spanner.databases.read`           | Read from a database using the read API.                                                                 |            ✔             |                   |
| `spanner.databases.setIamPolicy`   | Set a database's IAM policy.                                                                             |            ✔             |                   |
| `spanner.databases.update`         | Update a database's metadata.                                                                            |            ✔             |                   |
| `spanner.databases.updateDdl`      | Update a database's schema.                                                                              |            ✔             |                   |
| `spanner.databases.write`          | Write into a database.                                                                                   |            ✔             |                   |
| `spanner.databases.compact`        | Compacts tables in a database                                                                            |                          | ✔                 |
| `spanner.databases.export`         | Exports a Spanner Omni database                                                                          |                          | ✔                 |
| `spanner.databases.import`         | Imports a Spanner Omni database                                                                          |                          | ✔                 |
| `spanner.databases.addSplitPoints` | Adds split points to a database.                                                                         |            ✔             |                   |

### Database operations

The following permissions apply to Spanner Omni database operations.

| Database operation permission name  | Description                                    | Spanner and Spanner Omni | Spanner Omni only |
| ----------------------------------- | ---------------------------------------------- | :----------------------: | ----------------- |
| `spanner.databaseOperations.cancel` | Cancel a database operation.                   |            ✔             |                   |
| `spanner.databaseOperations.delete` | Delete a database operation.                   |            ✔             |                   |
| `spanner.databaseOperations.get`    | Get a specific database operation.             |            ✔             |                   |
| `spanner.databaseOperations.list`   | List database and restore database operations. |            ✔             |                   |

### Backups

The following permissions apply to Spanner Omni backups.

| Backup permission name                     | Description                                                                                               | Spanner and Spanner Omni | Spanner Omni only |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------- | :----------------------: | ----------------- |
| `spanner.backups.copy`                     | Copy a backup.                                                                                            |            ✔             |                   |
| `spanner.backups.create`                   | Create a backup. Also requires `spanner.databases.createBackup` on the source database.                   |            ✔             |                   |
| `spanner.backups.createDatabaseFromBackup` | Create a database from a backup.                                                                          |            ✔             |                   |
| `spanner.backups.delete`                   | Delete a backup.                                                                                          |            ✔             |                   |
| `spanner.backups.get`                      | Get a backup.                                                                                             |            ✔             |                   |
| `spanner.backups.getIamPolicy`             | Get a backup's IAM policy.                                                                                |            ✔             |                   |
| `spanner.backups.list`                     | List backups.                                                                                             |            ✔             |                   |
| `spanner.backups.restoreDatabase`          | Restore database from a backup. Also requires `spanner.databases.create` to create the restored database. |            ✔             |                   |
| `spanner.backups.setIamPolicy`             | Set a backup's IAM policy.                                                                                |            ✔             |                   |
| `spanner.backups.update`                   | Update a backup.                                                                                          |            ✔             |                   |
| `spanner.backups.import`                   | Imports a backup from external storage                                                                    |                          | ✔                 |

### Backup operations

The following permissions apply to Spanner Omni backup operations.

| Backup operation permission name  | Description                      | Spanner and Spanner Omni | Spanner Omni only |
| --------------------------------- | -------------------------------- | :----------------------: | ----------------- |
| `spanner.backupOperations.cancel` | Cancel a backup operation.       |            ✔             |                   |
| `spanner.backupOperations.get`    | Get a specific backup operation. |            ✔             |                   |
| `spanner.backupOperations.list`   | List backup operations.          |            ✔             |                   |

### Backup schedules

The following permissions apply to Spanner Omni backup schedules.

| Backup schedule permission name        | Description                                                                                      | Spanner and Spanner Omni | Spanner Omni only |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ | :----------------------: | ----------------- |
| `spanner.backupSchedules.create`       | Create a backup schedule. Also requires `spanner.databases.createBackup` on the source database. |            ✔             |                   |
| `spanner.backupSchedules.delete`       | Delete a backup schedule.                                                                        |            ✔             |                   |
| `spanner.backupSchedules.get`          | Get a backup schedule.                                                                           |            ✔             |                   |
| `spanner.backupSchedules.list`         | List backup schedules.                                                                           |            ✔             |                   |
| `spanner.backupSchedules.update`       | Update a backup schedule.                                                                        |            ✔             |                   |
| `spanner.backupSchedules.getIamPolicy` | Get a backup schedule's IAM policy.                                                              |            ✔             |                   |
| `spanner.backupSchedules.setIamPolicy` | Set a backup schedule's IAM policy.                                                              |            ✔             |                   |

### Backup descriptors

The following permissions apply to Spanner Omni backup descriptors.

| Backup descriptor permission name  | Description                               | Spanner and Spanner Omni | Spanner Omni only |
| ---------------------------------- | ----------------------------------------- | ------------------------ | :---------------: |
| `spanner.backupDescriptors.import` | Imports a backup from a backup descriptor |                          |         ✔         |
| `spanner.backupDescriptors.list`   | List backup descriptors.                  |                          |         ✔         |

### Sessions

The following permissions apply to Spanner Omni sessions.

| Session permission name   | Description       | Spanner and Spanner Omni | Spanner Omni only |
| ------------------------- | ----------------- | :----------------------: | ----------------- |
| `spanner.sessions.create` | Create a session. |            ✔             |                   |
| `spanner.sessions.delete` | Delete a session. |            ✔             |                   |
| `spanner.sessions.get`    | Get a session.    |            ✔             |                   |
| `spanner.sessions.list`   | List sessions.    |            ✔             |                   |

### Location and zones

The following permissions apply to Spanner Omni locations and zones.

| Permission name                    | Description                             | Spanner and Spanner Omni | Spanner Omni only |
| ---------------------------------- | --------------------------------------- | ------------------------ | :---------------: |
| `spanner.locations.create`         | Create a Spanner Omni location          |                          |         ✔         |
| `spanner.locations.delete`         | Delete a Spanner Omni location          |                          |         ✔         |
| `spanner.locations.get`            | Get a Spanner Omni location             |                          |         ✔         |
| `spanner.locations.list`           | List Spanner Omni locations             |                          |         ✔         |
| `spanner.locationDistances.create` | Create a Spanner Omni location distance |                          |         ✔         |
| `spanner.locationDistances.delete` | Delete a Spanner Omni location distance |                          |         ✔         |
| `spanner.locationDistances.get`    | Get a Spanner Omni location distance    |                          |         ✔         |
| `spanner.locationDistances.list`   | List Spanner Omni location distances    |                          |         ✔         |
| `spanner.locationDistances.update` | Update a Spanner Omni location distance |                          |         ✔         |
| `spanner.zones.create`             | Create a Spanner Omni zone              |                          |         ✔         |
| `spanner.zones.delete`             | Delete a Spanner Omni zone              |                          |         ✔         |
| `spanner.zones.get`                | Get a Spanner Omni zone                 |                          |         ✔         |
| `spanner.zones.list`               | List Spanner Omni zones                 |                          |         ✔         |

### Servers

The following permissions apply to Spanner Omni servers.

| Server permission name   | Description                   | Spanner and Spanner Omni | Spanner Omni only |
| ------------------------ | ----------------------------- | ------------------------ | :---------------: |
| `spanner.servers.create` | Creates a Spanner Omni server |                          |         ✔         |
| `spanner.servers.delete` | Deletes a Spanner Omni server |                          |         ✔         |
| `spanner.servers.get`    | Gets a Spanner Omni server    |                          |         ✔         |
| `spanner.servers.list`   | Lists Spanner Omni servers    |                          |         ✔         |

### Users and roles

The following permissions apply to Spanner Omni users and roles.

| Permission name        | Description                 | Spanner and Spanner Omni | Spanner Omni only |
| ---------------------- | --------------------------- | ------------------------ | :---------------: |
| `spanner.users.create` | Creates a Spanner Omni user |                          |         ✔         |
| `spanner.users.delete` | Deletes a Spanner Omni user |                          |         ✔         |
| `spanner.users.get`    | Gets a Spanner Omni user    |                          |         ✔         |
| `spanner.users.list`   | Lists Spanner Omni users    |                          |         ✔         |
| `spanner.users.update` | Updates a Spanner Omni user |                          |         ✔         |
| `spanner.roles.get`    | Gets a Spanner Omni role    |                          |         ✔         |
| `spanner.roles.list`   | Lists Spanner Omni roles    |                          |         ✔         |

### External storage

The following permissions apply to Spanner Omni external storage.

| External storage permission name        | Description                                | Spanner and Spanner Omni | Spanner Omni only |
| --------------------------------------- | ------------------------------------------ | ------------------------ | :---------------: |
| `spanner.externalStorages.create`       | Creates external storage                   |                          |         ✔         |
| `spanner.externalStorages.delete`       | Deletes an external storage                |                          |         ✔         |
| `spanner.externalStorages.get`          | Gets external storage                      |                          |         ✔         |
| `spanner.externalStorages.getIamPolicy` | Gets the IAM policy of an external storage |                          |         ✔         |
| `spanner.externalStorages.list`         | Lists external storages                    |                          |         ✔         |
| `spanner.externalStorages.setIamPolicy` | Sets the IAM policy of an external storage |                          |         ✔         |

### File system and descriptors

The following permissions apply to the Spanner Omni file system.

| Permission name             | Description                                        | Spanner and Spanner Omni | Spanner Omni only |
| --------------------------- | -------------------------------------------------- | ------------------------ | :---------------: |
| `spanner.filesystem.cat`    | Prints files in the Spanner Omni file system       |                          |         ✔         |
| `spanner.filesystem.ls`     | Lists files in the Spanner Omni file system        |                          |         ✔         |
| `spanner.descriptors.print` | Prints descriptors in the Spanner Omni file system |                          |         ✔         |

### Other administrator permissions

The following permissions apply to other Spanner Omni administrator tasks.

| Permission name               | Description                                                | Spanner and Spanner Omni | Spanner Omni only |
| ----------------------------- | ---------------------------------------------------------- | ------------------------ | :---------------: |
| `spanner.chubby.list`         | Lists Chubby cells                                         |                          |         ✔         |
| `spanner.chubby.print`        | Prints Chubby cell contents                                |                          |         ✔         |
| `spanner.deployment.get`      | Get Spanner Omni deployment                                |                          |         ✔         |
| `spanner.diagnostics.create`  | Collects artifacts from Spanner Omni servers for debugging |                          |         ✔         |
| `spanner.internal-tables.sql` | Runs SQL queries against internal tables                   |                          |         ✔         |
| `spanner.logs.copy`           | Copies logs from a Spanner Omni server                     |                          |         ✔         |
| `spanner.tablet.move`         | Moves a tablet from one Spanner Omni server to another     |                          |         ✔         |
| `spanner.workflows.delete`    | Deletes a workflow in a database                           |                          |         ✔         |
| `spanner.groups.compact`      | Compacts tablets in a group                                |                          |         ✔         |
| `spanner.directories.compact` | Compacts tablets in a directory                            |                          |         ✔         |

## Predefined roles

A predefined role is a bundle of one or more [permissions](https://docs.cloud.google.com/spanner-omni/iam#permissions) . Spanner Omni supports the following predefined roles:

| Role                           | Description                                                                                 |
| ------------------------------ | ------------------------------------------------------------------------------------------- |
| `roles/spanner.admin`          | Has complete access to all Spanner Omni resources. Includes all permissions.                |
| `roles/spanner.backupAdmin`    | Has complete access to Spanner Omni backups and backup operations.                          |
| `roles/spanner.backupWriter`   | Can create backups, but cannot update or delete them.                                       |
| `roles/spanner.databaseAdmin`  | Has complete access to all Spanner Omni databases in a project.                             |
| `roles/spanner.databaseReader` | Can read from the Spanner Omni database and view schema.                                    |
| `roles/spanner.databaseUser`   | Can read from and write to the Spanner Omni database.                                       |
| `roles/spanner.editor`         | Editor role for Spanner Omni.                                                               |
| `roles/spanner.restoreAdmin`   | Can restore a database from a backup.                                                       |
| `roles/spanner.viewer`         | Can view all Spanner Omni resources but cannot modify them. Includes read-only permissions. |

Spanner Omni has a few more notable limitations:

  - No custom roles.
  - No fine-grained access control.
  - No IAM conditions.

## What's next

  - Learn more about [IAM](https://docs.cloud.google.com/iam/docs/overview) .
  - Learn about [Authentication and authorization in Spanner Omni](https://docs.cloud.google.com/spanner-omni/authentication) .
