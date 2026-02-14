Insert data into a Spanner Graph.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Set up and query Spanner Graph](/spanner/docs/graph/set-up)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void InsertData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;

  auto insert_accounts =
      spanner::InsertMutationBuilder(
          "Account", {"id", "create_time", "is_blocked", "nick_name"})
          .EmplaceRow(7, spanner::Value("2020-01-10T06:22:20.12Z"), false,
                      "Vacation Fund")
          .EmplaceRow(16, spanner::Value("2020-01-27T17:55:09.12Z"), true,
                      "Vacation Fund")
          .EmplaceRow(20, spanner::Value("2020-02-18T05:44:20.12Z"), false,
                      "Rainy Day Fund")
          .Build();

  auto insert_persons =
      spanner::InsertMutationBuilder(
          "Person", {"id", "name", "birthday", "country", "city"})
          .EmplaceRow(1, "Alex", spanner::Value("1991-12-21T00:00:00.12Z"),
                      "Australia", "Adelaide")
          .EmplaceRow(2, "Dana", spanner::Value("1980-10-31T00:00:00.12Z"),
                      "Czech_Republic", "Moravia")
          .EmplaceRow(3, "Lee", spanner::Value("1986-12-07T00:00:00.12Z"),
                      "India", "Kollam")
          .Build();

  auto insert_transfers =
      spanner::InsertMutationBuilder(
          "AccountTransferAccount",
          {"id", "to_id", "amount", "create_time", "order_number"})
          .EmplaceRow(7, 16, 300.0, spanner::Value("2020-08-29T15:28:58.12Z"),
                      "304330008004315")
          .EmplaceRow(7, 16, 100.0, spanner::Value("2020-10-04T16:55:05.12Z"),
                      "304120005529714")
          .EmplaceRow(16, 20, 300.0, spanner::Value("2020-09-25T02:36:14.12Z"),
                      "103650009791820")
          .EmplaceRow(20, 7, 500.0, spanner::Value("2020-10-04T16:55:05.12Z"),
                      "304120005529714")
          .EmplaceRow(20, 16, 200.0, spanner::Value("2020-10-17T03:59:40.12Z"),
                      "302290001255747")
          .Build();

  auto insert_ownerships =
      spanner::InsertMutationBuilder("PersonOwnAccount",
                                     {"id", "account_id", "create_time"})
          .EmplaceRow(1, 7, spanner::Value("2020-01-10T06:22:20.12Z"))
          .EmplaceRow(2, 20, spanner::Value("2020-01-27T17:55:09.12Z"))
          .EmplaceRow(3, 16, spanner::Value("2020-02-18T05:44:20.12Z"))
          .Build();

  auto commit_result = client.Commit(spanner::Mutations{
      insert_accounts, insert_persons, insert_transfers, insert_ownerships});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Insert was successful [spanner_insert_graph_data]\n";
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "io"
 "time"

 "cloud.google.com/go/spanner"
)

func parseTime(rfc3339Time string) time.Time {
 t, _ := time.Parse(time.RFC3339, rfc3339Time)
 return t
}

func insertGraphData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 // Values are inserted into the node and edge tables corresponding to
 // using Spanner 'Insert' mutations.
 // The tables and columns comply with the schema defined for the
 // property graph 'FinGraph', comprising 'Person' and 'Account' nodes,
 // and 'PersonOwnAccount' and 'AccountTransferAccount' edges.
 personColumns := []string{"id", "name", "birthday", "country", "city"}
 accountColumns := []string{"id", "create_time", "is_blocked", "nick_name"}
 ownColumns := []string{"id", "account_id", "create_time"}
 transferColumns := []string{"id", "to_id", "amount", "create_time", "order_number"}
 m := []*spanner.Mutation{
     spanner.Insert("Account", accountColumns,
         []interface{}{7, parseTime("2020-01-10T06:22:20.12Z"), false, "Vacation Fund"}),
     spanner.Insert("Account", accountColumns,
         []interface{}{16, parseTime("2020-01-27T17:55:09.12Z"), true, "Vacation Fund"}),
     spanner.Insert("Account", accountColumns,
         []interface{}{20, parseTime("2020-02-18T05:44:20.12Z"), false, "Rainy Day Fund"}),
     spanner.Insert("Person", personColumns,
         []interface{}{1, "Alex", parseTime("1991-12-21T00:00:00.12Z"), "Australia", " Adelaide"}),
     spanner.Insert("Person", personColumns,
         []interface{}{2, "Dana", parseTime("1980-10-31T00:00:00.12Z"), "Czech_Republic", "Moravia"}),
     spanner.Insert("Person", personColumns,
         []interface{}{3, "Lee", parseTime("1986-12-07T00:00:00.12Z"), "India", "Kollam"}),
     spanner.Insert("AccountTransferAccount", transferColumns,
         []interface{}{7, 16, 300.0, parseTime("2020-08-29T15:28:58.12Z"), "304330008004315"}),
     spanner.Insert("AccountTransferAccount", transferColumns,
         []interface{}{7, 16, 100.0, parseTime("2020-10-04T16:55:05.12Z"), "304120005529714"}),
     spanner.Insert("AccountTransferAccount", transferColumns,
         []interface{}{16, 20, 300.0, parseTime("2020-09-25T02:36:14.12Z"), "103650009791820"}),
     spanner.Insert("AccountTransferAccount", transferColumns,
         []interface{}{20, 7, 500.0, parseTime("2020-10-04T16:55:05.12Z"), "304120005529714"}),
     spanner.Insert("AccountTransferAccount", transferColumns,
         []interface{}{20, 16, 200.0, parseTime("2020-10-17T03:59:40.12Z"), "302290001255747"}),
     spanner.Insert("PersonOwnAccount", ownColumns,
         []interface{}{1, 7, parseTime("2020-01-10T06:22:20.12Z")}),
     spanner.Insert("PersonOwnAccount", ownColumns,
         []interface{}{2, 20, parseTime("2020-01-27T17:55:09.12Z")}),
     spanner.Insert("PersonOwnAccount", ownColumns,
         []interface{}{3, 16, parseTime("2020-02-18T05:44:20.12Z")}),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
/** Class to contain sample Person data. */
static class Person {

  final long id;
  final String name;
  final Timestamp birthday;
  final String country;
  final String city;

  Person(long id, String name, Timestamp birthday, String country, String city) {
    this.id = id;
    this.name = name;
    this.birthday = birthday;
    this.country = country;
    this.city = city;
  }
}

/** Class to contain sample Account data. */
static class Account {

  final long id;
  final Timestamp createTime;
  final boolean isBlocked;
  final String nickName;

  Account(long id, Timestamp createTime, boolean isBlocked, String nickName) {
    this.id = id;
    this.createTime = createTime;
    this.isBlocked = isBlocked;
    this.nickName = nickName;
  }
}

/** Class to contain sample Transfer data. */
static class Transfer {

  final long id;
  final long toId;
  final double amount;
  final Timestamp createTime;
  final String orderNumber;

  Transfer(long id, long toId, double amount, Timestamp createTime, String orderNumber) {
    this.id = id;
    this.toId = toId;
    this.amount = amount;
    this.createTime = createTime;
    this.orderNumber = orderNumber;
  }
}

/** Class to contain sample Ownership data. */
static class Own {

  final long id;
  final long accountId;
  final Timestamp createTime;

  Own(long id, long accountId, Timestamp createTime) {
    this.id = id;
    this.accountId = accountId;
    this.createTime = createTime;
  }
}

static final List<Account> ACCOUNTS =
    Arrays.asList(
        new Account(
            7, Timestamp.parseTimestamp("2020-01-10T06:22:20.12Z"), false, "Vacation Fund"),
        new Account(
            16, Timestamp.parseTimestamp("2020-01-27T17:55:09.12Z"), true, "Vacation Fund"),
        new Account(
            20, Timestamp.parseTimestamp("2020-02-18T05:44:20.12Z"), false, "Rainy Day Fund"));

static final List<Person> PERSONS =
    Arrays.asList(
        new Person(
            1,
            "Alex",
            Timestamp.parseTimestamp("1991-12-21T00:00:00.12Z"),
            "Australia",
            " Adelaide"),
        new Person(
            2,
            "Dana",
            Timestamp.parseTimestamp("1980-10-31T00:00:00.12Z"),
            "Czech_Republic",
            "Moravia"),
        new Person(
            3, "Lee", Timestamp.parseTimestamp("1986-12-07T00:00:00.12Z"), "India", "Kollam"));

static final List<Transfer> TRANSFERS =
    Arrays.asList(
        new Transfer(
            7, 16, 300.0, Timestamp.parseTimestamp("2020-08-29T15:28:58.12Z"), "304330008004315"),
        new Transfer(
            7, 16, 100.0, Timestamp.parseTimestamp("2020-10-04T16:55:05.12Z"), "304120005529714"),
        new Transfer(
            16,
            20,
            300.0,
            Timestamp.parseTimestamp("2020-09-25T02:36:14.12Z"),
            "103650009791820"),
        new Transfer(
            20, 7, 500.0, Timestamp.parseTimestamp("2020-10-04T16:55:05.12Z"), "304120005529714"),
        new Transfer(
            20,
            16,
            200.0,
            Timestamp.parseTimestamp("2020-10-17T03:59:40.12Z"),
            "302290001255747"));

static final List<Own> OWNERSHIPS =
    Arrays.asList(
        new Own(1, 7, Timestamp.parseTimestamp("2020-01-10T06:22:20.12Z")),
        new Own(2, 20, Timestamp.parseTimestamp("2020-01-27T17:55:09.12Z")),
        new Own(3, 16, Timestamp.parseTimestamp("2020-02-18T05:44:20.12Z")));

static void insertData(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();
  for (Account account : ACCOUNTS) {
    mutations.add(
        Mutation.newInsertBuilder("Account")
            .set("id")
            .to(account.id)
            .set("create_time")
            .to(account.createTime)
            .set("is_blocked")
            .to(account.isBlocked)
            .set("nick_name")
            .to(account.nickName)
            .build());
  }
  for (Person person : PERSONS) {
    mutations.add(
        Mutation.newInsertBuilder("Person")
            .set("id")
            .to(person.id)
            .set("name")
            .to(person.name)
            .set("birthday")
            .to(person.birthday)
            .set("country")
            .to(person.country)
            .set("city")
            .to(person.city)
            .build());
  }
  for (Transfer transfer : TRANSFERS) {
    mutations.add(
        Mutation.newInsertBuilder("AccountTransferAccount")
            .set("id")
            .to(transfer.id)
            .set("to_id")
            .to(transfer.toId)
            .set("amount")
            .to(transfer.amount)
            .set("create_time")
            .to(transfer.createTime)
            .set("order_number")
            .to(transfer.orderNumber)
            .build());
  }
  for (Own own : OWNERSHIPS) {
    mutations.add(
        Mutation.newInsertBuilder("PersonOwnAccount")
            .set("id")
            .to(own.id)
            .set("account_id")
            .to(own.accountId)
            .set("create_time")
            .to(own.createTime)
            .build());
  }

  dbClient.write(mutations);
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def insert_data(instance_id, database_id):
    """Inserts sample data into the given database.

    The database and tables must already exist and can be created using
    `create_database_with_property_graph`.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.batch() as batch:
        batch.insert(
            table="Account",
            columns=("id", "create_time", "is_blocked", "nick_name"),
            values=[
                (7, "2020-01-10T06:22:20.12Z", False, "Vacation Fund"),
                (16, "2020-01-27T17:55:09.12Z", True, "Vacation Fund"),
                (20, "2020-02-18T05:44:20.12Z", False, "Rainy Day Fund"),
            ],
        )

        batch.insert(
            table="Person",
            columns=("id", "name", "birthday", "country", "city"),
            values=[
                (1, "Alex", "1991-12-21T00:00:00.12Z", "Australia", " Adelaide"),
                (2, "Dana", "1980-10-31T00:00:00.12Z", "Czech_Republic", "Moravia"),
                (3, "Lee", "1986-12-07T00:00:00.12Z", "India", "Kollam"),
            ],
        )

        batch.insert(
            table="AccountTransferAccount",
            columns=("id", "to_id", "amount", "create_time", "order_number"),
            values=[
                (7, 16, 300.0, "2020-08-29T15:28:58.12Z", "304330008004315"),
                (7, 16, 100.0, "2020-10-04T16:55:05.12Z", "304120005529714"),
                (16, 20, 300.0, "2020-09-25T02:36:14.12Z", "103650009791820"),
                (20, 7, 500.0, "2020-10-04T16:55:05.12Z", "304120005529714"),
                (20, 16, 200.0, "2020-10-17T03:59:40.12Z", "302290001255747"),
            ],
        )

        batch.insert(
            table="PersonOwnAccount",
            columns=("id", "account_id", "create_time"),
            values=[
                (1, 7, "2020-01-10T06:22:20.12Z"),
                (2, 20, "2020-01-27T17:55:09.12Z"),
                (3, 16, "2020-02-18T05:44:20.12Z"),
            ],
        )

    print("Inserted data.")
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
