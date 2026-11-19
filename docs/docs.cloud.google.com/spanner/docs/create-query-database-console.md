---
name: documents/docs.cloud.google.com/spanner/docs/create-query-database-console
uri: https://docs.cloud.google.com/spanner/docs/create-query-database-console
title: Create and query a database in the Google Cloud console
description: This quickstart shows you how to use the {{dynamic_data.site_values.cloud_name_short}} console to create a database in Spanner, insert data, and run a SQL query.
data_source: docs.cloud.google.com
---

This quickstart shows you how to use the Google Cloud console to create a database in Spanner, insert data, and run a SQL query.

In the quickstart, you will:

- Create a Spanner instance.
- Create a database.
- Create a schema.
- Insert and modify data.
- Run a query.

For information on the cost of using Spanner, see [Pricing](https://docs.cloud.google.com/spanner/pricing) .

> **Note:** To explore Spanner using a 90-day free trial instance, see [Create a Spanner free trial instance](https://docs.cloud.google.com/spanner/docs/free-trial-quickstart) .

## Before you begin

1.  Optional: The Cloud Spanner API is enabled automatically when you create a Spanner instance. If it isn't, enable it manually:

2.  To get the permissions that you need to create instances and databases, ask your administrator to grant you the Cloud Spanner Admin (roles/spanner.admin) IAM role on your project.

## Create an instance with the Google Cloud console

When you first use Spanner, you must create an instance, which is an allocation of resources that Spanner databases use in that instance.

1.  In the Google Cloud console, go to the **Spanner** page.

2.  On the **Spanner** page, click **Create a provisioned instance** .

    If you've used Spanner before, you see the Spanner **Instances** page instead of the product page. Click **Create instance** .

3.  In the **Select an edition** section, select **Standard** , and then click **Continue** .

4.  In the **Name your instance** section, enter an **Instance name** , such as `Test Instance` .

    Spanner populates the **Instance ID** based on the instance name, for example, `test-instance` . Change the **Instance ID** if required, and then click **Continue** .

5.  In the **Configure your instance** section, for **Choose a configuration** , retain the default option **Regional** and select a configuration location from the drop-down menu.

    Your instance configuration determines the geographic location where Spanner stores and replicates your instances.

6.  Click **Continue** .

7.  In the **Configure compute capacity** section, for **Select unit** , select **Processing units (PUs)** . For **Choose a scaling mode** , retain **Manual allocation** and the default **Quantity** of `1000` processing units.

8.  Optional: In the **Backups** section, clear the **Enable default backup schedules** checkbox if you don't want default backup schedules created for this test instance.

9.  Click **Create** .

    The Google Cloud console displays the **Overview** page for the instance that you created.

## Create a database

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the instance you created, for example **Test Instance** .

3.  On the instance **Overview** page that opens, click **Create database** .

4.  For the database name, enter a name, such as `example-db` .

5.  Select a database dialect.

    For information about support for PostgreSQL and for guidance for choosing a dialect, see [PostgreSQL interface](https://docs.cloud.google.com/spanner/docs/postgresql-interface) . Leave the **Define your schema (optional)** section blank. You define the schema in **Spanner Studio** in the next section of this quickstart.

6.  Click **Create** .

    The Google Cloud console displays the **Overview** page for the database you created.

## Create a schema for your database

1.  In the navigation menu, click **Spanner Studio** .

2.  On the **Spanner Studio** page, click **New tab** or use the **Untitled query** tab, and then enter one of the following queries, depending on the dialect of your database:

    ### GoogleSQL

    ```
    CREATE TABLE Singers (
      SingerId   INT64 NOT NULL,
      FirstName  STRING(1024),
      LastName   STRING(1024),
      BirthDate  DATE
    ) PRIMARY KEY(SingerId);
    ```

    ### PostgreSQL

    ```
    CREATE TABLE Singers (
      SingerId   BIGINT PRIMARY KEY,
      FirstName  VARCHAR(1024),
      LastName   VARCHAR(1024),
      BirthDate  TIMESTAMPTZ
    );
    ```

3.  Click **Run** .

    Spanner runs the statement. When finished, the **Results** tab displays **Update completed** .

## Insert and modify data

The Google Cloud console provides an interface for inserting, editing, and deleting data.

### Insert data

1.  In the **Explorer** pane, click **View actions** for the **Singers** table, and then click **Insert data** .

    Spanner opens a new query tab in **Spanner Studio** that contains a template `INSERT` statement.

2.  Edit the template `INSERT` statement's `VALUES` clause to insert a row in the `Singers` table and view the result of that insertion:

    ### GoogleSQL

    ```
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      Singers (SingerId,
        FirstName,
        LastName,
        BirthDate)
    VALUES
      (1, -- type: INT64
        'Marc', -- type: STRING(1024)
        'Richards', -- type: STRING(1024)
        NULL -- type: DATE
        )
    THEN RETURN
      SingerId,
      FirstName,
      LastName,
      BirthDate;
    ```

    ### PostgreSQL

    ```
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        firstname,
        lastname,
        birthdate)
    VALUES
      (1, -- type: bigint
        'Marc', -- type: character varying
        'Richards', -- type: character varying
        NULL -- type: timestamp with time zone
        )
    RETURNING
      singerid,
      firstname,
      lastname,
      birthdate;
    ```

    Notice that PostgreSQL converts the column names to all lowercase.

3.  Click **Run** .

    Spanner runs the statement. When finished, the **Results** tab displays **1 row inserted** and the data of the inserted row.

4.  In the **Explorer** pane, click **View actions** for the **Singers** table, and then click **Insert data** .

5.  Edit the `INSERT` statement's `VALUES` clause:

    ### GoogleSQL

    ```
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      Singers (SingerId,
        FirstName,
        LastName,
        BirthDate)
    VALUES
      (2, -- type: INT64
        'Catalina', -- type: STRING(1024)
        'Smith', -- type: STRING(1024)
        NULL -- type: DATE
        )
    THEN RETURN
      SingerId,
      FirstName,
      LastName,
      BirthDate;
    ```

    ### PostgreSQL

    ```
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        firstname,
        lastname,
        birthdate)
    VALUES
      (2, -- type: bigint
        'Catalina', -- type: character varying
        'Smith', -- type: character varying
        NULL -- type: timestamp with time zone
        )
    RETURNING
      singerid,
      firstname,
      lastname,
      birthdate;
    ```

6.  Click **Run** .

    Spanner runs the statement. When execution completes, the **Results** tab displays **1 row inserted** and the data of the inserted row.

7.  In the **Explorer** pane, click **View actions** for the **Singers** table, and then click **Preview data** .

8.  Click **Run** .

    The **Results** tab displays the two rows in the `Singers` table.

You can also insert empty strings into string columns.

1.  In the **Explorer** pane, click **View actions** for the **Singers** table, and then click **Insert data** .

    Spanner opens a new query tab in **Spanner Studio** that contains the template `INSERT` statement.

2.  Edit the template `INSERT` statement's `VALUES` clause:

    ### GoogleSQL

    ```
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      Singers (SingerId,
        FirstName,
        LastName,
        BirthDate)
    VALUES
      (3, -- type: INT64
        'Kena', -- type: STRING(1024)
        '', -- type: STRING(1024)
        NULL -- type: DATE
        )
    THEN RETURN
      SingerId,
      FirstName,
      LastName,
      BirthDate;
    ```

    ### PostgreSQL

    ```
      -- Add new values in the VALUES clause in order of the column list.
      -- Each value must be type compatible with its associated column.
    INSERT INTO
      singers (singerid,
        firstname,
        lastname,
        birthdate)
    VALUES
      (3, -- type: bigint
        'Kena', -- type: character varying
        '', -- type: character varying
        NULL -- type: timestamp with time zone
        )
    RETURNING
      singerid,
      firstname,
      lastname,
      birthdate;
    ```

3.  Click **Run** .

    Spanner runs the statement. When execution completes, the **Results** tab displays **1 row inserted** and the data of the inserted row.

    Notice that the value provided for the `LastName` column is an empty string, `''` , not a `NULL` value.

4.  In the **Explorer** pane, click **View actions** for the **Singers** table, and then click **Preview data** .

5.  Click **Run** .

    The **Results** tab displays the three rows in the `Singers` table. The row with the primary key value of `3` contains an empty string in the `LastName` column.

### Edit data

1.  To update the birth date for the row with the primary key value of `3` , click **New tab** , and then enter the following `UPDATE` statement:

    ### GoogleSQL

    ```
    UPDATE
      Singers
    SET
      BirthDate='1961-04-01'
    WHERE
      SingerId=3
    THEN RETURN
      SingerId,
      FirstName,
      LastName,
      BirthDate;
    ```

    ### PostgreSQL

    ```
    UPDATE
      singers
    SET
      birthdate='1961-04-01 00:00:00 -8:00'
    WHERE
      singerid=3
    RETURNING
      singerid,
      firstname,
      lastname,
      birthdate;
    ```

2.  Click **Run** .

    Spanner runs the statement. When execution completes, the **Results** tab displays **1 row updated** and the data of the updated row.

3.  In the **Explorer** pane, click **View actions** for the **Singers** table, and then click **Preview data** .

4.  Click **Run** .

    The **Results** tab displays the rows in the `Singers` table, showing the updated birth date for singer `3` .

### Delete data

1.  To delete the row with the primary key value of `2` , click **New tab** , and then enter the following `DELETE` statement:

    ### GoogleSQL

    ```
    DELETE FROM Singers
    WHERE SingerId = 2;
    ```

    ### PostgreSQL

    ```
    DELETE FROM singers
    WHERE singerid = 2;
    ```

2.  Click **Run** .

    Spanner runs the statement. When execution completes, the **Results** tab displays **1 row deleted** . The `Singers` table contains two rows.

## Run a query in the Google Cloud console

1.  Click **New tab** to open a new query tab, and then enter the following query in the query editor:

    ### GoogleSQL

    ```
    SELECT * FROM Singers;
    ```

    ### PostgreSQL

    ```
    SELECT * FROM singers;
    ```

2.  Click **Run** .

    Spanner runs the query. When finished, the **Results** tab displays the result of your query.

Congratulations. You successfully created a new database, defined a schema, inserted data, and ran a query in the Spanner query editor.

## Clean up

To avoid incurring additional charges to your Google Cloud account, delete the database and instance. Disabling the Cloud Billing API doesn't stop charges. When you delete an instance, Spanner deletes all databases in that instance.

### Delete the database

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the name of the instance that has the database that you want to delete, for example, **Test Instance** .

3.  Click the name of the database that you want to delete, for example, **example-db** .

4.  On the database **Overview** page, click **Delete database** .

5.  Confirm that you want to delete the database by entering the database name and clicking **Delete** .

### Delete the instance

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Click the name of the instance that you want to delete, for example, **Test Instance** .

3.  Click **Delete instance** .

4.  Confirm that you want to delete the instance by entering the instance name and clicking **Delete** .

## What's next

- Learn about [instances](https://docs.cloud.google.com/spanner/docs/instances) .
- Understand the Spanner [schema and data model](https://docs.cloud.google.com/spanner/docs/schema-and-data-model) .
- Learn more about [GoogleSQL data definition language (DDL)](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language) .
- Learn more about [query execution plans](https://docs.cloud.google.com/spanner/docs/query-execution-plans) .
- Learn how to use Spanner with [C++](https://docs.cloud.google.com/spanner/docs/getting-started/cpp) , [C#](https://docs.cloud.google.com/spanner/docs/getting-started/csharp) , [Go](https://docs.cloud.google.com/spanner/docs/getting-started/go) , [Java](https://docs.cloud.google.com/spanner/docs/getting-started/java) , [Node.js](https://docs.cloud.google.com/spanner/docs/getting-started/nodejs) , [PHP](https://docs.cloud.google.com/spanner/docs/getting-started/php) , [Python](https://docs.cloud.google.com/spanner/docs/getting-started/python) , [Ruby](https://docs.cloud.google.com/spanner/docs/getting-started/ruby) , [REST](https://docs.cloud.google.com/spanner/docs/getting-started/rest) , or [the Google Cloud CLI](https://docs.cloud.google.com/spanner/docs/getting-started/gcloud) .
