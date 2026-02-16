This quickstart shows you how to use the Google Cloud console to create a database in Spanner, insert data, and run a SQL query.

In the quickstart, you will:

  - Create a Spanner instance.
  - Create a database.
  - Create a schema.
  - Insert and modify data.
  - Run a query.

For information on the cost of using Spanner, see [Pricing](/spanner/pricing) .

**Note:** To explore Spanner using a 90-day free trial instance, see [Create a Spanner free trial instance](/spanner/docs/free-trial-quickstart) .

## Before you begin

1.  Sign in to your Google Cloud account. If you're new to Google Cloud, [create an account](https://console.cloud.google.com/freetrial) to evaluate how our products perform in real-world scenarios. New customers also get $300 in free credits to run, test, and deploy workloads.

2.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

3.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

4.  Enable the required API.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

5.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

6.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

7.  Enable the required API.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

8.  Optional: The Spanner API should be auto-enabled. If not, enable it manually:

9.  To get the permissions that you need to create instances and databases, ask your administrator to grant you the Cloud Spanner Admin (roles/spanner.admin) IAM role on your project.

## Create an instance with the Google Cloud console

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases in that instance.

1.  In the Google Cloud console, go to the **Spanner** page.

2.  Select or create a Google Cloud project if you haven't done so already.

3.  On the **Spanner** page, click **Create a provisioned instance** .
    
    If you've used Spanner before, you'll see the Spanner **Instances** page instead of the product page. Click **Create instance** .

4.  In the **Name your instance** page, enter an instance name, such as **Test Instance** .

5.  The instance ID is automatically entered based on the instance name, for example, as **test-instance** . Change it, if required. Click **Continue** .

6.  In the **Configure your instance** page, retain the default option **Regional** and select a configuration from the drop-down menu.
    
    Your instance configuration determines the geographic location where your instances are stored and replicated.

7.  Click **Continue** .

8.  In the **Allocate compute capacity** page, select **Processing units (PUs)** and retain the default value of 1000 processing units.

9.  Click **Create** .
    
    The Google Cloud console displays the **Overview** page for the instance you created.

## Create a database

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the instance you created, for example **Test Instance** .

3.  In the instance Overview page that opens, click **Create database** .

4.  For the database name, enter a name, such as **example-db** .

5.  Select a database dialect.
    
    For information about support for PostgreSQL and for guidance for choosing a dialect, see [PostgreSQL interface](/spanner/docs/postgresql-interface) . If you selected GoogleSQL, you'll define the schema in the **Define your schema** text field in the next section of this quickstart.
    
    Your database creation page now looks like this:

6.  Click **Create** .
    
    The Google Cloud console displays the **Overview** page for the database you created.

## Create a schema for your database

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

1.  In the navigation menu, click **Spanner Studio** .

2.  In the **Spanner Studio** page, click add **New tab** or use the empty editor tab.

3.  Enter:
    
    ### GoogleSQL
    
    ``` text
    CREATE TABLE Singers (
      SingerId   INT64 NOT NULL,
      FirstName  STRING(1024),
      LastName   STRING(1024),
      SingerInfo BYTES(MAX),
      BirthDate  DATE
    ) PRIMARY KEY(SingerId);
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE TABLE Singers (
      BirthDate  TIMESTAMPTZ,
      SingerId   BIGINT PRIMARY KEY,
      FirstName  VARCHAR(1024),
      LastName   VARCHAR(1024),
      SingerInfo BYTEA
    );
    ```

4.  Click play\_circle **Run** .
    
    The Google Cloud console returns to the database **Overview** page and shows that **Schema updates** are underway. When the update is complete, the page looks like this:
    
    ### GoogleSQL
    
    ### PostgreSQL
    
    Notice that PostgreSQL converts the table name to lowercase.

## Insert and modify data

The Google Cloud console provides an interface for inserting, editing, and deleting data.

### Insert data

1.  In the list of tables on the database **Overview** page, click the Singers table.
    
    The Google Cloud console displays the Singers table's **Schema** page.

2.  In the navigation menu, click **Data** to display the Singers table's **Data** page.

3.  Click **Insert** .
    
    The Google Cloud console displays the Singers table's Spanner Studio page with a new query tab that contains the `  INSERT  ` statement that you edit to insert a row in the Singers table and view the result of that insertion:
    
    ### GoogleSQL
    
    ``` text
     -- Add new values in the VALUES clause in order of the column list.
     -- Each value must be type compatible with its associated column.
    INSERT INTO
     Singers (SingerId,
       FirstName,
       LastName,
       SingerInfo,
       BirthDate)
    VALUES
     (<SingerId>, -- type: INT64
       <FirstName>, -- type: STRING(1024)
       <LastName>, -- type: STRING(1024)
       <SingerInfo>, -- type: BYTES(MAX)
       <BirthDate> -- type: DATE
       )
    THEN RETURN
     SingerId,
     FirstName,
     LastName,
     SingerInfo,
     BirthDate;
    ```
    
    ### PostgreSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        firstname,
        lastname,
        singerinfo,
        birthdate)
    VALUES
      (<singerid>, -- type: bigint
        <firstname>, -- type: character varying
        <lastname>, -- type: character varying
        <singerinfo>, -- type: bytea
        <birthdate> -- type: timestamp with time zone
        );
    THEN RETURN
       singerid,
       firstname,
       lastname,
       singerinfo,
       birthdate;
    ```
    
    Notice that PostgreSQL converts the column names to all lower case.

4.  Edit the `  INSERT  ` statement's `  VALUES  ` clause.
    
    ### GoogleSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      Singers (SingerId,
        BirthDate,
        FirstName,
        LastName,
        SingerInfo)
    VALUES
      (1, -- type: INT64
        NULL, -- type: DATE
        'Marc', -- type: STRING(1024)
        'Richards', -- type: STRING(1024)
        NULL -- type: BYTES(MAX)
        )
    THEN RETURN
      SingerId,
      FirstName,
      LastName,
      SingerInfo,
      BirthDate;
    ```
    
    ### PostgreSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        birthdate,
        firstname,
        lastname,
        singerinfo)
    VALUES
      (1, -- type: bigint
        NULL, -- type: timestamp with time zone
        'Marc', -- type: character varying
        'Richards', -- type: character varying
        NULL -- type: bytea
        );
    THEN RETURN
       singerid,
       firstname,
       lastname,
       singerinfo,
       birthdate;
    ```

5.  Click play\_circle **Run** .
    
    Spanner runs the statements. When finished, the **Results** tab shows that the statement inserted one row:
    
    ### GoogleSQL
    
    ### PostgreSQL

6.  In the Explorer, click more\_vert **View actions** next to the **Singers** table, and then click **Insert data** .

7.  Edit the `  INSERT  ` statement's `  VALUES  ` clause and the `  SELECT  ` statement's `  WHERE  ` clause:
    
    ### GoogleSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      Singers (SingerId,
        BirthDate,
        FirstName,
        LastName,
        SingerInfo)
    VALUES
      (2, -- type: INT64
        NULL, -- type: DATE
        'Catalina', -- type: STRING(1024)
        'Smith', -- type: STRING(1024)
        NULL -- type: BYTES(MAX)
        )
      -- Change values in the WHERE condition to match the inserted row.
    SELECT
      *
    FROM
      Singers
    WHERE
      SingerId=2;
    ```
    
    ### PostgreSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        birthdate,
        firstname,
        lastname,
        singerinfo)
    VALUES
      (2, -- type: bigint
        NULL, -- type: timestamp with time zone
        'Catalina', -- type: character varying
        'Smith', -- type: character varying
        NULL -- type: bytea
        );
      -- Change values in the WHERE condition to match the inserted row.
    SELECT
      *
    FROM
      singers
    WHERE
      singerid=2;
    ```

8.  Click play\_circle **Run** .
    
    After Spanner runs the statements, the **Results** tab shows that the statement inserted one row.

9.  In the Explorer, click more\_vert **View actions** next to the **Singers** table, and then click **Preview Data** .

10. Click play\_circle **Run** . The Singers table now has two rows:
    
    ### GoogleSQL
    
    ### PostgreSQL

You can also insert empty string values when you enter data.

1.  Click **Insert** to add a row.
    
    Spanner again displays the Singers table's **Spanner Studio** page with a new query tab that contains the same `  INSERT  ` and `  SELECT  ` statements.

2.  Edit the template `  INSERT  ` statement's `  VALUES  ` clause and `  SELECT  ` statement's `  WHERE  ` clause:
    
    ### GoogleSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      Singers (SingerId,
        BirthDate,
        FirstName,
        LastName,
        SingerInfo)
    VALUES
      (3, -- type: INT64
        NULL, -- type: DATE
        'Kena', -- type: STRING(1024)
        '', -- type: STRING(1024)
        NULL -- type: BYTES(MAX)
        );
      -- Change values in the WHERE condition to match the inserted row.
    SELECT
      *
    FROM
      Singers
    WHERE
      SingerId=3;
    ```
    
    ### PostgreSQL
    
    ``` text
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        birthdate,
        firstname,
        lastname,
        singerinfo)
    VALUES
      (3, -- type: bigint
        NULL, -- type: timestamp with time zone
        'Kena', -- type: character varying
        '', -- type: character varying
        NULL -- type: bytea
        );
      -- Change values in the WHERE condition to match the inserted row.
    SELECT
      *
    FROM
      singers
    WHERE
      singerid=3;
    ```
    
    Notice that the value provided for the last name column is an empty string, `  ''  ` , not a `  NULL  ` value.

3.  Click play\_circle **Run** .
    
    After Spanner runs the statements, the **Results** tab shows that the statement inserted one row.

4.  In the Explorer, click more\_vert **View actions** next to the **Singers** table, and then click **Preview Data** .

5.  Click play\_circle **Run** . The `  Singers  ` table now has three rows, and the row with the primary key value of `  3  ` has an empty string in the `  LastName  ` column:
    
    ### GoogleSQL
    
    ### PostgreSQL

### Edit data

1.  On the Singers table's **Data** page, select the checkbox on the row with the primary key value of `  3  ` , and then click **Edit** .
    
    The Spanner displays the **Spanner Studio** page with a new tab containing template `  UPDATE  ` and `  SET  ` statements that you can edit. Note that the `  WHERE  ` clauses of both statements indicate that the row to update is the one with the primary key value of `  3  ` .
    
    ### GoogleSQL
    
    ``` text
      -- Change values in the SET clause to update the row where the WHERE condition is true.
    UPDATE
      Singers
    SET
      BirthDate='',
      FirstName='Kena',
      LastName='',
      SingerInfo=''
    WHERE
      SingerId=3;
    SELECT
      *
    FROM
      Singers
    WHERE
      SingerId=3;
    ```
    
    ### PostgreSQL
    
    ``` text
      -- Change values in the SET clause to update the row where the WHERE condition is true.
    UPDATE
      singers
    SET
      birthdate=NULL,
      firstname='Kena',
      lastname='',
      singerinfo=NULL
    WHERE
      singerid='3';
    SELECT
      *
    FROM
      singers
    WHERE
      singerid='3';
    ```

2.  Edit the `  UPDATE  ` statement's `  SET  ` clause to update only the birth date:
    
    ### GoogleSQL
    
    ``` text
      -- Change values in the SET clause to update the row where the WHERE condition is true.
    UPDATE
      Singers
    SET
      BirthDate='1961-04-01'
    WHERE
      SingerId=3;
    SELECT
      *
    FROM
      Singers
    WHERE
      SingerId=3;
    ```
    
    ### PostgreSQL
    
    ``` text
      -- Change values in the SET clause to update the row where the WHERE condition is true.
    UPDATE
      singers
    SET
      birthdate='1961-04-01 00:00:00 -8:00'
    WHERE
      singerid='3';
    SELECT
      *
    FROM
      singers
    WHERE
      singerid='3';
    ```

3.  Click play\_circle **Run** .
    
    Spanner runs the statements. When finished, the **Results** tab shows that the first statement updated one row.

4.  In the Explorer, click more\_vert **View actions** next to the **Singers** table, and then click **Preview Data** .

5.  Click play\_circle **Run** . The updated row now has a value for the birth date.
    
    ### GoogleSQL
    
    ### PostgreSQL

### Delete data

1.  On the Singers table's **Data** page, select the checkbox on the row with `  2  ` in the first column, and then click **Delete** .

2.  In the dialog that appears, click **Confirm** .
    
    The Singers table now has two rows:
    
    ### GoogleSQL
    
    ### PostgreSQL

## Run a query in the Google Cloud console

1.  On the database **Overview** page, click **Spanner Studio** in the navigation menu.

2.  Click **New tab** to create a new query tab. Then, enter the following query in the query editor:
    
    ### GoogleSQL
    
    ``` text
    SELECT * FROM Singers;
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT * FROM singers;
    ```

3.  Click play\_circle **Run** .
    
    Spanner runs the query. When finished, the **Results** tab displays the result of your query:
    
    ### GoogleSQL
    
    ### PostgreSQL

Congratulations\! You've successfully created a Spanner database and executed a SQL statement by using the query editor\!

## Clean up

To avoid incurring additional charges to your Google Cloud account, delete the database and instance. Disabling the Cloud Billing API doesn't stop charges. When you delete an instance, all databases in the instance are deleted.

### Delete the database

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the name of the instance that has the database that you want to delete, for example, **Test Instance** .

3.  Click the name of the database that you want to delete, for example, **example-db** .

4.  In the **Database details** page, click *delete* **Delete database** .

5.  Confirm that you want to delete the database by entering the database name and clicking **Delete** .

### Delete the instance

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the name of the instance that you want to delete, for example, **Test Instance** .

3.  Click *delete* **Delete instance** .

4.  Confirm that you want to delete the instance by entering the instance name and clicking **Delete** .

## What's next

  - Learn about [Instances](/spanner/docs/instances) .
  - Understand the Spanner [Schema and Data Model](/spanner/docs/schema-and-data-model) .
  - Learn more about [GoogleSQL Data Definition Language (DDL)](/spanner/docs/reference/standard-sql/data-definition-language) .
  - Learn more about [Query Execution Plans](/spanner/docs/query-execution-plans) .
  - Learn how to use Spanner with [C++](/spanner/docs/getting-started/cpp) , [C\#](/spanner/docs/getting-started/csharp) , [Go](/spanner/docs/getting-started/go) , [Java](/spanner/docs/getting-started/java) , [Node.js](/spanner/docs/getting-started/nodejs) , [PHP](/spanner/docs/getting-started/php) , [Python](/spanner/docs/getting-started/python) , [Ruby](/spanner/docs/getting-started/ruby) , [REST](/spanner/docs/getting-started/rest) , or [gcloud](/spanner/docs/getting-started/gcloud) .
