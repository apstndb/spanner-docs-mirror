Create a hibernate UUID.

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
@JdbcTypeCode(java.sql.Types.VARCHAR)
private UUID id;
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
