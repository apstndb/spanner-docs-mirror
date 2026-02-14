This page describes how to connect to a Spanner database in IntelliJ and other JetBrains products.

[IntelliJ IDEA](https://www.jetbrains.com/idea/) is an integrated development environment for developing software in Java, Kotlin, Groovy, and other JVM-based languages.

This page assumes that you already have IntelliJ or another JetBrains IDE installed on your computer.

## Connect to Spanner

Connecting IntelliJ to your Spanner database lets you execute queries and other SQL statements on your Spanner database directly from IntelliJ. It also enables code completion and validation for table and column names in your code.

You can connect to both GoogleSQL-dialect databases and PostgreSQL-dialect databases by following these steps:

1.  In IntelliJ, click the menu option `  File > New > Datasource > Google Spanner  ` . The `  Google Spanner  ` driver is under the `  Other  ` sub-menu if you have not used this driver before.

2.  In the Data Sources window, enter your project, instance, and database ID.

3.  Optional: Select a service account key file if you want to authenticate using a service account. Select `  No Auth  ` in the Credentials drop-down if you want to use your default credentials.

4.  Click OK to create the data source. The Spanner database is added to the Database window in IntelliJ.

5.  In the Databases window, expand the data source that you just added. Then click the `  ...  ` button next to the text `  No schemas selected  ` .

6.  Select the option `  All schemas  ` to instruct IntelliJ to introspect all schemas in the database. IntelliJ will then populate the database view with all tables and views in your database.

## Connect to the Spanner Emulator

You can also connect IntelliJ to a database in the [Spanner Emulator](/spanner/docs/emulator) :

1.  First start the emulator with one of the following commands:
    
    1.  `  gcloud emulators spanner start  `
    2.  `  docker run -p 9010:9010 -p 9020:9020 gcr.io/cloud-spanner-emulator/emulator  `

2.  Click the menu option `  File > New > Datasource > Google Spanner  ` . The `  Google Spanner  ` driver is under the `  Other  ` sub-menu if you have not used this driver before.

3.  Enter the project, instance, and database ID. **NOTE** : The project, instance and database don't need to exist in the emulator.

4.  Select `  No Auth  ` in the Credentials drop-down.

5.  Click the `  Advanced  ` tab in the Data Sources window.

6.  Modify the value of `  autoConfigEmulator  ` to `  true  ` .

7.  Click OK to accept all changes. The project, instance, and database will be created on the emulator automatically if these don't exist already.

8.  In the Databases window, expand the data source that you just added. Click the `  ...  ` button next to the text `  No schemas selected  ` .

9.  Select the option `  All schemas  ` to instruct IntelliJ to introspect all schemas in the database. IntelliJ will then populate the database view with all tables and views in your database.

## What's next

  - For more documentation on how to add and work with data sources in IntelliJ, visit [IntelliJ Data sources](https://www.jetbrains.com/help/idea/managing-data-sources.html) .
  - Learn more about the [Spanner Emulator](/spanner/docs/emulator) .
