## Objectives

This tutorial walks you through the following steps using the Spanner client library for Node.js:

  - Create a Spanner instance and database.
  - Write, read, and execute SQL queries on data in the database.
  - Update the database schema.
  - Update data using a read-write transaction.
  - Add a secondary index to the database.
  - Use the index to read and execute SQL queries on data.
  - Retrieve data using a read-only transaction.

## Costs

This tutorial uses Spanner, which is a billable component of the Google Cloud. For information on the cost of using Spanner, see [Pricing](https://cloud.google.com/spanner/pricing) .

## Before you begin

Complete the steps described in [Set up](/spanner/docs/getting-started/set-up#set_up_a_project) , which cover creating and setting a default Google Cloud project, enabling billing, enabling the Cloud Spanner API, and setting up OAuth 2.0 to get authentication credentials to use the Cloud Spanner API.

In particular, make sure that you run [`  gcloud auth application-default login  `](/sdk/gcloud/reference/auth/application-default/login) to set up your local development environment with authentication credentials.

**Note:** If you don't plan to keep the resources that you create in this tutorial, consider creating a new Google Cloud project instead of selecting an existing project. After you finish the tutorial, you can delete the project, removing all resources associated with the project.

## Prepare your local Node.js environment

1.  Follow the steps to [Set Up a Node.js Development Environment](/nodejs/docs/setup)

2.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/googleapis/nodejs-spanner
    ```
    
    Alternatively, you can [download the sample](https://github.com/googleapis/nodejs-spanner/archive/main.zip) as a zip file and extract it.

3.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd samples/
    ```

4.  Install dependencies using `  npm  ` :
    
    ``` text
    npm install
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with Node.js.

Take a look through the `  samples/schema.js  ` file, which shows how to create a database and modify a database schema. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

``` text
node schema.js createDatabase test-instance example-db MY_PROJECT_ID
```

### PostgreSQL

``` text
node schema.js createPgDatabase test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
Created database example-db on instance test-instance.
```

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectID,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

const createSingersTableStatement = `
CREATE TABLE Singers (
  SingerId    INT64 NOT NULL,
  FirstName   STRING(1024),
  LastName    STRING(1024),
  SingerInfo  BYTES(MAX),
  FullName    STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED,
) PRIMARY KEY (SingerId)`;
const createAlbumsTableStatement = `
CREATE TABLE Albums (
  SingerId    INT64 NOT NULL,
  AlbumId     INT64 NOT NULL,
  AlbumTitle  STRING(MAX)
) PRIMARY KEY (SingerId, AlbumId),
INTERLEAVE IN PARENT Singers ON DELETE CASCADE`;

// Creates a new database
try {
  const [operation] = await databaseAdminClient.createDatabase({
    createStatement: 'CREATE DATABASE `' + databaseID + '`',
    extraStatements: [
      createSingersTableStatement,
      createAlbumsTableStatement,
    ],
    parent: databaseAdminClient.instancePath(projectID, instanceID),
  });

  console.log(`Waiting for creation of ${databaseID} to complete...`);
  await operation.promise();

  console.log(`Created database ${databaseID} on instance ${instanceID}.`);
} catch (err) {
  console.error('ERROR:', err);
}
```

### PostgreSQL

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner, protos} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function createPgDatabase() {
  // Creates a PostgreSQL database. PostgreSQL create requests may not contain any additional
  // DDL statements. We need to execute these separately after the database has been created.
  const [operationCreate] = await databaseAdminClient.createDatabase({
    createStatement: 'CREATE DATABASE "' + databaseId + '"',
    parent: databaseAdminClient.instancePath(projectId, instanceId),
    databaseDialect:
      protos.google.spanner.admin.database.v1.DatabaseDialect.POSTGRESQL,
  });

  console.log(`Waiting for operation on ${databaseId} to complete...`);
  await operationCreate.promise();
  const [metadata] = await databaseAdminClient.getDatabase({
    name: databaseAdminClient.databasePath(projectId, instanceId, databaseId),
  });
  console.log(
    `Created database ${databaseId} on instance ${instanceId} with dialect ${metadata.databaseDialect}.`,
  );

  // Create a couple of tables using a separate request. We must use PostgreSQL style DDL as the
  // database has been created with the PostgreSQL dialect.
  const statements = [
    `CREATE TABLE Singers 
      (SingerId   bigint NOT NULL,
      FirstName   varchar(1024),
      LastName    varchar(1024),
      SingerInfo  bytea,
      FullName    character varying(2048) GENERATED ALWAYS AS (FirstName || ' ' || LastName) STORED,
      PRIMARY KEY (SingerId)
      );
      CREATE TABLE Albums 
      (AlbumId    bigint NOT NULL,
      SingerId    bigint NOT NULL REFERENCES Singers (SingerId),
      AlbumTitle  text,
      PRIMARY KEY (AlbumId)
      );`,
  ];
  const [operationUpdateDDL] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: [statements],
  });
  await operationUpdateDDL.promise();
  console.log('Updated schema');
}
createPgDatabase();
```

The next step is to write data to your database.

## Create a database client

Before you can do reads or writes, you must create a [`  Database  `](https://googleapis.dev/nodejs/spanner/latest/Database.html) :

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({projectId});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// The query to execute
const query = {
  sql: 'SELECT 1',
};

// Execute a simple SQL statement
const [rows] = await database.run(query);
console.log(`Query: ${rows.length} found.`);
rows.forEach(row => console.log(row));
```

You can think of a `  Database  ` as a database connection: all of your interactions with Spanner must go through a `  Database  ` . Typically you create a `  Database  ` when your application starts up, then you re-use that `  Database  ` to read, write, and execute transactions. Each client uses resources in Spanner.

If you create multiple clients in the same app, you should call [`  Database.close()  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#close) to clean up the client's resources, including network connections, as soon as it is no longer needed.

Read more in the [`  Database  `](https://googleapis.dev/nodejs/spanner/latest/Database.html) reference.

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  runUpdate()  ` method to execute a DML statement.

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

database.runTransaction(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  try {
    const [rowCount] = await transaction.runUpdate({
      sql: `INSERT Singers (SingerId, FirstName, LastName) VALUES
      (12, 'Melissa', 'Garcia'),
      (13, 'Russell', 'Morales'),
      (14, 'Jacqueline', 'Long'),
      (15, 'Dylan', 'Shaw')`,
    });
    console.log(`${rowCount} records inserted.`);
    await transaction.commit();
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
});
```

Run the sample using the `  writeUsingDml  ` argument.

``` text
node dml.js writeUsingDml test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
4 records inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

You write data using a [`  Table  `](https://googleapis.dev/nodejs/spanner/latest/Table.html) object. The [`  Table.insert()  `](https://googleapis.dev/nodejs/spanner/latest/Table.html#insert) method adds new rows to the table. All inserts in a single batch are applied atomically.

This code shows how to write the data using mutations:

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Instantiate Spanner table objects
const singersTable = database.table('Singers');
const albumsTable = database.table('Albums');

// Inserts rows into the Singers table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so
// they must be converted to strings before being inserted as INT64s
try {
  await singersTable.insert([
    {SingerId: '1', FirstName: 'Marc', LastName: 'Richards'},
    {SingerId: '2', FirstName: 'Catalina', LastName: 'Smith'},
    {SingerId: '3', FirstName: 'Alice', LastName: 'Trentor'},
    {SingerId: '4', FirstName: 'Lea', LastName: 'Martin'},
    {SingerId: '5', FirstName: 'David', LastName: 'Lomond'},
  ]);

  await albumsTable.insert([
    {SingerId: '1', AlbumId: '1', AlbumTitle: 'Total Junk'},
    {SingerId: '1', AlbumId: '2', AlbumTitle: 'Go, Go, Go'},
    {SingerId: '2', AlbumId: '1', AlbumTitle: 'Green'},
    {SingerId: '2', AlbumId: '2', AlbumTitle: 'Forever Hold your Peace'},
    {SingerId: '2', AlbumId: '3', AlbumTitle: 'Terrified'},
  ]);

  console.log('Inserted data.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  await database.close();
}
```

Run the sample using the `  insert  ` argument.

``` text
node crud.js insert test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
Inserted data.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Node.js.

### On the command line

Execute the following SQL statement to read the values of all columns from the `  Albums  ` table:

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
```

**Note:** For the GoogleSQL reference, see [Query syntax in GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax) and for PostgreSQL reference, see [PostgreSQL lexical structure and syntax](/spanner/docs/reference/postgresql/lexical) .

The result shows:

``` text
SingerId AlbumId AlbumTitle
1        1       Total Junk
1        2       Go, Go, Go
2        1       Green
2        2       Forever Hold Your Peace
2        3       Terrified
```

### Use the Spanner client library for Node.js

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner client library for Node.js.

Use [`  Database.run()  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#run) to run the SQL query.

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const query = {
  sql: 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums',
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  await database.close();
}
```

Here's how to issue the query and access the data:

``` text
node crud.js query test-instance example-db MY_PROJECT_ID
```

You should see the following result:

``` text
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold your Peace
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
```

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `  WHERE  ` clause to query records containing a specific value for `  LastName  ` .

### GoogleSQL

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const query = {
  sql: `SELECT SingerId, FirstName, LastName
        FROM Singers WHERE LastName = @lastName`,
  params: {
    lastName: 'Garcia',
  },
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `SingerId: ${json.SingerId}, FirstName: ${json.FirstName}, LastName: ${json.LastName}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PostgreSQL

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud Spanner client library
const {Spanner} = require('@google-cloud/spanner');

// Instantiates a client
const spanner = new Spanner({
  projectId: projectId,
});

async function queryWithPgParameter() {
  // Gets a reference to a Cloud Spanner instance and database.
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  const fieldType = {
    type: 'string',
  };

  const query = {
    sql: `SELECT singerid, firstname, lastname
          FROM singers
          WHERE firstname LIKE $1`,
    params: {
      p1: 'A%',
    },
    types: {
      p1: fieldType,
    },
  };

  // Queries rows from the Singers table.
  try {
    const [rows] = await database.run(query);

    rows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.singerid}, FirstName: ${json.firstname}, LastName: ${json.lastname}`,
      );
    });
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    database.close();
  }
}
queryWithPgParameter();
```

Here's how to issue the query and access the data:

``` text
node dml.js queryWithParameter test-instance example-db MY_PROJECT_ID
```

You should see the following result:

``` text
SingerId: 12, FirstName: Melissa, LastName: Garcia
```

## Read data using the read API

In addition to Spanner's SQL interface, Spanner also supports a read interface.

Use [`  Table.read()  `](https://googleapis.dev/nodejs/spanner/latest/Table.html#read) to read rows from the database. Use a `  KeySet  ` object to define a collection of keys and key ranges to read.

Here's how to read the data:

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Reads rows from the Albums table
const albumsTable = database.table('Albums');

const query = {
  columns: ['SingerId', 'AlbumId', 'AlbumTitle'],
  keySet: {
    all: true,
  },
};

try {
  const [rows] = await albumsTable.read(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(
      `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  await database.close();
}
```

Run the sample using the `  read  ` argument.

``` text
node crud.js read test-instance example-db MY_PROJECT_ID
```

You should see output similar to:

``` text
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold your Peace
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Node.js.

#### On the command line

Use the following [`  ALTER TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) command to add the new column to the table:

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget BIGINT'
```

You should see:

``` text
Schema updating...done.
```

#### Use the Spanner client library for Node.js

Use [`  Database.updateSchema  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#updateSchema) to modify the schema:

### GoogleSQL

``` javascript
/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

// Creates a new index in the database
try {
  const [operation] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: ['ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'],
  });

  console.log('Waiting for operation to complete...');
  await operation.promise();

  console.log('Added the MarketingBudget column.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the spanner client when finished.
  // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
  spanner.close();
}
```

### PostgreSQL

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function pgAddColumn() {
  const request = ['ALTER TABLE Albums ADD COLUMN MarketingBudget BIGINT'];

  // Alter existing table to add a column.
  const [operation] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: request,
  });

  console.log(`Waiting for operation on ${databaseId} to complete...`);

  await operation.promise();

  console.log(
    `Added MarketingBudget column to Albums table in database ${databaseId}.`,
  );
}
pgAddColumn();
```

Run the sample using the `  addColumn  ` argument.

``` text
node schema.js addColumn test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
Added the MarketingBudget column.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Update a row in the Albums table
// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
// must be converted to strings before being inserted as INT64s
const albumsTable = database.table('Albums');

try {
  await albumsTable.update([
    {SingerId: '1', AlbumId: '1', MarketingBudget: '100000'},
    {SingerId: '2', AlbumId: '2', MarketingBudget: '500000'},
  ]);
  console.log('Updated data.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

Run the sample using the `  update  ` argument.

``` text
node crud.js update test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
Updated data.
```

You can also execute a SQL query or a read call to fetch the values that you just wrote.

Here's the code to execute the query:

``` javascript
// This sample uses the `MarketingBudget` column. You can add the column
// by running the `add_column` sample or by running this DDL statement against
// your database:
//    ALTER TABLE Albums ADD COLUMN MarketingBudget INT64

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const query = {
  sql: 'SELECT SingerId, AlbumId, MarketingBudget FROM Albums',
};

// Queries rows from the Albums table
try {
  const [rows] = await database.run(query);

  rows.forEach(async row => {
    const json = row.toJSON();

    console.log(
      `SingerId: ${json.SingerId}, AlbumId: ${
        json.AlbumId
      }, MarketingBudget: ${
        json.MarketingBudget ? json.MarketingBudget : null
      }`,
    );
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

To execute this query, run the sample using the `  queryNewColumn  ` argument.

``` text
node schema.js queryNewColumn test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
SingerId: 1, AlbumId: 1, MarketingBudget: 100000
SingerId: 1, AlbumId: 2, MarketingBudget: null
SingerId: 2, AlbumId: 1, MarketingBudget: null
SingerId: 2, AlbumId: 2, MarketingBudget: 500000
SingerId: 2, AlbumId: 3, MarketingBudget: null
```

## Update data

You can update data using DML in a read-write transaction.

You use the `  runUpdate()  ` method to execute a DML statement.

### GoogleSQL

``` javascript
// This sample transfers 200,000 from the MarketingBudget field
// of the second Album to the first Album, as long as the second
// Album has enough money in its budget. Make sure to run the
// addColumn and updateData samples first (in that order).

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const transferAmount = 200000;

database.runTransaction((err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  let firstBudget, secondBudget;
  const queryOne = `SELECT MarketingBudget FROM Albums
    WHERE SingerId = 2 AND AlbumId = 2`;

  const queryTwo = `SELECT MarketingBudget FROM Albums
  WHERE SingerId = 1 AND AlbumId = 1`;

  Promise.all([
    // Reads the second album's budget
    transaction.run(queryOne).then(results => {
      // Gets second album's budget
      const rows = results[0].map(row => row.toJSON());
      secondBudget = rows[0].MarketingBudget;
      console.log(`The second album's marketing budget: ${secondBudget}`);

      // Makes sure the second album's budget is large enough
      if (secondBudget < transferAmount) {
        throw new Error(
          `The second album's budget (${secondBudget}) is less than the transfer amount (${transferAmount}).`,
        );
      }
    }),

    // Reads the first album's budget
    transaction.run(queryTwo).then(results => {
      // Gets first album's budget
      const rows = results[0].map(row => row.toJSON());
      firstBudget = rows[0].MarketingBudget;
      console.log(`The first album's marketing budget: ${firstBudget}`);
    }),
  ])
    .then(() => {
      // Transfers the budgets between the albums
      console.log(firstBudget, secondBudget);
      firstBudget += transferAmount;
      secondBudget -= transferAmount;

      console.log(firstBudget, secondBudget);

      // Updates the database
      // Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
      // must be converted (back) to strings before being inserted as INT64s.

      return transaction
        .runUpdate({
          sql: `UPDATE Albums SET MarketingBudget = @Budget
              WHERE SingerId = 1 and AlbumId = 1`,
          params: {
            Budget: firstBudget,
          },
        })
        .then(() =>
          transaction.runUpdate({
            sql: `UPDATE Albums SET MarketingBudget = @Budget
                  WHERE SingerId = 2 and AlbumId = 2`,
            params: {
              Budget: secondBudget,
            },
          }),
        );
    })
    .then(() => {
      // Commits the transaction and send the changes to the database
      return transaction.commit();
    })
    .then(() => {
      console.log(
        `Successfully executed read-write transaction using DML to transfer ${transferAmount} from Album 2 to Album 1.`,
      );
    })
    .then(() => {
      // Closes the database when finished
      database.close();
    });
});
```

### PostgreSQL

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud Spanner client library
const {Spanner} = require('@google-cloud/spanner');

// Instantiates a client
const spanner = new Spanner({
  projectId: projectId,
});

function updateUsingDml() {
  // Gets a reference to a Cloud Spanner instance and database
  const instance = spanner.instance(instanceId);
  const database = instance.database(databaseId);

  database.runTransaction(async (err, transaction) => {
    if (err) {
      console.error(err);
      return;
    }
    try {
      const [rowCount] = await transaction.runUpdate({
        sql: 'UPDATE singers SET FirstName = $1 WHERE singerid = 1',
        params: {
          p1: 'Virginia',
        },
      });

      console.log(
        `Successfully updated ${rowCount} record in the Singers table.`,
      );

      await transaction.commit();
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the database when finished.
      await database.close();
    }
  });
}
updateUsingDml();
```

Run the sample using the `  writeWithTransactionUsingDml  ` argument.

``` text
node dml.js writeWithTransactionUsingDml test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
Successfully executed read-write transaction using DML to transfer $200000 from Album 2 to Album 1.
```

**Note:** You can also [update data using mutations](/spanner/docs/modify-mutation-api#updating_rows_in_a_table) .

## Use a secondary index

Suppose you wanted to fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range. You could read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that don't meet the criteria, but doing this full table scan is expensive, especially for tables with a lot of rows. Instead you can speed up the retrieval of rows when searching by non-primary key columns by creating a [secondary index](/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Like other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to the indexed table during this process. For more details, see [Add a secondary index](/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that are likely to run faster with the index. If you use the read interface, you must specify the index that you want to use.

### Add a secondary index

You can add an index on the command line using the gcloud CLI or programmatically using the Spanner client library for Node.js.

#### On the command line

Use the following [`  CREATE INDEX  `](/spanner/docs/reference/standard-sql/data-definition-language#create_index) command to add an index to the database:

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)'
```

You should see:

``` text
Schema updating...done.
```

#### Using the Spanner client library for Node.js

Use [`  Database.updateSchema()  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#updateSchema) to add an index:

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

const request = ['CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)'];

// Creates a new index in the database
try {
  const [operation] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: request,
  });

  console.log('Waiting for operation to complete...');
  await operation.promise();

  console.log('Added the AlbumsByAlbumTitle index.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the spanner client when finished.
  // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
  spanner.close();
}
```

Run the sample using the `  createIndex  ` argument.

``` text
node indexing.js createIndex test-instance example-db MY_PROJECT_ID
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Added the AlbumsByAlbumTitle index.
```

### Read using the index

For SQL queries, Spanner automatically uses an appropriate index. In the read interface, you must specify the index in your request.

To use the index in the read interface, use the [`  Table.read()  `](https://googleapis.dev/nodejs/spanner/latest/Table.html#read) method.

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const albumsTable = database.table('Albums');

const query = {
  columns: ['AlbumId', 'AlbumTitle'],
  keySet: {
    all: true,
  },
  index: 'AlbumsByAlbumTitle',
};

// Reads the Albums table using an index
try {
  const [rows] = await albumsTable.read(query);

  rows.forEach(row => {
    const json = row.toJSON();
    console.log(`AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`);
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

Run the sample using the `  readIndex  ` argument.

``` text
node indexing.js readIndex test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
AlbumId: 2, AlbumTitle: Forever Hold your Peace
AlbumId: 2, AlbumTitle: Go, Go, Go
AlbumId: 1, AlbumTitle: Green
AlbumId: 3, AlbumTitle: Terrified
AlbumId: 1, AlbumTitle: Total Junk
```

### Add an index for index-only reads

You might have noticed that the previous read example doesn't include reading the `  MarketingBudget  ` column. This is because Spanner's read interface doesn't support the ability to join an index with a data table to look up values that are not stored in the index.

Create an alternate definition of `  AlbumsByAlbumTitle  ` that stores a copy of `  MarketingBudget  ` in the index.

#### On the command line

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) INCLUDE (MarketingBudget)
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Schema updating...done.
```

#### Using the Spanner client library for Node.js

Use [`  Database.updateSchema()  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#updateSchema) to add an index with a `  STORING  ` clause:

``` javascript
// "Storing" indexes store copies of the columns they index
// This speeds up queries, but takes more space compared to normal indexes
// See the link below for more information:
// https://cloud.google.com/spanner/docs/secondary-indexes#storing_clause

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

const request = [
  'CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)',
];

// Creates a new index in the database
try {
  const [operation] = await databaseAdminClient.updateDatabaseDdl({
    database: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    statements: request,
  });

  console.log('Waiting for operation to complete...');
  await operation.promise();

  console.log('Added the AlbumsByAlbumTitle2 index.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the spanner client when finished.
  // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
  spanner.close();
}
```

Run the sample using the `  createStoringIndex  ` argument.

``` text
node indexing.js createStoringIndex test-instance example-db MY_PROJECT_ID
```

You should see:

``` text
Added the AlbumsByAlbumTitle2 index.
```

Now you can execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

``` javascript
// "Storing" indexes store copies of the columns they index
// This speeds up queries, but takes more space compared to normal indexes
// See the link below for more information:
// https://cloud.google.com/spanner/docs/secondary-indexes#storing_clause

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

const albumsTable = database.table('Albums');

const query = {
  columns: ['AlbumId', 'AlbumTitle', 'MarketingBudget'],
  keySet: {
    all: true,
  },
  index: 'AlbumsByAlbumTitle2',
};

// Reads the Albums table using a storing index
try {
  const [rows] = await albumsTable.read(query);

  rows.forEach(row => {
    const json = row.toJSON();
    let rowString = `AlbumId: ${json.AlbumId}`;
    rowString += `, AlbumTitle: ${json.AlbumTitle}`;
    if (json.MarketingBudget) {
      rowString += `, MarketingBudget: ${json.MarketingBudget}`;
    }
    console.log(rowString);
  });
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

Run the sample using the `  readStoringIndex  ` argument.

``` text
node indexing.js readStoringIndex test-instance example-db MY_PROJECT_ID
```

You should see output similar to:

``` text
AlbumId: 2, AlbumTitle: Forever Hold your Peace, MarketingBudget: 300000
AlbumId: 2, AlbumTitle: Go, Go, Go, MarketingBudget: null
AlbumId: 1, AlbumTitle: Green, MarketingBudget: null
AlbumId: 3, AlbumTitle: Terrified, MarketingBudget: null
AlbumId: 1, AlbumTitle: Total Junk, MarketingBudget: 300000
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data.Use [`  Database.runTransaction()  `](https://googleapis.dev/nodejs/spanner/latest/Database.html#runTransaction) for executing read-only transactions.

The following shows how to run a query and perform a read in the same read-only transaction:

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Gets a transaction object that captures the database state
// at a specific point in time
database.getSnapshot(async (err, transaction) => {
  if (err) {
    console.error(err);
    return;
  }
  const queryOne = 'SELECT SingerId, AlbumId, AlbumTitle FROM Albums';

  try {
    // Read #1, using SQL
    const [qOneRows] = await transaction.run(queryOne);

    qOneRows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
      );
    });

    const queryTwo = {
      columns: ['SingerId', 'AlbumId', 'AlbumTitle'],
    };

    // Read #2, using the `read` method. Even if changes occur
    // in-between the reads, the transaction ensures that both
    // return the same data.
    const [qTwoRows] = await transaction.read('Albums', queryTwo);

    qTwoRows.forEach(row => {
      const json = row.toJSON();
      console.log(
        `SingerId: ${json.SingerId}, AlbumId: ${json.AlbumId}, AlbumTitle: ${json.AlbumTitle}`,
      );
    });

    console.log('Successfully executed read-only transaction.');
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    transaction.end();
    // Close the database when finished.
    await database.close();
  }
});
```

Run the sample using the `  readOnly  ` argument.

``` text
node transaction.js readOnly test-instance example-db MY_PROJECT_ID
```

You should see output similar to:

``` text
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold your Peace
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold your Peace
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
Successfully executed read-only transaction.
```

## Cleanup

To avoid incurring additional charges to your Cloud Billing account for the resources used in this tutorial, drop the database and delete the instance that you created.

### Delete the database

If you delete an instance, all databases within it are automatically deleted. This step shows how to delete a database without deleting an instance (you would still incur charges for the instance).

#### On the command line

``` text
gcloud spanner databases delete example-db --instance=test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click the instance.

3.  Click the database that you want to delete.

4.  In the **Database details** page, click **Delete** .

5.  Confirm that you want to delete the database and click **Delete** .

### Delete the instance

Deleting an instance automatically drops all databases created in that instance.

#### On the command line

``` text
gcloud spanner instances delete test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click your instance.

3.  Click **Delete** .

4.  Confirm that you want to delete the instance and click **Delete** .

## What's next

  - Learn how to [access Spanner with a virtual machine instance](/spanner/docs/configure-virtual-machine-instance) .

  - Learn about authorization and authentication credentials in [Authenticate to Cloud services using client libraries](/docs/authentication/getting-started) .

  - Learn more about Spanner [Schema design best practices](/spanner/docs/schema-design) .
