Create an array of STRUCT objects populated with data.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Working with STRUCT objects](/spanner/docs/structs)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
// Cloud Spanner STRUCT<> types with named fields are represented by
// std::tuple<std::pair<std::string, T>...>, create an alias to make it easier
// to follow this code.
using SingerName = std::tuple<std::pair<std::string, std::string>,
                              std::pair<std::string, std::string>>;
auto make_name = [](std::string first_name, std::string last_name) {
  return std::make_tuple(std::make_pair("FirstName", std::move(first_name)),
                         std::make_pair("LastName", std::move(last_name)));
};
std::vector<SingerName> singer_info{
    make_name("Elena", "Campbell"),
    make_name("Gabriel", "Wright"),
    make_name("Benjamin", "Martinez"),
};
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
var bandMembers = new List<SpannerStruct>
{
    new SpannerStruct { { "FirstName", SpannerDbType.String, "Elena" }, { "LastName", SpannerDbType.String, "Campbell" } },
    new SpannerStruct { { "FirstName", SpannerDbType.String, "Gabriel" }, { "LastName", SpannerDbType.String, "Wright" } },
    new SpannerStruct { { "FirstName", SpannerDbType.String, "Benjamin" }, { "LastName", SpannerDbType.String, "Martinez" } },
};
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
var bandMembers = []nameType{
 {"Elena", "Campbell"},
 {"Gabriel", "Wright"},
 {"Benjamin", "Martinez"},
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
List<Struct> bandMembers = new ArrayList<>();
bandMembers.add(
    Struct.newBuilder().set("FirstName").to("Elena").set("LastName").to("Campbell").build());
bandMembers.add(
    Struct.newBuilder().set("FirstName").to("Gabriel").set("LastName").to("Wright").build());
bandMembers.add(
    Struct.newBuilder().set("FirstName").to("Benjamin").set("LastName").to("Martinez").build());
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
const bandMembersType = {
  type: 'array',
  child: nameType,
};

const bandMembers = [
  Spanner.struct({
    FirstName: 'Elena',
    LastName: 'Campbell',
  }),
  Spanner.struct({
    FirstName: 'Gabriel',
    LastName: 'Wright',
  }),
  Spanner.struct({
    FirstName: 'Benjamin',
    LastName: 'Martinez',
  }),
];
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` php
$bandMembers = [
    (new StructValue)
        ->add('FirstName', 'Elena')
        ->add('LastName', 'Campbell'),
    (new StructValue)
        ->add('FirstName', 'Gabriel')
        ->add('LastName', 'Wright'),
    (new StructValue)
        ->add('FirstName', 'Benjamin')
        ->add('LastName', 'Martinez')
];
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
band_members = [
    ("Elena", "Campbell"),
    ("Gabriel", "Wright"),
    ("Benjamin", "Martinez"),
]
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
band_members = [name_type.struct(["Elena", "Campbell"]),
                name_type.struct(["Gabriel", "Wright"]),
                name_type.struct(["Benjamin", "Martinez"])]
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
