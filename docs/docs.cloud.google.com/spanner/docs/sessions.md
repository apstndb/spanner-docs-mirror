This page describes the advanced concept of sessions in Spanner, including best practices for sessions when creating a client library, using the REST or RPC APIs, or using the Google client libraries.

A session represents a communication channel with the Spanner database service. A session is used to perform transactions that read, write, or modify data in a Spanner database. Each session applies to a single database.

Sessions can execute a single or multiple [transactions](/spanner/docs/transactions) at a time. When performing multiple transactions, the session is called a [multiplexed session](#multiplexed_sessions) . Standalone reads, writes, and queries use one transaction internally.

## Performance benefits of a session pool

Creating a session is expensive. To avoid the performance cost each time a database operation is made, clients should keep a *session pool* , which is a pool of available sessions that are ready to use. The pool should store existing sessions and return the appropriate type of session when requested, as well as handle cleanup of unused sessions. For an example of how to implement a session pool, see the source code for one of the Spanner client libraries, such as the [Go client library](https://github.com/GoogleCloudPlatform/google-cloud-go/blob/master/spanner/session.go) or the [Java client library](https://github.com/googleapis/java-spanner/blob/master/google-cloud-spanner/src/main/java/com/google/cloud/spanner/SessionPool.java) .

Sessions are intended to be long-lived, so after a session is used for a database operation, the client should return the session to the pool for reuse.

## Overview of gRPC channels

gRPC channels are used by the Spanner client for communication. One gRPC channel is roughly equivalent to a TCP connection. One gRPC channel can handle up to 100 concurrent requests. This means that an application will need at least as many gRPC channels as the number of concurrent requests the application will execute, divided by 100.

The Spanner client creates a pool of gRPC channels when you create it.

## Best practices when using Google client libraries

The following describes best practices when using the Google [client libraries](/spanner/docs/reference/libraries) for Spanner.

### Configure the number of sessions and gRPC channels in the pools

The client libraries have a default number of sessions in the session pool and a default number of gRPC channels in the channel pool. Both defaults are adequate for most cases. The following are the default minimum and maximum sessions and the default number of gRPC channels for each programming language.

### C++

``` text
MinSessions: 100
MaxSessions: 400
NumChannels: 4
```

### C\#

``` text
MinSessions: 100
MaxSessions: 400
NumChannels: 4
```

### Go

``` text
MinSessions: 100
MaxSessions: 400
NumChannels: 4
```

### Java

``` text
MinSessions: 100
MaxSessions: 400
NumChannels: 4
```

### Node.js

The Node.js client does not support multiple gRPC channels. It is therefore recommended to create multiple clients instead of increasing the size of the session pool beyond 100 sessions for a single client.

``` text
MinSessions: 25
MaxSessions: 100
```

### PHP

The PHP client does not support a configurable number of gRPC channels.

``` text
MinSessions: 1
MaxSessions: 500
```

### Python

Python supports four different [session pool types](/python/docs/reference/spanner/latest/advanced-session-pool-topics) that you can use to manage sessions.

### Ruby

The Ruby client does not support multiple gRPC channels. It is therefore recommended to create multiple clients instead of increasing the size of the session pool beyond 100 sessions for a single client.

``` text
MinSessions: 10
MaxSessions: 100
```

The number of sessions that your application uses is equal to the number of concurrent transactions that your application executes. You should modify the default session pool settings only if you expect a single application instance to execute more concurrent transactions than the default session pool can handle.

For high concurrency applications the following is recommended:

1.  Set `  MinSessions  ` to the expected number of concurrent transactions that a single client will execute.
2.  Set `  MaxSessions  ` to the maximum number of concurrent transactions that a single client can execute.
3.  Set `  MinSessions=MaxSessions  ` if the expected concurrency does not change much during the application lifetime. This prevents the session pool from scaling up or down. Scaling the session pool up or down also consumes some resources.
4.  Set `  NumChannels  ` to `  MaxSessions / 100  ` . One gRPC channel can handle up to 100 requests concurrently. Increase this value if you observe a high tail latency (p95/p99 latency), because this could be an indication of gRPC channel congestion.

Increasing the number of active sessions uses additional resources on the Spanner database service and in the client library. Increasing the number of sessions beyond the actual need of the application could degrade the performance of your system.

### Increase the session pool versus increasing the number of clients

The session pool size for an application determines how many concurrent transactions a single application instance can execute. Increasing the session pool size beyond the maximum concurrency that a single application instance can handle is not recommended. If the application receives a burst of requests that goes beyond the number of sessions in the pool, the requests are queued while waiting for a session to become available.

The resources that are consumed by the client library are the following:

1.  Each gRPC channel uses one TCP connection.
2.  Each gRPC invocation requires a thread. The maximum number of threads that is used by the client library is equal to the maximum number of concurrent queries that the application executes. These threads come on top of any threads that the application uses for its own business logic.

Increasing the size of the session pool beyond the maximum number of threads that a single application instance can handle is not recommended. Instead, increase the number of application instances.

### Manage the write-sessions fraction

For some client libraries, Spanner reserves a portion of the sessions for read-write transactions, called the write-sessions fraction. If your app uses up all the read sessions, then Spanner uses the read-write sessions, even for read-only transactions. Read-write sessions require `  spanner.databases.beginOrRollbackReadWriteTransaction  ` . If the user is in the [`  spanner.databaseReader  `](/spanner/docs/iam#roles) IAM role, then the call fails and Spanner returns this error message:

``` text
generic::permission_denied: Resource %resource% is missing IAM permission:
spanner.databases.beginOrRollbackReadWriteTransaction
```

For the client libraries that maintain a write-sessions fraction, you can set the write-sessions fraction.

### C++

All C++ sessions are the same. There are no read or read-write only sessions.

### C\#

The default write-sessions fraction for C\# is 0.2. You can change the fraction using the WriteSessionsFraction field of [`  SessionPoolOptions  `](/dotnet/docs/reference/Google.Cloud.Spanner.V1/latest/Google.Cloud.Spanner.V1.SessionPoolOptions#Google_Cloud_Spanner_V1_SessionPoolOptions_WriteSessionsFraction) .

### Go

All Go sessions are the same. There are no read or read-write only sessions.

### Java

All Java sessions are the same. There are no read or read-write only sessions.

### Node.js

All Node.js sessions are the same. There are no read or read-write only sessions.

### PHP

All PHP sessions are the same. There are no read or read-write only sessions.

### Python

Python supports four different [session pool types](/python/docs/reference/spanner/latest/advanced-session-pool-topics) that you can use to manage read and read-write sessions.

### Ruby

The default write-sessions fraction for Ruby is 0.3. You can change the fraction using the [client](https://github.com/googleapis/ruby-spanner/blob/-/google-cloud-spanner/lib/google/cloud/spanner/pool.rb#L39) initialize method.

## Best practices when creating a client library or using REST/RPC

The following describes best practices for implementing sessions in a client library for Spanner, or for using sessions with the [REST](/spanner/docs/reference/rest) or [RPC](/spanner/docs/reference/rpc) APIs.

These best practices only apply if you are *developing* a client library, or if you are using REST/RPC APIs. If you are using one of the Google client libraries for Spanner, refer to [Best practices when using Google client libraries](#best_practices_when_using_google_client_libraries) .

### Create and size the session pool

To determine an optimal size of the session pool for a client process, set the lower bound to the number of expected concurrent transactions, and set the upper bound to an initial test number, such as 100. If the upper bound is not adequate, increase it. Increasing the number of active sessions uses additional resources on the Spanner database service, so failing to clean up unused sessions can degrade performance. For users working with the RPC API, we recommend having no more than 100 sessions per gRPC channel.

### Handle deleted sessions

There are three ways to delete a session:

  - A client can delete a session.
  - The Spanner database service can delete a session when the session is idle for more than 1 hour.
  - The Spanner database service may delete a session if the session is more than 28 days old.

Attempts to use a deleted session result in [`  NOT_FOUND  `](/spanner/docs/reference/rpc/google.rpc#google.rpc.Code) . If you encounter this error, create and use a new session, add the new session to the pool, and remove the deleted session from the pool.

### Keep an idle session alive

The Spanner database service reserves the right to drop an unused session. If you definitely need to keep an idle session alive, for example, if a significant near-term increase in database use is expected, then you can prevent the session from being dropped. Perform an inexpensive operation such as executing the SQL query `  SELECT 1  ` to keep the session alive. If you have an idle session that is not needed for near-term use, let Spanner drop the session, and then create a new session the next time a session is needed.

One scenario for keeping sessions alive is to handle regular peak demand on the database. If heavy database use occurs daily from 9:00 AM to 6:00 PM, you should keep some idle sessions alive during that time, since they are likely required for the peak usage. After 6:00 PM, you can let Spanner drop idle sessions. Prior to 9:00 AM each day, create some new sessions so they will be ready for the expected demand.

Another scenario is if you have an application that uses Spanner but must avoid the connection overhead when it does. You can keep a set of sessions alive to avoid the connection overhead.

### Hide session details from the client library user

If you are creating a client library, don't expose sessions to the client library consumer. Provide the ability for the client to make database calls without the complexity of creating and maintaining sessions. For an example of a client library that hides the session details from the client library consumer, see the Spanner client library for Java.

### Handle errors for write transactions that are not idempotent

Write transactions without replay protection may apply mutations more than once. If a mutation is not idempotent, a mutation that is applied more than once could result in a failure. For example, an insert may fail with [`  ALREADY_EXISTS  `](/spanner/docs/reference/rpc/google.rpc#google.rpc.Code) even though the row did not exist prior to the write attempt. This could occur if the backend server committed the mutation but was unable to communicate the success to the client. In that event, the mutation could be retried, resulting in the `  ALREADY_EXISTS  ` failure.

Here are possible ways to address this scenario when you implement your own client library or use the REST API:

  - Structure your writes to be idempotent.
  - Use writes with replay protection.
  - Implement a method that performs "upsert" logic: insert if new or update if exists.
  - Handle the error on behalf of the client.

### Maintain stable connections

For best performance, the connection that you use to host a session should remain stable. When the connection that hosts a session changes, Spanner might abort the active transaction on the session and cause a small amount of extra load on your database while it updates the session metadata. It is OK if a few connections change sporadically, but you should avoid situations that would change a large number of connections at the same time. If you use a proxy between the client and Spanner, you should maintain connection stability for each session.

### Monitor active sessions

You can use the `  ListSessions  ` command to monitor active sessions in your database from the [command line](/spanner/docs/gcloud-spanner#manage_sessions) , with [the REST API](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list) , or with [the RPC API](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ListSessions) . `  ListSessions  ` shows the active sessions for a given database. This is useful if you need to find the cause of a session leak. (A session leak is an incident where sessions are being created but not returned to a session pool for reuse.)

`  ListSessions  ` lets you view metadata about your active sessions, including when a session was created and when a session was last used. Analyzing this data will point you in the right direction when troubleshooting sessions. If most active sessions don't have a recent `  approximate_last_use_time  ` , this could indicate that sessions aren't being reused properly by your application. See the [RPC API reference](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session) for more information about the `  approximate_last_use_time  ` field.

See the [REST API reference](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list) , the [RPC API reference](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ListSessions) , or the [gcloud command-line tool reference](/spanner/docs/gcloud-spanner#manage_sessions) for more information on using `  ListSessions  ` .

### Automatic cleanup of session leaks

**Preview — [Automatic cleanup of session leaks](/spanner/docs/sessions#automatic_cleanup_of_session_leaks)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

When you use all the sessions in your session pool, each new transaction waits until a session is returned to the pool. When sessions are created but not returned to the session pool for reuse, this is called a session leak. When there is a session leak, transactions waiting for an open session get stuck indefinitely and block the application. Session leaks are often caused by problematic transactions that are running for an extremely long time and aren't committed.

You can setup your session pool to automatically resolve these inactive transactions. When you enable your client library to automatically resolve inactive transition, it identifies problematic transactions that might cause a session leak, removes them from the session pool, and replaces them with a new session.

Logging can also help identify these problematic transactions. If logging is enabled, warning logs are shared by default when more than 95% of your session pool is in use. If your session usage is greater than 95%, then you either need to increase the max sessions allowed in your session pool, or you might have a session leak. Warning logs contain stack traces of transactions that run for longer than expected and can help identify the cause of high session pool utilization. Warning logs are pushed depending on your log exporter configuration.

#### Enable client library to automatically resolve inactive transactions

You can either enable client library to send warning logs and automatically resolve inactive transactions, or enable client library to only receive warning logs.

### Java

To receive warning logs and remove inactive transactions, use `  setWarnAndCloseIfInactiveTransactions  ` .

``` text
 final SessionPoolOptions sessionPoolOptions = SessionPoolOptions.newBuilder().setWarnAndCloseIfInactiveTransactions().build()

 final Spanner spanner =
         SpannerOptions.newBuilder()
             .setSessionPoolOption(sessionPoolOptions)
             .build()
             .getService();
 final DatabaseClient client = spanner.getDatabaseClient(databaseId);
```

To only receive warning logs, use `  setWarnIfInactiveTransactions  ` .

``` text
 final SessionPoolOptions sessionPoolOptions = SessionPoolOptions.newBuilder().setWarnIfInactiveTransactions().build()

 final Spanner spanner =
         SpannerOptions.newBuilder()
             .setSessionPoolOption(sessionPoolOptions)
             .build()
             .getService();
 final DatabaseClient client = spanner.getDatabaseClient(databaseId);
```

### Go

To receive warning logs and remove inactive transactions, use `  SessionPoolConfig  ` with `  InactiveTransactionRemovalOptions  ` .

``` text
 client, err := spanner.NewClientWithConfig(
     ctx, database, spanner.ClientConfig{SessionPoolConfig: spanner.SessionPoolConfig{
         InactiveTransactionRemovalOptions: spanner.InactiveTransactionRemovalOptions{
         ActionOnInactiveTransaction: spanner.WarnAndClose,
         }
     }},
 )
 if err != nil {
     return err
 }
 defer client.Close()
```

To only receive warning logs, use `  customLogger  ` .

``` text
 customLogger := log.New(os.Stdout, "spanner-client: ", log.Lshortfile)
 // Create a logger instance using the golang log package
 cfg := spanner.ClientConfig{
         Logger: customLogger,
     }
 client, err := spanner.NewClientWithConfig(ctx, db, cfg)
```

## Multiplexed sessions

Multiplexed sessions let you create a large number of concurrent requests on a single session. A multiplexed session is an identifier that you use across multiple gRPC channels. It doesn't introduce any additional bottlenecks. Multiplexed sessions have the following advantages:

  - Reduced backend resource consumption due to a more straightforward session management protocol. For example, they avoid session maintenance activities associated with session ownership maintenance and garbage collection.
  - Long-lived session that doesn't require keep-alive requests when idle.

Multiplexed sessions are supported in the following:

  - The C++, Go, Java, Node.js, PHP, Python, and Ruby [client libraries](/spanner/docs/reference/libraries) .

  - Spanner ecosystem tools that depend on the mentioned client Libraries, such as PGAdapter, JDBC, Hibernate, database/sql driver, dbAPI driver and GORM.

  - Spanner ecosystem tools that depend on the Java and Go client Libraries, such as PGAdapter, JDBC, Hibernate, database or sql driver, and GORM. You can use [OpenTelemetry metrics](#opentelemetry) to see how traffic is split between the existing session pool and the multiplexed session. OpenTelemetry has a metric filter, `  is_multiplexed  ` , that shows multiplexed sessions when set to `  true  ` .

Multiplexed sessions are supported for all types of transactions.

Client libraries rotate multiplexed sessions every 7 days to prevent sending transactions on stale sessions.

Multiplexed sessions are enabled by default in some client libraries. For others, you must use environment variables to enable them. For details, see [Enable multiplexed sessions](#enable-multiplex) .

### Considerations

If you're trying to commit either an empty read or write transaction body or a transaction where every query or DML statement has failed, there are a couple of scenarios to consider with multiplexed sessions. Multiplexed sessions require that you include a server-generated pre-commit token in each commit request. For transactions that contain queries or DML, there must be at least one previous successful query or DML transaction for the server to send back a valid token to the client library. If there haven't been any successful queries or DML transactions, the client library implicitly adds `  SELECT 1  ` before a commit.

For a read or write transaction on a multiplexed session that only has mutations, if one of the mutations is for a table or a column that does NOT exist in the schema, the client could return an `  INVALID_ARGUMENT  ` error instead of a `  NOT_FOUND  ` error.

### Enable multiplexed sessions

Multiplexed sessions are enabled by default in the following client libraries:

  - C++ in version 2.41.0 and later.
  - Go in version 1.85.0 and later.
  - Java in version 6.98.0 and later.
  - Node.js in version 8.3.0 and later.
  - PHP in version 2.0.0 and later.
  - Python in version 3.57.0 and later.
  - Ruby in version 2.30.0 and later.

To use multiplexed sessions in earlier versions of Node.js, Java and Go client libraries, you must first set an environment variable to enable it.

To enable multiplexed sessions, set the `  GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS  ` environment variable to `  TRUE  ` . This flag also enables the multiplexed sessions support for `  ReadOnly  ` transactions.

``` text
export GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS=TRUE
```

To enable partitioned operations support for multiplexed sessions, set the `  GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS_PARTITIONED_OPS  ` environment variable to `  TRUE  ` .

``` text
export GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS_PARTITIONED_OPS=TRUE
```

To enable read-write transactions support for multiplexed sessions, set the `  GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS_FOR_RW  ` environment variable to `  TRUE  ` .

``` text
export GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS_FOR_RW=True
```

You must set `  GOOGLE_CLOUD_SPANNER_MULTIPLEXED_SESSIONS  ` to `  TRUE  ` as a prerequisite for supporting a transaction on a multiplexed session.

### View traffic for regular and multiplexed sessions

OpenTelemetry has the `  is_multiplexed  ` filter to show the traffic for multiplexed sessions. You set this filter to `  true to view multiplexed sessions and  ` false\` to view regular sessions.

1.  Set up OpenTelemetry for Spanner using the procedures in the Spanner OpenTelemetry [Before you begin](/spanner/docs/capture-visualize-latency#before_you_begin) section.

2.  Navigate to the **Metrics Explorer** .

3.  In the **Metric** drop-down, filter on `  generic  ` .

4.  Click **Generic Task** and navigate to **Spanner** \> **Spanner/num\_acquired\_sessions** .

5.  In the **Filter** field, select from the following options:
    
    a. `  is_multiplexed = false  ` to view regular sessions. b. `  is_multiplexed = true  ` to view multiplexed sessions.
    
    The following image shows the **Filter** option with multiplexed sessions selected.

For more information about using OpenTelemetry with Spanner, see [Leveraging OpenTelemetry to democratize Spanner Observability](https://cloud.google.com/blog/products/databases/consume-spanner-metrics-using-opentelemetery) and [Examine latency in a Spanner component with OpenTelemetry](/spanner/docs/capture-visualize-latency) .

## Troubleshoot

Common session-related errors that your application might encounter include:

  - `  Session not found  `
  - `  RESOURCE_EXHAUSTED  `

For more information, see [Session errors](/spanner/docs/error-codes#sessions) .
