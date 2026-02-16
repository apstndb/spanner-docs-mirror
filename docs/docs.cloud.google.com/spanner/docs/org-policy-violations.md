This document describes how to troubleshoot customer-managed encryption key (CMEK) and data residency organization policy violations in Spanner. To help you monitor your database fleet, Database Center detects CMEK and data residency organization policy violations using the following health check:

  - An *Encryption org policy not satisfied* violation indicates that a CMEK organization policy on a Spanner database isn't satisfied.

  - A *Location org policy not satisfied* violation indicates that a database is in a region that's not allowed by an organization policy. This can happen when a database was created in an allowed region, but after the database was created an organization policy disallowed the region.

If you see this violations in Database Center, use the topic in this document to fix the issue. To learn more about Database Center, see [Database Center overview](/database-center/docs/overview) .

## Troubleshoot CMEK violations

If an *Encryption org policy not satisfied* violation on a Spanner database occurs in Database Center, you need to create a new database from a backup of the database on which the violation occurred. To learn more about CMEK in Spanner, see [CMEK overview](/spanner/docs/cmek) . To learn more about CMEK in Cloud Key Management Service, see [Customer-managed encryption keys](/kms/docs/cmek) . To create a new database from a backup, follow these steps:

1.  If you don't have a key ring, create one using the steps in [Create a key ring](/kms/docs/create-key-ring) .

2.  If you don't have a valid customer managed key, create one using the steps in [Create a key](/kms/docs/create-key) .

3.  Create a backup of the database with the policy violation. For more information, see [Create a backup](/spanner/docs/backup/create-backups#create-backup) . You can use an encryption key when you create the backup. If you don't, then you can specify an encryption key in the next step.

4.  Restore the backup using the steps in [Restore from a backup](/spanner/docs/use-cmek#restore) . Choose one of the following when you create your restored database:
    
      - If you used a CMEK key when you created the backup, then choose **Use existing encryption** .
    
      - If you didn't encrypt the backup, then choose **Cloud KMS key** .

## Troubleshoot data residency violations

If a *Location org policy not satisfied* violation on a Spanner database occurs in Database Center, then you need to move the database to an instance that's in an allowed region. For more information about allowed regions, see [Resource locations](/resource-manager/docs/organization-policy/defining-locations) .

To move a database, follow these steps:

1.  Make sure you have an available instance in an allowed region. To see a list of available instance configurations, run the following Google Cloud CLI command:
    
    ``` text
    gcloud spanner instance-configs list
    ```
    
    If you need to create a new instance, see [Create a custom instance configuration](/spanner/docs/create-manage-configurations) .

2.  Use the [`  gcloud spanner instances move  `](/sdk/gcloud/reference/spanner/instances/move) command to move the database to the new instance.

To prevent a database from being created in a region, add the region to the `  denied_values  ` list when you set the organization policy for the database. For more information, see [Set the organization policy](/resource-manager/docs/organization-policy/defining-locations#setting_the_organization_policy) .
