[Terraform](https://www.terraform.io/) is a tool for building, changing, and versioning infrastructure. It uses configuration files to describe the components needed to run a single application or your entire infrastructure.

You can use the [Google Cloud Terraform Provider](https://www.terraform.io/docs/providers/google/index.html) to create Spanner instances and databases, and to modify or delete Spanner resources.

**PostgreSQL interface note:** For PostgreSQL-dialect databases, Terraform doesn't support submitting DDL while creating a database. You must submit DDL in a separate operation.

The Google Cloud Terraform Provider offers the following Spanner resources:

  - [google\_spanner\_instance](https://www.terraform.io/docs/providers/google/r/spanner_instance)
  - [google\_spanner\_database](https://www.terraform.io/docs/providers/google/r/spanner_database)
  - [google\_spanner\_instance\_iam](https://www.terraform.io/docs/providers/google/r/spanner_instance_iam)
  - [google\_spanner\_database\_iam](https://www.terraform.io/docs/providers/google/r/spanner_database_iam)

For more information about using these resources to create and manage Spanner instances and databases, see [Using Terraform with Google Cloud](https://cloud.google.com/docs/terraform) or consult the Google Cloud Terraform Provider [reference](https://registry.terraform.io/providers/hashicorp/google/latest/docs) .

## What's next

  - Learn more about [Terraform](https://www.terraform.io/) .
  - Try out [code examples](https://github.com/cloudspannerecosystem/spanner-terraform#examples) that use the Google Cloud Terraform Provider with Spanner.
  - For a step-by-step tutorial, try the Spanner with Terraform [codelab](https://codelabs.developers.google.com/codelabs/cloud-spanner-terraform#0) .
  - View the Google Cloud Terraform Provider [repository on GitHub](https://github.com/hashicorp/terraform-provider-google) .
  - [File a GitHub issue](https://github.com/hashicorp/terraform-provider-google/issues) to report a bug or ask a question about Terraform.
