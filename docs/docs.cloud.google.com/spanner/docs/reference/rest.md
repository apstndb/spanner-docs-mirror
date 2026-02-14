Cloud Spanner is a managed, mission-critical, globally consistent and scalable relational database service.

  - [REST Resource: v1.projects.instanceConfigOperations](#v1.projects.instanceConfigOperations)
  - [REST Resource: v1.projects.instanceConfigs](#v1.projects.instanceConfigs)
  - [REST Resource: v1.projects.instanceConfigs.operations](#v1.projects.instanceConfigs.operations)
  - [REST Resource: v1.projects.instances](#v1.projects.instances)
  - [REST Resource: v1.projects.instances.backupOperations](#v1.projects.instances.backupOperations)
  - [REST Resource: v1.projects.instances.backups](#v1.projects.instances.backups)
  - [REST Resource: v1.projects.instances.backups.operations](#v1.projects.instances.backups.operations)
  - [REST Resource: v1.projects.instances.databaseOperations](#v1.projects.instances.databaseOperations)
  - [REST Resource: v1.projects.instances.databases](#v1.projects.instances.databases)
  - [REST Resource: v1.projects.instances.databases.backupSchedules](#v1.projects.instances.databases.backupSchedules)
  - [REST Resource: v1.projects.instances.databases.databaseRoles](#v1.projects.instances.databases.databaseRoles)
  - [REST Resource: v1.projects.instances.databases.operations](#v1.projects.instances.databases.operations)
  - [REST Resource: v1.projects.instances.databases.sessions](#v1.projects.instances.databases.sessions)
  - [REST Resource: v1.projects.instances.instancePartitionOperations](#v1.projects.instances.instancePartitionOperations)
  - [REST Resource: v1.projects.instances.instancePartitions](#v1.projects.instances.instancePartitions)
  - [REST Resource: v1.projects.instances.instancePartitions.operations](#v1.projects.instances.instancePartitions.operations)
  - [REST Resource: v1.projects.instances.operations](#v1.projects.instances.operations)

## Service: spanner.googleapis.com

To call this service, we recommend that you use the Google-provided [client libraries](https://cloud.google.com/apis/docs/client-libraries-explained) . If your application needs to use your own libraries to call this service, use the following information when you make the API requests.

### Discovery document

A [Discovery Document](https://developers.google.com/discovery/v1/reference/apis) is a machine-readable specification for describing and consuming REST APIs. It is used to build client libraries, IDE plugins, and other tools that interact with Google APIs. One service may provide multiple discovery documents. This service provides the following discovery document:

  - <https://spanner.googleapis.com/$discovery/rest?version=v1>

### Service endpoint

A [service endpoint](https://cloud.google.com/apis/design/glossary#api_service_endpoint) is a base URL that specifies the network address of an API service. One service might have multiple service endpoints. This service has the following service endpoint and all URIs below are relative to this service endpoint:

  - `  https://spanner.googleapis.com  `

### Regional service endpoint

A regional service endpoint is a base URL that specifies the network address of an API service in a single region. A service that is available in multiple regions might have multiple regional endpoints. Select a location to see its regional service endpoint for this service.

  

`  https://spanner.googleapis.com  `

## REST Resource: [v1.projects.instanceConfigOperations](/spanner/docs/reference/rest/v1/projects.instanceConfigOperations)

Methods

`  list  `

`  GET /v1/{parent=projects/*}/instanceConfigOperations  `  
Lists the user-managed instance configuration long-running operations in the given project.

## REST Resource: [v1.projects.instanceConfigs](/spanner/docs/reference/rest/v1/projects.instanceConfigs)

Methods

`  create  `

`  POST /v1/{parent=projects/*}/instanceConfigs  `  
Creates an instance configuration and begins preparing it to be used.

`  delete  `

`  DELETE /v1/{name=projects/*/instanceConfigs/*}  `  
Deletes the instance configuration.

`  get  `

`  GET /v1/{name=projects/*/instanceConfigs/*}  `  
Gets information about a particular instance configuration.

`  list  `

`  GET /v1/{parent=projects/*}/instanceConfigs  `  
Lists the supported instance configurations for a given project.

`  patch  `

`  PATCH /v1/{instanceConfig.name=projects/*/instanceConfigs/*}  `  
Updates an instance configuration.

## REST Resource: [v1.projects.instanceConfigs.operations](/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations)

Methods

`  cancel  `

`  POST /v1/{name=projects/*/instanceConfigs/*/operations/*}:cancel  `  
Starts asynchronous cancellation on a long-running operation.

`  delete  `

`  DELETE /v1/{name=projects/*/instanceConfigs/*/operations/*}  `  
Deletes a long-running operation.

`  get  `

`  GET /v1/{name=projects/*/instanceConfigs/*/operations/*}  `  
Gets the latest state of a long-running operation.

`  list  `

`  GET /v1/{name=projects/*/instanceConfigs/*/operations}  `  
Lists operations that match the specified filter in the request.

## REST Resource: [v1.projects.instances](/spanner/docs/reference/rest/v1/projects.instances)

Methods

`  create  `

`  POST /v1/{parent=projects/*}/instances  `  
Creates an instance and begins preparing it to begin serving.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*}  `  
Deletes an instance.

`  get  `

`  GET /v1/{name=projects/*/instances/*}  `  
Gets information about a particular instance.

`  getIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*}:getIamPolicy  `  
Gets the access control policy for an instance resource.

`  list  `

`  GET /v1/{parent=projects/*}/instances  `  
Lists all instances in the given project.

`  move  `

`  POST /v1/{name=projects/*/instances/*}:move  `  
Moves an instance to the target instance configuration.

`  patch  `

`  PATCH /v1/{instance.name=projects/*/instances/*}  `  
Updates an instance, and begins allocating or releasing resources as requested.

`  setIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*}:setIamPolicy  `  
Sets the access control policy on an instance resource.

`  testIamPermissions  `

`  POST /v1/{resource=projects/*/instances/*}:testIamPermissions  `  
Returns permissions that the caller has on the specified instance resource.

## REST Resource: [v1.projects.instances.backupOperations](/spanner/docs/reference/rest/v1/projects.instances.backupOperations)

Methods

`  list  `

`  GET /v1/{parent=projects/*/instances/*}/backupOperations  `  
Lists the backup long-running operations in the given instance.

## REST Resource: [v1.projects.instances.backups](/spanner/docs/reference/rest/v1/projects.instances.backups)

Methods

`  copy  `

`  POST /v1/{parent=projects/*/instances/*}/backups:copy  `  
Starts copying a Cloud Spanner Backup.

`  create  `

`  POST /v1/{parent=projects/*/instances/*}/backups  `  
Starts creating a new Cloud Spanner Backup.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/backups/*}  `  
Deletes a pending or completed `  Backup  ` .

`  get  `

`  GET /v1/{name=projects/*/instances/*/backups/*}  `  
Gets metadata on a pending or completed `  Backup  ` .

`  getIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*/backups/*}:getIamPolicy  `  
Gets the access control policy for a database or backup resource.

`  list  `

`  GET /v1/{parent=projects/*/instances/*}/backups  `  
Lists completed and pending backups.

`  patch  `

`  PATCH /v1/{backup.name=projects/*/instances/*/backups/*}  `  
Updates a pending or completed `  Backup  ` .

`  setIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*/backups/*}:setIamPolicy  `  
Sets the access control policy on a database or backup resource.

`  testIamPermissions  `

`  POST /v1/{resource=projects/*/instances/*/backups/*}:testIamPermissions  `  
Returns permissions that the caller has on the specified database or backup resource.

## REST Resource: [v1.projects.instances.backups.operations](/spanner/docs/reference/rest/v1/projects.instances.backups.operations)

Methods

`  cancel  `

`  POST /v1/{name=projects/*/instances/*/backups/*/operations/*}:cancel  `  
Starts asynchronous cancellation on a long-running operation.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/backups/*/operations/*}  `  
Deletes a long-running operation.

`  get  `

`  GET /v1/{name=projects/*/instances/*/backups/*/operations/*}  `  
Gets the latest state of a long-running operation.

`  list  `

`  GET /v1/{name=projects/*/instances/*/backups/*/operations}  `  
Lists operations that match the specified filter in the request.

## REST Resource: [v1.projects.instances.databaseOperations](/spanner/docs/reference/rest/v1/projects.instances.databaseOperations)

Methods

`  list  `

`  GET /v1/{parent=projects/*/instances/*}/databaseOperations  `  
Lists database longrunning-operations.

## REST Resource: [v1.projects.instances.databases](/spanner/docs/reference/rest/v1/projects.instances.databases)

Methods

`  addSplitPoints  `

`  POST /v1/{database=projects/*/instances/*/databases/*}:addSplitPoints  `  
Adds split points to specified tables and indexes of a database.

`  changequorum  `

`  POST /v1/{name=projects/*/instances/*/databases/*}:changequorum  `  
`  ChangeQuorum  ` is strictly restricted to databases that use dual-region instance configurations.

`  create  `

`  POST /v1/{parent=projects/*/instances/*}/databases  `  
Creates a new Spanner database and starts to prepare it for serving.

`  dropDatabase  `

`  DELETE /v1/{database=projects/*/instances/*/databases/*}  `  
Drops (aka deletes) a Cloud Spanner database.

`  get  `

`  GET /v1/{name=projects/*/instances/*/databases/*}  `  
Gets the state of a Cloud Spanner database.

`  getDdl  `

`  GET /v1/{database=projects/*/instances/*/databases/*}/ddl  `  
Returns the schema of a Cloud Spanner database as a list of formatted DDL statements.

`  getIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*/databases/*}:getIamPolicy  `  
Gets the access control policy for a database or backup resource.

`  list  `

`  GET /v1/{parent=projects/*/instances/*}/databases  `  
Lists Cloud Spanner databases.

`  patch  `

`  PATCH /v1/{database.name=projects/*/instances/*/databases/*}  `  
Updates a Cloud Spanner database.

`  restore  `

`  POST /v1/{parent=projects/*/instances/*}/databases:restore  `  
Create a new database by restoring from a completed backup.

`  setIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*/databases/*}:setIamPolicy  `  
Sets the access control policy on a database or backup resource.

`  testIamPermissions  `

`  POST /v1/{resource=projects/*/instances/*/databases/*}:testIamPermissions  `  
Returns permissions that the caller has on the specified database or backup resource.

`  updateDdl  `

`  PATCH /v1/{database=projects/*/instances/*/databases/*}/ddl  `  
Updates the schema of a Cloud Spanner database by creating/altering/dropping tables, columns, indexes, etc.

## REST Resource: [v1.projects.instances.databases.backupSchedules](/spanner/docs/reference/rest/v1/projects.instances.databases.backupSchedules)

Methods

`  create  `

`  POST /v1/{parent=projects/*/instances/*/databases/*}/backupSchedules  `  
Creates a new backup schedule.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/databases/*/backupSchedules/*}  `  
Deletes a backup schedule.

`  get  `

`  GET /v1/{name=projects/*/instances/*/databases/*/backupSchedules/*}  `  
Gets backup schedule for the input schedule name.

`  getIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*/databases/*/backupSchedules/*}:getIamPolicy  `  
Gets the access control policy for a database or backup resource.

`  list  `

`  GET /v1/{parent=projects/*/instances/*/databases/*}/backupSchedules  `  
Lists all the backup schedules for the database.

`  patch  `

`  PATCH /v1/{backupSchedule.name=projects/*/instances/*/databases/*/backupSchedules/*}  `  
Updates a backup schedule.

`  setIamPolicy  `

`  POST /v1/{resource=projects/*/instances/*/databases/*/backupSchedules/*}:setIamPolicy  `  
Sets the access control policy on a database or backup resource.

`  testIamPermissions  `

`  POST /v1/{resource=projects/*/instances/*/databases/*/backupSchedules/*}:testIamPermissions  `  
Returns permissions that the caller has on the specified database or backup resource.

## REST Resource: [v1.projects.instances.databases.databaseRoles](/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles)

Methods

`  list  `

`  GET /v1/{parent=projects/*/instances/*/databases/*}/databaseRoles  `  
Lists Cloud Spanner database roles.

`  testIamPermissions  `

`  POST /v1/{resource=projects/*/instances/*/databases/*/databaseRoles/*}:testIamPermissions  `  
Returns permissions that the caller has on the specified database or backup resource.

## REST Resource: [v1.projects.instances.databases.operations](/spanner/docs/reference/rest/v1/projects.instances.databases.operations)

Methods

`  cancel  `

`  POST /v1/{name=projects/*/instances/*/databases/*/operations/*}:cancel  `  
Starts asynchronous cancellation on a long-running operation.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/databases/*/operations/*}  `  
Deletes a long-running operation.

`  get  `

`  GET /v1/{name=projects/*/instances/*/databases/*/operations/*}  `  
Gets the latest state of a long-running operation.

`  list  `

`  GET /v1/{name=projects/*/instances/*/databases/*/operations}  `  
Lists operations that match the specified filter in the request.

## REST Resource: [v1.projects.instances.databases.sessions](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions)

Methods

`  adaptMessage  `

`  POST /v1/{name=projects/*/instances/*/databases/*/sessions/*}:adaptMessage  `  
Handles a single message from the client and returns the result as a stream.

`  adapter  `

`  POST /v1/{parent=projects/*/instances/*/databases/*}/sessions:adapter  `  
Creates a new session to be used for requests made by the adapter.

`  batchCreate  `

`  POST /v1/{database=projects/*/instances/*/databases/*}/sessions:batchCreate  `  
Creates multiple new sessions.

`  batchWrite  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:batchWrite  `  
Batches the supplied mutation groups in a collection of efficient transactions.

`  beginTransaction  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:beginTransaction  `  
Begins a new transaction.

`  commit  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:commit  `  
Commits a transaction.

`  create  `

`  POST /v1/{database=projects/*/instances/*/databases/*}/sessions  `  
Creates a new session.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/databases/*/sessions/*}  `  
Ends a session, releasing server resources associated with it.

`  executeBatchDml  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeBatchDml  `  
Executes a batch of SQL DML statements.

`  executeSql  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeSql  `  
Executes an SQL statement, returning all results in a single reply.

`  executeStreamingSql  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeStreamingSql  `  
Like `  ExecuteSql  ` , except returns the result set as a stream.

`  get  `

`  GET /v1/{name=projects/*/instances/*/databases/*/sessions/*}  `  
Gets a session.

`  list  `

`  GET /v1/{database=projects/*/instances/*/databases/*}/sessions  `  
Lists all sessions in a given database.

`  partitionQuery  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:partitionQuery  `  
Creates a set of partition tokens that can be used to execute a query operation in parallel.

`  partitionRead  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:partitionRead  `  
Creates a set of partition tokens that can be used to execute a read operation in parallel.

`  read  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:read  `  
Reads rows from the database using key lookups and scans, as a simple key/value style alternative to `  ExecuteSql  ` .

`  rollback  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:rollback  `  
Rolls back a transaction, releasing any locks it holds.

`  streamingRead  `

`  POST /v1/{session=projects/*/instances/*/databases/*/sessions/*}:streamingRead  `  
Like `  Read  ` , except returns the result set as a stream.

## REST Resource: [v1.projects.instances.instancePartitionOperations](/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations)

Methods

`  list  `

`  GET /v1/{parent=projects/*/instances/*}/instancePartitionOperations  `  
Lists instance partition long-running operations in the given instance.

## REST Resource: [v1.projects.instances.instancePartitions](/spanner/docs/reference/rest/v1/projects.instances.instancePartitions)

Methods

`  create  `

`  POST /v1/{parent=projects/*/instances/*}/instancePartitions  `  
Creates an instance partition and begins preparing it to be used.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/instancePartitions/*}  `  
Deletes an existing instance partition.

`  get  `

`  GET /v1/{name=projects/*/instances/*/instancePartitions/*}  `  
Gets information about a particular instance partition.

`  list  `

`  GET /v1/{parent=projects/*/instances/*}/instancePartitions  `  
Lists all instance partitions for the given instance.

`  patch  `

`  PATCH /v1/{instancePartition.name=projects/*/instances/*/instancePartitions/*}  `  
Updates an instance partition, and begins allocating or releasing resources as requested.

## REST Resource: [v1.projects.instances.instancePartitions.operations](/spanner/docs/reference/rest/v1/projects.instances.instancePartitions.operations)

Methods

`  cancel  `

`  POST /v1/{name=projects/*/instances/*/instancePartitions/*/operations/*}:cancel  `  
Starts asynchronous cancellation on a long-running operation.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/instancePartitions/*/operations/*}  `  
Deletes a long-running operation.

`  get  `

`  GET /v1/{name=projects/*/instances/*/instancePartitions/*/operations/*}  `  
Gets the latest state of a long-running operation.

`  list  `

`  GET /v1/{name=projects/*/instances/*/instancePartitions/*/operations}  `  
Lists operations that match the specified filter in the request.

## REST Resource: [v1.projects.instances.operations](/spanner/docs/reference/rest/v1/projects.instances.operations)

Methods

`  cancel  `

`  POST /v1/{name=projects/*/instances/*/operations/*}:cancel  `  
Starts asynchronous cancellation on a long-running operation.

`  delete  `

`  DELETE /v1/{name=projects/*/instances/*/operations/*}  `  
Deletes a long-running operation.

`  get  `

`  GET /v1/{name=projects/*/instances/*/operations/*}  `  
Gets the latest state of a long-running operation.

`  list  `

`  GET /v1/{name=projects/*/instances/*/operations}  `  
Lists operations that match the specified filter in the request.
