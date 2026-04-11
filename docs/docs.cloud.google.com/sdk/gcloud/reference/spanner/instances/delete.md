NAME

gcloud spanner instances delete - delete a Cloud Spanner instance

SYNOPSIS

`gcloud spanner instances delete` `  INSTANCE  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

Delete a Cloud Spanner instance.

EXAMPLES

To delete a Cloud Spanner instance, run:

``` wrap-code
gcloud spanner instances delete my-instance-id
```

POSITIONAL ARGUMENTS

  - `  INSTANCE  `  
    Cloud Spanner instance ID.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --access-token-file  ` , `  --account  ` , `  --billing-project  ` , `  --configuration  ` , `  --flags-file  ` , `  --flatten  ` , `  --format  ` , `  --help  ` , `  --impersonate-service-account  ` , `  --log-http  ` , `  --project  ` , `  --quiet  ` , `  --trace-token  ` , `  --user-output-enabled  ` , `  --verbosity  ` .

Run ` $ gcloud help  ` for details.

NOTES

These variants are also available:

``` wrap-code
gcloud alpha spanner instances delete
```

``` wrap-code
gcloud beta spanner instances delete
```
