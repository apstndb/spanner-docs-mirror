Insert data into a table with example data types by using mutations.

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void InsertDatatypesData(google::cloud::spanner::Client client) {
  namespace spanner = ::google::cloud::spanner;
  std::vector<absl::CivilDay> available_dates1 = {absl::CivilDay(2020, 12, 1),
                                                  absl::CivilDay(2020, 12, 2),
                                                  absl::CivilDay(2020, 12, 3)};
  std::vector<absl::CivilDay> available_dates2 = {absl::CivilDay(2020, 11, 1),
                                                  absl::CivilDay(2020, 11, 5),
                                                  absl::CivilDay(2020, 11, 15)};
  std::vector<absl::CivilDay> available_dates3 = {absl::CivilDay(2020, 10, 1),
                                                  absl::CivilDay(2020, 10, 7)};
  auto insert_venues =
      spanner::InsertMutationBuilder(
          "Venues", {"VenueId", "VenueName", "VenueInfo", "Capacity",
                     "AvailableDates", "LastContactDate", "OutdoorVenue",
                     "PopularityScore", "LastUpdateTime"})
          .EmplaceRow(4, "Venue 4", spanner::Bytes("Hello World 1"), 1800,
                      available_dates1, absl::CivilDay(2018, 9, 2), false,
                      0.85543, spanner::CommitTimestamp())
          .EmplaceRow(19, "Venue 19", spanner::Bytes("Hello World 2"), 6300,
                      available_dates2, absl::CivilDay(2019, 1, 15), true,
                      0.98716, spanner::CommitTimestamp())
          .EmplaceRow(42, "Venue 42", spanner::Bytes("Hello World 3"), 3000,
                      available_dates3, absl::CivilDay(2018, 10, 1), false,
                      0.72598, spanner::CommitTimestamp())
          .Build();

  auto commit_result = client.Commit(spanner::Mutations{insert_venues});
  if (!commit_result) throw std::move(commit_result).status();
  std::cout << "Insert was successful [spanner_insert_datatypes_data]\n";
}
```

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

public class InsertDataTypesDataAsyncSample
{
    public class Venue
    {
        public int VenueId { get; set; }
        public string VenueName { get; set; }
        public byte[] VenueInfo { get; set; }
        public int Capacity { get; set; }
        public List<DateTime> AvailableDates { get; set; }
        public DateTime LastContactDate { get; set; }
        public bool OutdoorVenue { get; set; }
        public float PopularityScore { get; set; }
    }

    public async Task InsertDataTypesDataAsync(string projectId, string instanceId, string databaseId)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";

        byte[] exampleBytes1 = Encoding.UTF8.GetBytes("Hello World 1");
        byte[] exampleBytes2 = Encoding.UTF8.GetBytes("Hello World 2");
        byte[] exampleBytes3 = Encoding.UTF8.GetBytes("Hello World 3");

        var availableDates1 = new List<DateTime>
        {
            DateTime.Parse("2020-12-01"),
            DateTime.Parse("2020-12-02"),
            DateTime.Parse("2020-12-03")
        };

        var availableDates2 = new List<DateTime>
        {
            DateTime.Parse("2020-11-01"),
            DateTime.Parse("2020-11-05"),
            DateTime.Parse("2020-11-15")
        };

        var availableDates3 = new List<DateTime>
        {
            DateTime.Parse("2020-10-01"),
            DateTime.Parse("2020-10-07")
        };
        List<Venue> venues = new List<Venue> {
            new Venue {
                VenueId = 4,
                VenueName = "Venue 4",
                VenueInfo = exampleBytes1,
                Capacity = 1800,
                AvailableDates = availableDates1,
                LastContactDate = DateTime.Parse("2018-09-02"),
                OutdoorVenue = false,
                PopularityScore = 0.85543f
            },
            new Venue {
                VenueId = 19,
                VenueName = "Venue 19",
                VenueInfo = exampleBytes2,
                Capacity = 6300,
                AvailableDates = availableDates2,
                LastContactDate = DateTime.Parse("2019-01-15"),
                OutdoorVenue = true,
                PopularityScore = 0.98716f
            },
            new Venue {
                VenueId = 42,
                VenueName = "Venue 42",
                VenueInfo = exampleBytes3,
                Capacity = 3000,
                AvailableDates = availableDates3,
                LastContactDate = DateTime.Parse("2018-10-01"),
                OutdoorVenue = false,
                PopularityScore = 0.72598f
            }};

        using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();

        await Task.WhenAll(venues.Select(venue =>
        {
            var cmd = connection.CreateInsertCommand("Venues",
            new SpannerParameterCollection {
                { "VenueId", SpannerDbType.Int64, venue.VenueId },
                { "VenueName", SpannerDbType.String, venue.VenueName },
                { "VenueInfo", SpannerDbType.Bytes, venue.VenueInfo },
                { "Capacity", SpannerDbType.Int64, venue.Capacity },
                { "AvailableDates", SpannerDbType.ArrayOf(SpannerDbType.Date), venue.AvailableDates },
                { "LastContactDate", SpannerDbType.Date, venue.LastContactDate },
                { "OutdoorVenue", SpannerDbType.Bool,  venue.OutdoorVenue },
                { "PopularityScore", SpannerDbType.Float64, venue.PopularityScore },
                { "LastUpdateTime", SpannerDbType.Timestamp, SpannerParameter.CommitTimestamp },
            });
            return cmd.ExecuteNonQueryAsync();
        }));
    }
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func writeDatatypesData(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 venueColumns := []string{"VenueId", "VenueName", "VenueInfo", "Capacity", "AvailableDates",
     "LastContactDate", "OutdoorVenue", "PopularityScore", "LastUpdateTime"}
 m := []*spanner.Mutation{
     spanner.InsertOrUpdate("Venues", venueColumns,
         []interface{}{4, "Venue 4", []byte("Hello World 1"), 1800,
             []string{"2020-12-01", "2020-12-02", "2020-12-03"},
             "2018-09-02", false, 0.85543, spanner.CommitTimestamp}),
     spanner.InsertOrUpdate("Venues", venueColumns,
         []interface{}{19, "Venue 19", []byte("Hello World 2"), 6300,
             []string{"2020-11-01", "2020-11-05", "2020-11-15"},
             "2019-01-15", true, 0.98716, spanner.CommitTimestamp}),
     spanner.InsertOrUpdate("Venues", venueColumns,
         []interface{}{42, "Venue 42", []byte("Hello World 3"), 3000,
             []string{"2020-10-01", "2020-10-07"}, "2018-10-01",
             false, 0.72598, spanner.CommitTimestamp}),
 }
 _, err = client.Apply(ctx, m)
 return err
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static Value availableDates1 =
    Value.dateArray(
        Arrays.asList(
            Date.parseDate("2020-12-01"),
            Date.parseDate("2020-12-02"),
            Date.parseDate("2020-12-03")));
static Value availableDates2 =
    Value.dateArray(
        Arrays.asList(
            Date.parseDate("2020-11-01"),
            Date.parseDate("2020-11-05"),
            Date.parseDate("2020-11-15")));
static Value availableDates3 =
    Value.dateArray(Arrays.asList(Date.parseDate("2020-10-01"), Date.parseDate("2020-10-07")));
static String exampleBytes1 = BaseEncoding.base64().encode("Hello World 1".getBytes());
static String exampleBytes2 = BaseEncoding.base64().encode("Hello World 2".getBytes());
static String exampleBytes3 = BaseEncoding.base64().encode("Hello World 3".getBytes());
static final List<Venue> VENUES =
    Arrays.asList(
        new Venue(
            4,
            "Venue 4",
            exampleBytes1,
            1800,
            availableDates1,
            "2018-09-02",
            false,
            0.85543f,
            new BigDecimal("215100.10"),
            Value.json(
                "[{\"name\":\"room 1\",\"open\":true},{\"name\":\"room 2\",\"open\":false}]")),
        new Venue(
            19,
            "Venue 19",
            exampleBytes2,
            6300,
            availableDates2,
            "2019-01-15",
            true,
            0.98716f,
            new BigDecimal("1200100.00"),
            Value.json("{\"rating\":9,\"open\":true}")),
        new Venue(
            42,
            "Venue 42",
            exampleBytes3,
            3000,
            availableDates3,
            "2018-10-01",
            false,
            0.72598f,
            new BigDecimal("390650.99"),
            Value.json(
                "{\"name\":null,"
                    + "\"open\":{\"Monday\":true,\"Tuesday\":false},"
                    + "\"tags\":[\"large\",\"airy\"]}")));
static void writeDatatypesData(DatabaseClient dbClient) {
  List<Mutation> mutations = new ArrayList<>();
  for (Venue venue : VENUES) {
    mutations.add(
        Mutation.newInsertBuilder("Venues")
            .set("VenueId")
            .to(venue.venueId)
            .set("VenueName")
            .to(venue.venueName)
            .set("VenueInfo")
            .to(venue.venueInfo)
            .set("Capacity")
            .to(venue.capacity)
            .set("AvailableDates")
            .to(venue.availableDates)
            .set("LastContactDate")
            .to(venue.lastContactDate)
            .set("OutdoorVenue")
            .to(venue.outdoorVenue)
            .set("PopularityScore")
            .to(venue.popularityScore)
            .set("Revenue")
            .to(venue.revenue)
            .set("VenueDetails")
            .to(venue.venueDetails)
            .set("LastUpdateTime")
            .to(Value.COMMIT_TIMESTAMP)
            .build());
  }
  dbClient.write(mutations);
}
```

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` javascript
// Imports the Google Cloud client library.
const {Spanner} = require('@google-cloud/spanner');

/**
 * TODO(developer): Uncomment the following lines before running the sample.
 */
// const projectId = 'my-project-id';
// const instanceId = 'my-instance';
// const databaseId = 'my-database';

// Creates a client.
const spanner = new Spanner({
  projectId: projectId,
});

// Gets a reference to a Cloud Spanner instance and database.
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// Instantiate Spanner table objects.
const venuesTable = database.table('Venues');
const exampleBytes1 = new Buffer.from('Hello World 1');
const exampleBytes2 = new Buffer.from('Hello World 2');
const exampleBytes3 = new Buffer.from('Hello World 3');
const availableDates1 = ['2020-12-01', '2020-12-02', '2020-12-03'];
const availableDates2 = ['2020-11-01', '2020-11-05', '2020-11-15'];
const availableDates3 = ['2020-10-01', '2020-10-07'];

// Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
// must be converted to strings before being inserted as INT64s.
const data = [
  {
    VenueId: '4',
    VenueName: 'Venue 4',
    VenueInfo: exampleBytes1,
    Capacity: '1800',
    AvailableDates: availableDates1,
    LastContactDate: '2018-09-02',
    OutdoorVenue: false,
    PopularityScore: Spanner.float(0.85543),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '19',
    VenueName: 'Venue 19',
    VenueInfo: exampleBytes2,
    Capacity: '6300',
    AvailableDates: availableDates2,
    LastContactDate: '2019-01-15',
    OutdoorVenue: true,
    PopularityScore: Spanner.float(0.98716),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
  {
    VenueId: '42',
    VenueName: 'Venue 42',
    VenueInfo: exampleBytes3,
    Capacity: '3000',
    AvailableDates: availableDates3,
    LastContactDate: '2018-10-01',
    OutdoorVenue: false,
    PopularityScore: Spanner.float(0.72598),
    LastUpdateTime: 'spanner.commit_timestamp()',
  },
];

// Inserts rows into the Venues table.
try {
  await venuesTable.insert(data);
  console.log('Inserted data.');
} catch (err) {
  console.error('ERROR:', err);
} finally {
  // Close the database when finished.
  database.close();
}
```

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Inserts sample data into a table with supported datatypes.
 *
 * The database and table must already exist and can be created using
 * `create_table_with_datatypes`.
 * Example:
 * ```
 * insert_data_with_datatypes($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function insert_data_with_datatypes(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->insertBatch('Venues', [
            [
                'VenueId' => 4,
                'VenueName' => 'Venue 4',
                'VenueInfo' => base64_encode('Hello World 1'),
                'Capacity' => 1800,
                'AvailableDates' => ['2020-12-01', '2020-12-02', '2020-12-03'],
                'LastContactDate' => '2018-09-02',
                'OutdoorVenue' => false,
                'PopularityScore' => 0.85543,
                'LastUpdateTime' => $spanner->commitTimestamp()
            ], [
                'VenueId' => 19,
                'VenueName' => 'Venue 19',
                'VenueInfo' => base64_encode('Hello World 2'),
                'Capacity' => 6300,
                'AvailableDates' => ['2020-11-01', '2020-11-05', '2020-11-15'],
                'LastContactDate' => '2019-01-15',
                'OutdoorVenue' => true,
                'PopularityScore' => 0.98716,
                'LastUpdateTime' => $spanner->commitTimestamp()
            ], [
                'VenueId' => 42,
                'VenueName' => 'Venue 42',
                'VenueInfo' => base64_encode('Hello World 3'),
                'Capacity' => 3000,
                'AvailableDates' => ['2020-10-01', '2020-10-07'],
                'LastContactDate' => '2018-10-01',
                'OutdoorVenue' => false,
                'PopularityScore' => 0.72598,
                'LastUpdateTime' => $spanner->commitTimestamp()
            ],
        ])
        ->commit();

    print('Inserted data.' . PHP_EOL);
}
````

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

exampleBytes1 = base64.b64encode("Hello World 1".encode())
exampleBytes2 = base64.b64encode("Hello World 2".encode())
exampleBytes3 = base64.b64encode("Hello World 3".encode())
available_dates1 = ["2020-12-01", "2020-12-02", "2020-12-03"]
available_dates2 = ["2020-11-01", "2020-11-05", "2020-11-15"]
available_dates3 = ["2020-10-01", "2020-10-07"]
with database.batch() as batch:
    batch.insert(
        table="Venues",
        columns=(
            "VenueId",
            "VenueName",
            "VenueInfo",
            "Capacity",
            "AvailableDates",
            "LastContactDate",
            "OutdoorVenue",
            "PopularityScore",
            "LastUpdateTime",
        ),
        values=[
            (
                4,
                "Venue 4",
                exampleBytes1,
                1800,
                available_dates1,
                "2018-09-02",
                False,
                0.85543,
                spanner.COMMIT_TIMESTAMP,
            ),
            (
                19,
                "Venue 19",
                exampleBytes2,
                6300,
                available_dates2,
                "2019-01-15",
                True,
                0.98716,
                spanner.COMMIT_TIMESTAMP,
            ),
            (
                42,
                "Venue 42",
                exampleBytes3,
                3000,
                available_dates3,
                "2018-10-01",
                False,
                0.72598,
                spanner.COMMIT_TIMESTAMP,
            ),
        ],
    )

print("Inserted data.")
```

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

# Get commit_timestamp
commit_timestamp = client.commit_timestamp

client.commit do |c|
  c.insert "Venues", [
    { VenueId: 4, VenueName: "Venue 4", VenueInfo: StringIO.new("Hello World 1"),
      Capacity: 1_800, AvailableDates: ["2020-12-01", "2020-12-02", "2020-12-03"],
      LastContactDate: "2018-09-02", OutdoorVenue: false, PopularityScore: 0.85543,
      LastUpdateTime: commit_timestamp },
    { VenueId: 19, VenueName: "Venue 19", VenueInfo: StringIO.new("Hello World 2"),
      Capacity: 6_300, AvailableDates: ["2020-11-01", "2020-11-05", "2020-11-15"],
      LastContactDate: "2019-01-15", OutdoorVenue: true, PopularityScore: 0.98716,
      LastUpdateTime: commit_timestamp },
    { VenueId: 42, VenueName: "Venue 42", VenueInfo: StringIO.new("Hello World 3"),
      Capacity: 3_000, AvailableDates: ["2020-10-01", "2020-10-07"],
      LastContactDate: "2018-10-01", OutdoorVenue: false, PopularityScore: 0.72598,
      LastUpdateTime: commit_timestamp }
  ]
end

puts "Inserted data"
```

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
