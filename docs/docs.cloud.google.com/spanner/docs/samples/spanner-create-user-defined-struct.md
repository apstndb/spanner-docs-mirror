Create a user-defined STRUCT.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Working with STRUCT objects](/spanner/docs/structs)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
// Cloud Spanner STRUCT<> types are represented by std::tuple<...>. The
// following represents a STRUCT<> with two unnamed STRING fields.
using NameType = std::tuple<std::string, std::string>;
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
var nameType = new SpannerStruct
{
    { "FirstName", SpannerDbType.String, null},
    { "LastName", SpannerDbType.String, null}
};
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
type nameType struct {
 FirstName string
 LastName  string
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
Type nameType =
    Type.struct(
        Arrays.asList(
            StructField.of("FirstName", Type.string()),
            StructField.of("LastName", Type.string())));
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
const nameType = {
  type: 'struct',
  fields: [
    {
      name: 'FirstName',
      type: 'string',
    },
    {
      name: 'LastName',
      type: 'string',
    },
  ],
};
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` php
$nameType = new ArrayType(
    (new StructType)
        ->add('FirstName', Database::TYPE_STRING)
        ->add('LastName', Database::TYPE_STRING)
);
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
name_type = param_types.Struct(
    [
        param_types.StructField("FirstName", param_types.STRING),
        param_types.StructField("LastName", param_types.STRING),
    ]
)
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
name_type = client.fields FirstName: :STRING, LastName: :STRING
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
