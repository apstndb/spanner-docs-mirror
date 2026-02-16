NAME

gcloud alpha spanner instances describe - describe a Cloud Spanner instance

SYNOPSIS

`  gcloud alpha spanner instances describe  ` `  INSTANCE  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

`  (ALPHA)  ` Describe a Cloud Spanner instance.

EXAMPLES

To describe a Cloud Spanner instance, run:

``` text
gcloud alpha spanner instances describe my-instance-id
```

POSITIONAL ARGUMENTS

  - `  INSTANCE  `  
    Cloud Spanner instance ID.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --access-token-file  ` , `  --account  ` , `  --billing-project  ` , `  --configuration  ` , `  --flags-file  ` , `  --flatten  ` , `  --format  ` , `  --help  ` , `  --impersonate-service-account  ` , `  --log-http  ` , `  --project  ` , `  --quiet  ` , `  --trace-token  ` , `  --user-output-enabled  ` , `  --verbosity  ` .

Run `  $ gcloud help  ` for details.

NOTES

This command is currently in alpha and might change without notice. If this command fails with API permission errors despite specifying the correct project, you might be trying to access an API with an invitation-only early access allowlist. These variants are also available:

``` text
gcloud spanner instances describe
```

``` text
gcloud beta spanner instances describe
```
