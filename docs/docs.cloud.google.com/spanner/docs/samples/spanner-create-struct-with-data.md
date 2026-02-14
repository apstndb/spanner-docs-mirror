Create a STRUCT object populated with data.

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
auto singer_info = NameType{"Elena", "Campbell"};
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
var nameStruct = new SpannerStruct
{
    { "FirstName", SpannerDbType.String, "Elena" },
    { "LastName", SpannerDbType.String, "Campbell" },
};
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
type name struct {
 FirstName string
 LastName  string
}
var singerInfo = name{"Elena", "Campbell"}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
Struct name =
    Struct.newBuilder().set("FirstName").to("Elena").set("LastName").to("Campbell").build();
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

const nameStruct = Spanner.struct({
  FirstName: 'Elena',
  LastName: 'Campbell',
});
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` php
$nameValue = (new StructValue)
    ->add('FirstName', 'Elena')
    ->add('LastName', 'Campbell');
$nameType = (new StructType)
    ->add('FirstName', Database::TYPE_STRING)
    ->add('LastName', Database::TYPE_STRING);
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
record_type = param_types.Struct(
    [
        param_types.StructField("FirstName", param_types.STRING),
        param_types.StructField("LastName", param_types.STRING),
    ]
)
record_value = ("Elena", "Campbell")
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
name_struct = { FirstName: "Elena", LastName: "Campbell" }
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
