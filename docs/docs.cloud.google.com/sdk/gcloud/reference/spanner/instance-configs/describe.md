NAME

gcloud spanner instance-configs describe - describe a Cloud Spanner instance configuration

SYNOPSIS

`  gcloud spanner instance-configs describe  ` `  INSTANCE_CONFIG  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

Describe a Cloud Spanner instance configuration.

EXAMPLES

To describe an instance config named regional-us-central1, run:

``` text
gcloud spanner instance-configs describe regional-us-central1
```

To describe an instance config named nam-eur-asia1, run:

``` text
gcloud spanner instance-configs describe nam-eur-asia1
```

POSITIONAL ARGUMENTS

  - `  INSTANCE_CONFIG  `  
    Cloud Spanner instance config.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --access-token-file  ` , `  --account  ` , `  --billing-project  ` , `  --configuration  ` , `  --flags-file  ` , `  --flatten  ` , `  --format  ` , `  --help  ` , `  --impersonate-service-account  ` , `  --log-http  ` , `  --project  ` , `  --quiet  ` , `  --trace-token  ` , `  --user-output-enabled  ` , `  --verbosity  ` .

Run `  $ gcloud help  ` for details.

NOTES

These variants are also available:

``` text
gcloud alpha spanner instance-configs describe
```

``` text
gcloud beta spanner instance-configs describe
```
