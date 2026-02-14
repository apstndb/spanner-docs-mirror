This page describes how to grant Spanner Identity and Access Management (IAM) permissions to an account for a Google Cloud project, instance, database, or backup.

For information on Google Cloud roles, see [Understanding roles](/iam/docs/understanding-roles) , and for more information on Spanner roles, see [Access control: roles](/spanner/docs/iam#roles) .

**Note:** If your account doesn't have sufficient permissions, you won't be able to view some of the items in the following directions. In this case, ask your project's owner to grant you additional permissions.

## Project-level permissions

You can grant IAM permissions for an entire Google Cloud project to an account in the **IAM** page of the Google Cloud console. Adding permissions at the project level grants the IAM permissions to an account for all Spanner instances, databases, and backups in the project.

### Verify that you can add permissions

Before you attempt to apply project-level permissions, check that you have sufficient permissions to apply roles to another account. You need permissions at the project level.

1.  Go to your project's **IAM** page.

2.  Select **Principals** as the **View by** option.

3.  Find your account in the list. If your account is listed as **Owner** or **Editor** in the **Role** column, you have sufficient permissions.

If you don't have sufficient permissions at the project level, ask the project's owner to grant you additional permissions.

### Grant permissions to principals

1.  Go to your project's **IAM** page.

2.  Select **Principals** as the **View by** option.

3.  Find the account in the list and click **Edit** .

4.  On the **Edit permissions** page, click **Add Another Role** .

5.  Select a role in the drop-down list.

6.  Click **Save** .

### Add principals to the project

1.  Go to your project's **IAM** page.

2.  Click the **Add** button below the toolbar.

3.  In the **New principals** box, enter the email for the account that you want to add.

4.  Select a role in the drop-down list.

5.  Click **Save** .

For more information, see [Granting, changing, and revoking access](/iam/docs/granting-changing-revoking-access) .

## Instance-level permissions

You can grant instance-level IAM permissions to an account in the **IAM** page of the Google Cloud console.

### Verify that you can add permissions

Before you attempt to apply instance-level permissions at the instance level, check that you have sufficient permissions to apply roles to another account. You need permissions at the project or instance level.

1.  Go to your project's **IAM** page.

2.  Select **Principals** as the **View by** option.

3.  Find your account in the list. If your account is listed as **Owner** , **Editor** , or **Cloud Spanner Admin** in the **Role** column, you have sufficient permissions. If not, continue to the next step.

4.  Go to the Spanner **Instances** page.

5.  Select the checkbox for the instance.

6.  In the **Permissions** tab of the **Info panel** , expand the principal lists and find your account. If your account is listed as **Owner** , **Editor** , or **Spanner Admin** , you have sufficient permissions.

If you don't have sufficient permissions at the project or instance level, ask the project's owner to grant you additional permissions.

### Add instance-level permissions

Use the following steps to apply roles for Spanner to an instance in a project.

1.  Go to the Spanner **Instances** page.

2.  Select the checkbox for the instance.

3.  Click the **Permissions** tab in the **Info panel** .

4.  In the **Add principals** box in the **Info panel** , enter the email address for the account that you want to add.

5.  Select one or more roles in the drop-down list.

6.  Click **Add** .

## Database-level permissions

You can grant database-level IAM permissions to an account in the **IAM** page of the Google Cloud console.

### Verify that you can add permissions

Before you attempt to apply database-level permissions, check that you have sufficient permissions to apply roles to another account. You need permissions at the project, instance, or database level.

1.  Go to your project's **IAM** page.

2.  Select **Principals** as the **View by** option.

3.  Find your account in the list. If your account is listed as **Owner** , **Editor** , **Cloud Spanner Admin** , or **Cloud Spanner Database Admin** in the **Role** column, you have sufficient permissions. If not, continue to the next step.

4.  Go to the Spanner **Instances** page.

5.  Select the checkbox for the instance that contains your database.

6.  In the **Permissions** tab of the **Info panel** , expand the principal lists and find your account. If your account is listed as **Owner** , **Editor** , **Spanner Admin** or **Spanner Database Admin** , you have sufficient permissions. If not, continue to the next step.

7.  Click the instance name to go to the **Instance details** page.

8.  Click **Show Info panel** .

9.  In the **Overview** tab of the page, select the checkbox for your database.

10. In the **Permissions** tab of the **Info panel** , expand the principal lists and find your account. If your account is listed as **Owner** , **Editor** , **Spanner Admin** , or **Spanner Database Admin** , you have sufficient permissions.

If you don't have sufficient permissions at the project, instance, or database level, ask the project's owner to grant you additional permissions.

### Add database-level permissions

Follow these steps to grant access to database-level roles for a principal.

1.  Go to the Spanner **Instances** page.

2.  Click the name of the instance that contains your database to go to the **Instance details** page.

3.  In the **Overview** tab, select the checkbox for your database.  
    The **Info panel** appears.

4.  Click **Add principal** .

5.  In the **Add principals** panel, in **New principals** , enter the email address for the account that you want to add.

6.  Select one or more roles in the drop-down list.

7.  Click **Save** .

### Remove database-level permissions

Follow these steps to remove database-level roles from a principal.

**Note:** This procedure assumes that you know the role that you want to remove from a principal. If you don't know which roles a principal has, use the IAM Google Cloud console to first view the roles that a principal has, and then revoke the roles. For details, see [Revoke a single role](/iam/docs/granting-changing-revoking-access#revoke-single-role) .

1.  Go to the Spanner **Instances** page.

2.  Click the name of the instance that contains your database to go to the **Instance details** page.

3.  In the **Overview** tab, select the checkbox for your database.  
    The **Info panel** appears.

4.  In the **Info panel** , under **Role/Principal** , locate the database-level role that you want to remove, and expand it.
    
    A list of principals who have this role is shown.

5.  Click the trash icon adjacent to the principal from whom you want to remove the role.

6.  In the confirmation dialog, select the checkbox and click **REMOVE** .

## Backup-level permissions

You can grant backup-level IAM permissions to an account in the **IAM** page of the Google Cloud console.

### Verify that you can add permissions

Before you attempt to apply backup-level permissions, check that you have sufficient permissions to apply roles to another account. You need permissions at the project, instance, or backup.

1.  Go to your project's **IAM** page.

2.  Select **Principals** as the **View by** option.

3.  Find your account in the list. If your account is listed as **Owner** , **Editor** , **Cloud Spanner Admin** , **Cloud Spanner Backup Admin** in the **Role** column, you have sufficient permissions. If not, continue to th next step.

4.  Go to the Spanner **Instances** page.

5.  Select the checkbox for the instance that contains your backup.

6.  In the **Permissions** tab of the **Info panel** , expand the principal lists and find your account. If your account is listed as **Owner** , **Editor** , **Spanner Admin** or **Spanner Backup Admin** , you have sufficient permissions. If not, continue to the next step.

7.  Click the instance name to go to the **Instance details** page.

8.  Click the **Backup/Restore** tab and select your backup from the **Backup** table.

9.  Click **Show Info Panel** .

10. In the **Info Panel** find your account. If your account is listed as **Owner** , **Editor** , **Cloud Spanner Admin** , or **Cloud Spanner Backup Admin** in the **Role** column, you have sufficient permissions.

If you don't have sufficient permissions at the project or instance level, ask the project's owner to grant you additional permissions.

### Add backup-level permissions

Use the following steps to apply roles for Spanner to an individual backup in a project.

1.  Go to the Spanner **Instances** page.

2.  Click the name of the instance that contains your backup to go to the **Instance details** page.

3.  In the **Backup/Restore** tab, select your backup.  
    The **Info panel** appears.

4.  Click the **Permissions** tab in the **Info panel** .

5.  In the **Add principals** box in the **Info panel** , enter the email address for the account that you want to add.

6.  Select one or more roles in the drop-down list.

7.  Click **Add** .
