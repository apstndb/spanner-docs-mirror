**PostgreSQL interface note:** The Spanner Entity Framework Core provider isn't supported in PostgreSQL databases.

The [Spanner Entity Framework Core provider](https://github.com/googleapis/dotnet-spanner-entity-framework) lets you create a Spanner database, run queries, and update data through an application using [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/) . The provider is compatible with [Microsoft.EntityFrameworkCore 8](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/8.0) .

## Set up the Spanner Entity Framework Core provider

To set up the Spanner Entity Framework Core provider in your application, add the following dependency.

``` text
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <OutputType>Exe</OutputType>
  </PropertyGroup>

  ...

  <ItemGroup>
    <PackageReference Include="Google.Cloud.EntityFrameworkCore.Spanner" Version="3.2.0" />
  </ItemGroup>

  ...

</Project>
```

As authentication for the Spanner Entity Framework Core provider, the [service account JSON credentials](/docs/authentication/getting-started) file location should be provided in the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable. Otherwise, the provider can also use the default credentials set in the Google Cloud CLI `  gcloud  ` application.

## Use the Spanner Entity Framework Core provider

For more information about the available features, limitations of the provider, recommendations on how to use the provider, and for code samples, consult the [reference documentation](https://github.com/googleapis/dotnet-spanner-entity-framework#readme) on GitHub.

## What's next

  - Checkout the [code examples](https://github.com/googleapis/dotnet-spanner-entity-framework/tree/master/Google.Cloud.EntityFrameworkCore.Spanner.Samples) on how to use the Spanner Entity Framework Core provider.
  - View the repository for the Spanner Entity Framework Core provider on [GitHub](https://github.com/googleapis/dotnet-spanner-entity-framework) .
  - File a [GitHub issue](https://github.com/googleapis/dotnet-spanner-entity-framework/issues) to report a bug or ask a question about the Spanner Entity Framework provider.
  - Learn more about [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/) .
  - Learn more about [NuGet](https://www.nuget.org/) .
  - Learn about authorization and authentication credentials in [Getting started with authentication](/docs/authentication/getting-started) .
