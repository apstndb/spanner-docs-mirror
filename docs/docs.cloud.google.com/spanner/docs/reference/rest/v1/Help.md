---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/Help
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Help
title: Help
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Help#SCHEMA_REPRESENTATION)
- [Link](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Help#Link)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Help#Link.SCHEMA_REPRESENTATION)

Provides links to documentation or for performing an out of band action.

For example, if a quota check failed with an error indicating the calling project hasn't enabled the accessed service, this can contain a URL pointing directly to the right place in the developer console to flip the bit.

**JSON representation**

```
{
  "links": [
    {
      object (Link)
    }
  ]
}
```

| Fields    |                                                                                                                                                                          |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `links[]` | `object ( `[`Link`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Help#Link)` )` URL(s) pointing to additional information on handling the current error. |

## Link

Describes a URL link.

**JSON representation**

```
{
  "description": string,
  "url": string
}
```

| Fields        |                                          |
|---------------|------------------------------------------|
| `description` | `string` Describes what the link offers. |
| `url`         | `string` The URL of the link.            |
