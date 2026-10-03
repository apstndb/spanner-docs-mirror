---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/EncryptionInfo
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/EncryptionInfo
title: EncryptionInfo
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/EncryptionInfo#SCHEMA_REPRESENTATION)
- [Type](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/EncryptionInfo#Type)

Encryption information for a Cloud Spanner database or backup.

**JSON representation**

```
{
  "encryptionType": enum (Type),
  "encryptionStatus": {
    object (Status)
  },
  "kmsKeyVersion": string
}
```

| Fields             |                                                                                                                                                                                                                                                                             |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `encryptionType`   | `enum ( `[`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/EncryptionInfo#Type)` )` Output only. The type of encryption.                                                                                                                                |
| `encryptionStatus` | `object ( `[`Status`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Status)` )` Output only. If present, the status of a recent encrypt/decrypt call on underlying data for this database or backup. Regardless of status, data is always encrypted at rest. |
| `kmsKeyVersion`    | `string` Output only. A Cloud KMS key version that is being used to protect the database or backup.                                                                                                                                                                         |

## Type

Possible encryption types.

| Enums                         |                                                                                                                                                                                      |
|-------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `TYPE_UNSPECIFIED`            | Encryption type was not specified, though data at rest remains encrypted.                                                                                                            |
| `GOOGLE_DEFAULT_ENCRYPTION`   | The data is encrypted at rest with a key that is fully managed by Google. No key version or status will be populated. This is the default state.                                     |
| `CUSTOMER_MANAGED_ENCRYPTION` | The data is encrypted at rest with a key that is managed by the customer. The active version of the key. `kmsKeyVersion` will be populated, and `encryptionStatus` may be populated. |
