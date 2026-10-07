---
name: documents/docs.cloud.google.com/spanner/docs/reference/standard-sql/security_functions
uri: https://docs.cloud.google.com/spanner/docs/reference/standard-sql/security_functions
title: Security functions in GoogleSQL
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

GoogleSQL for Spanner supports the following security functions.

## Function list

| Name                                                                                                                | Summary                                                                             |
|---------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| [`SESSION_USER`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/security_functions#session_user) | Get the email address or principal identifier of the user that's running the query. |

## `SESSION_USER`

```
SESSION_USER()
```

**Description**

For first-party users, returns the email address of the user that's running the query. For third-party users, returns the [principal identifier](https://cloud.google.com/iam/docs/principal-identifiers) of the user that's running the query. For more information about identities, see [Principals](https://cloud.google.com/docs/authentication#principal) .

**Return Data Type**

`STRING`

**Example**

```
SELECT SESSION_USER() as user;

/*----------------------+
 | user                 |
 +----------------------+
 | jdoe@example.com     |
 +----------------------*/
```
