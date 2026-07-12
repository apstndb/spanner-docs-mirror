# Google Cloud Spanner Documentation Mirror

This repository contains a local Markdown mirror of the official Google Cloud Spanner documentation, generated automatically to support offline analysis and parser development.

## Content Coverage

## Metadata
For detailed stats and sync info, see [metadata.yaml](./metadata.yaml).

- **Guides & Concepts**: /spanner/docs/*
- **SQL Reference**: GoogleSQL and PostgreSQL compatible interfaces.
- **API Reference**: REST and RPC definitions.
- **gcloud CLI**: Full command reference for Spanner (including alpha/beta).
- **Code Samples**: Extensive collection of Spanner code samples across multiple languages.
- **Product & Pricing**: The separate `cloud.google.com/spanner` product document, including pricing tables exposed by the Developer Knowledge API.

The Developer Knowledge API treats `docs.cloud.google.com` and `cloud.google.com` as distinct corpora. This mirror preserves both hostnames; legacy `cloud.google.com/spanner/docs` links are resolved through their web redirect to `docs.cloud.google.com`.

The product-page seed requires `gcp-docs-mirror-tools` v0.3.0 or newer and is active in the workflow.

This mirror was generated using the [gcp-docs-mirror-tools](https://github.com/apstndb/gcp-docs-mirror-tools).

## License
The content in this repository is mirrored from Google Cloud Documentation according to the [Google Developers Site Policies](https://developers.google.com/terms/site-policies).
- Documentation content is licensed under [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- Code samples are licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).
