Use the Dataflow connector to write data by using a MutationGroup class to ensure that a group of mutations is applied atomically.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Import, export, and modify data using Dataflow](/spanner/docs/dataflow-connector)

## Code sample

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
PCollection<MutationGroup> mutations =
    suspiciousUserIds.apply(
        MapElements.via(
            new SimpleFunction<>() {

              @Override
              public MutationGroup apply(String userId) {
                // Immediately block the user.
                Mutation userMutation =
                    Mutation.newUpdateBuilder("Users")
                        .set("id")
                        .to(userId)
                        .set("state")
                        .to("BLOCKED")
                        .build();
                long generatedId =
                    Hashing.sha1()
                        .newHasher()
                        .putString(userId, Charsets.UTF_8)
                        .putLong(timestamp.getSeconds())
                        .putLong(timestamp.getNanos())
                        .hash()
                        .asLong();

                // Add an entry to pending review requests.
                Mutation pendingReview =
                    Mutation.newInsertOrUpdateBuilder("PendingReviews")
                        .set("id")
                        .to(generatedId) // Must be deterministically generated.
                        .set("userId")
                        .to(userId)
                        .set("action")
                        .to("REVIEW ACCOUNT")
                        .set("note")
                        .to("Suspicious activity detected.")
                        .build();

                return MutationGroup.create(userMutation, pendingReview);
              }
            }));

mutations.apply(SpannerIO.write()
    .withInstanceId(instanceId)
    .withDatabaseId(databaseId)
    .grouped());
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
