**Preview — Data agents**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

For information about access to this release, see the [access request page](https://forms.gle/pJByTWfenZAWbaXo7) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This tutorial describes how to set up and use a data agent in Spanner using the Google Cloud console and integrate it with your application. You learn how to build the agent context file, create a data agent that uses the context, use [MCP Toolbox](https://github.com/googleapis/genai-toolbox) to call the [QueryData API](/gemini/docs/conversational-analytics-api/data-agent-authored-context-databases) to generate SQL queries for natural language questions, and finally integrate it with your application.

For more information, see [Data agents overview](/spanner/docs/data-agent-overview) .

## Objectives

  - Create and populate tables.
  - Build agent context with Gemini CLI and MCP toolbox.
  - Create a data agent and upload context.
  - Inspect the agent and generate SQL queries in Studio.
  - Integrate the agent with your application using [Gemini Data Analytics QueryData](https://googleapis.github.io/genai-toolbox/resources/tools/cloudgda/cloud-gda-query/) tool in MCP Toolbox.

## Costs

In this document, you use the following billable components of Google Cloud:

  - [Spanner](https://cloud.google.com/spanner/pricing)
  - [Gemini Data Analytics API](https://cloud.google.com/gemini/pricing)
  - [Gemini for Google Cloud API](https://cloud.google.com/gemini/pricing)

To generate a cost estimate based on your projected usage, use the [pricing calculator](https://cloud.google.com/products/calculator) .

New Google Cloud users might be eligible for a [free trial](https://cloud.google.com/free) .

When you finish the tasks described in this document, avoid continued billing by deleting the resources you created. For more information, see [Clean up](#clean-up) .

## Before you begin

Complete the following prerequisites before creating an agent.

**Note:** Ensure that you enable public IP for your Spanner instance.

### Enable required services

Enable the following services for your project:

  - [Gemini Data Analytics API](https://console.cloud.google.com/apis/library/geminidataanalytics.googleapis.com)
  - [Gemini for Google Cloud API](https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com)

### Prepare a Spanner instance

  - Make sure that a Spanner instance is available. For more information, see [Create an instance](/spanner/docs/create-manage-instances) .
  - Ensure that you create a database in your instance where you will create the tables. For more information, see [Create a database on the Spanner instance](/spanner/docs/create-manage-databases#create-database)

### Required roles and permissions

  - Add an IAM user or service account to the cluster. For more information, see [Apply IAM roles](/spanner/docs/grant-permissions) .
  - Grant the `  spanner.databaseReader  ` roles to the IAM user at the project level. For more information, see [Add IAM policy binding for a project](/sdk/gcloud/reference/projects/add-iam-policy-binding) .
  - [Grant roles and permissions](/spanner/docs/grant-permissions#project-level_permissions) to the IAM user at the project-level for the required databases.

## Create the `     flights    ` and `     airports    ` schema and tables

In this section, you create the `  flights  ` and `  airports  ` database for this tutorial.

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  Click **New SQL editor tab** or **New tab** to open a new tab.

5.  Create the `  airports  ` table and schema:
    
    ``` text
    CREATE TABLE IF NOT EXISTS airports (
      id INT64 PRIMARY KEY,
      iata STRING(MAX),
      name STRING(MAX),
      city STRING(MAX),
      country STRING(MAX)
    );
    ```

6.  Create the `  flights  ` table and schema:
    
    ``` text
    CREATE TABLE flights (
      id INT64 NOT NULL,
      airline STRING(10),
      flight_number INT64,
      departure_airport STRING(5),
      arrival_airport STRING(5),
      departure_time TIMESTAMP,
      arrival_time TIMESTAMP,
      departure_gate STRING(10),
      arrival_gate STRING(10)
    ) PRIMARY KEY (id);
    ```

## Populate the `     flights    ` and `     airport    ` table

In this section, you populate the `  flights  ` and `  airports  ` table using the provided SQL scripts.

1.  Populate the `  airports  ` table.
    
    #### See SQL script
    
    ``` text
    -- Insert Data into Airports
    INSERT INTO airports (id, iata, name, city, country) VALUES
    (3249, 'BOS', 'General Edward Lawrence Logan International Airport', 'Boston', 'United States'),
    (3257, 'OGG', 'Kahului Airport', 'Kahului', 'United States'),
    (3259, 'MCI', 'Kansas City International Airport', 'Kansas City', 'United States'),
    (3263, 'PHX', 'Phoenix Sky Harbor International Airport', 'Phoenix', 'United States'),
    (3270, 'SFO', 'San Francisco International Airport', 'San Francisco', 'United States'),
    (3285, 'LAX', 'Los Angeles International Airport', 'Los Angeles', 'United States'),
    (3287, 'CLE', 'Cleveland Hopkins International Airport', 'Cleveland', 'United States'),
    (3295, 'EWR', 'Newark Liberty International Airport', 'Newark', 'United States'),
    (3296, 'BOI', 'Boise Air Terminal/Gowen Field', 'Boise', 'United States'),
    (3303, 'DAL', 'Dallas Love Field', 'Dallas', 'United States'),
    (3315, 'KOA', 'Ellison Onizuka Kona International At Keahole Airport', 'Kona', 'United States'),
    (3321, 'DCA', 'Ronald Reagan Washington National Airport', 'Washington', 'United States'),
    (3336, 'SLC', 'Salt Lake City International Airport', 'Salt Lake City', 'United States'),
    (3350, 'IAH', 'George Bush Intercontinental Houston Airport', 'Houston', 'United States'),
    (3370, 'PIT', 'Pittsburgh International Airport', 'Pittsburgh', 'United States'),
    (3376, 'MIA', 'Miami International Airport', 'Miami', 'United States'),
    (3377, 'SEA', 'Seattle Tacoma International Airport', 'Seattle', 'United States'),
    (3382, 'LGB', 'Long Beach /Daugherty Field/ Airport', 'Long Beach', 'United States'),
    (3385, 'IND', 'Indianapolis International Airport', 'Indianapolis', 'United States'),
    (3402, 'LIH', 'Lihue Airport', 'Lihue', 'United States'),
    (3421, 'SAT', 'San Antonio International Airport', 'San Antonio', 'United States'),
    (3426, 'RDU', 'Raleigh Durham International Airport', 'Raleigh-durham', 'United States'),
    (3436, 'TUS', 'Tucson International Airport', 'Tucson', 'United States'),
    (3444, 'BUR', 'Bob Hope Airport', 'Burbank', 'United States'),
    (3445, 'DTW', 'Detroit Metropolitan Wayne County Airport', 'Detroit', 'United States'),
    (3470, 'DFW', 'Dallas Fort Worth International Airport', 'Dallas-Fort Worth', 'United States'),
    (3473, 'AUS', 'Austin Bergstrom International Airport', 'Austin', 'United States'),
    (3478, 'STL', 'St Louis Lambert International Airport', 'St. Louis', 'United States'),
    (3482, 'ATL', 'Hartsfield Jackson Atlanta International Airport', 'Atlanta', 'United States'),
    (3487, 'FAT', 'Fresno Yosemite International Airport', 'Fresno', 'United States'),
    (3514, 'IAD', 'Washington Dulles International Airport', 'Washington', 'United States'),
    (3517, 'MKE', 'General Mitchell International Airport', 'Milwaukee', 'United States'),
    (3520, 'PDX', 'Portland International Airport', 'Portland', 'United States'),
    (3528, 'HNL', 'Daniel K Inouye International Airport', 'Honolulu', 'United States'),
    (3531, 'SAN', 'San Diego International Airport', 'San Diego', 'United States'),
    (3534, 'ONT', 'Ontario International Airport', 'Ontario', 'United States'),
    (3547, 'MDW', 'Chicago Midway International Airport', 'Chicago', 'United States'),
    (3551, 'DEN', 'Denver International Airport', 'Denver', 'United States'),
    (3552, 'PHL', 'Philadelphia International Airport', 'Philadelphia', 'United States'),
    (3597, 'JFK', 'John F Kennedy International Airport', 'New York', 'United States'),
    (3607, 'RNO', 'Reno Tahoe International Airport', 'Reno', 'United States'),
    (3617, 'SMF', 'Sacramento International Airport', 'Sacramento', 'United States'),
    (3619, 'COS', 'City of Colorado Springs Municipal Airport', 'Colorado Springs', 'United States'),
    (3639, 'PSP', 'Palm Springs International Airport', 'Palm Springs', 'United States'),
    (3649, 'BWI', 'Baltimore/Washington International Thurgood Marshall Airport', 'Baltimore', 'United States'),
    (3658, 'MSP', 'Minneapolis-St Paul International/Wold-Chamberlain Airport', 'Minneapolis', 'United States'),
    (3661, 'MSY', 'Louis Armstrong New Orleans International Airport', 'New Orleans', 'United States'),
    (3667, 'SNA', 'John Wayne Airport-Orange County Airport', 'Santa Ana', 'United States'),
    (3676, 'CLT', 'Charlotte Douglas International Airport', 'Charlotte', 'United States'),
    (3677, 'LAS', 'McCarran International Airport', 'Las Vegas', 'United States'),
    (3678, 'MCO', 'Orlando International Airport', 'Orlando', 'United States'),
    (3742, 'MRY', 'Monterey Peninsula Airport', 'Monterey', 'United States'),
    (3743, 'SBA', 'Santa Barbara Municipal Airport', 'Santa Barbara', 'United States'),
    (3875, 'RDD', 'Redding Municipal Airport', 'Redding', 'United States'),
    (3876, 'EUG', 'Mahlon Sweet Field', 'Eugene', 'United States'),
    (3878, 'MFR', 'Rogue Valley International Medford Airport', 'Medford', 'United States'),
    (3880, 'RDM', 'Roberts Field', 'Redmond-Bend', 'United States'),
    (4111, 'ACV', 'California Redwood Coast-Humboldt County Airport', 'Arcata CA', 'United States'),
    (4419, 'CEC', 'Jack Mc Namara Field Airport', 'Crescent City', 'United States'),
    (4460, 'SBP', 'San Luis County Regional Airport', 'San Luis Obispo', 'United States'),
    (4768, 'PSC', 'Tri Cities Airport', 'Pasco', 'United States'),
    (5088, 'SUN', 'Friedman Memorial Airport', 'Hailey', 'United States'),
    (5305, 'ASE', 'Aspen-Pitkin Co/Sardy Field', 'Aspen', 'United States'),
    (5362, 'MMH', 'Mammoth Yosemite Airport', 'Mammoth Lakes', 'United States');
    ```

2.  Populate the `  flights  ` table.
    
    #### See SQL script
    
    ``` text
    -- Insert Data into Flights
    INSERT INTO flights (id, airline, flight_number, departure_airport, arrival_airport, departure_time, arrival_time, departure_gate, arrival_gate) VALUES
    (0, 'UA', 1532, 'SFO', 'DEN', '2025-01-01 05:50:00', '2025-01-01 09:23:00', 'E49', 'D6'),
    (1, 'UA', 1158, 'SFO', 'ORD', '2025-01-01 05:57:00', '2025-01-01 12:13:00', 'C38', 'D30'),
    (2, 'F9', 1057, 'IAH', 'SFO', '2025-01-01 05:52:00', '2025-01-01 08:14:00', 'E33', 'E43'),
    (3, 'OO', 6403, 'MFR', 'SFO', '2025-01-01 05:57:00', '2025-01-01 07:23:00', 'A6', 'E50'),
    (4, 'OO', 5628, 'RNO', 'SFO', '2025-01-01 06:19:00', '2025-01-01 07:37:00', 'A37', 'B29'),
    (5, 'CY', 922, 'SFO', 'LAX', '2025-01-01 06:38:00', '2025-01-01 07:49:00', 'D20', 'E45'),
    (6, 'AA', 338, 'LAX', 'SFO', '2025-01-01 06:44:00', '2025-01-01 08:03:00', 'E20', 'A45'),
    (7, 'UA', 1195, 'MCO', 'SFO', '2025-01-01 12:28:00', '2025-01-01 15:06:00', 'D22', 'B36'),
    (8, 'OO', 5632, 'SBA', 'SFO', '2025-01-01 06:56:00', '2025-01-01 08:39:00', 'E19', 'A49'),
    (9, 'UA', 300, 'CLE', 'SFO', '2025-01-01 07:15:00', '2025-01-01 09:22:00', 'E8', 'C28'),
    (10, 'UA', 706, 'SEA', 'SFO', '2025-01-01 07:14:00', '2025-01-01 09:07:00', 'A32', 'B26'),
    (11, 'AA', 167, 'MIA', 'SFO', '2025-01-01 07:47:00', '2025-01-01 10:51:00', 'A23', 'E4'),
    (12, 'OO', 5441, 'SFO', 'SBP', '2025-01-01 07:08:00', '2025-01-01 08:12:00', 'E21', 'D24'),
    (13, 'UA', 616, 'SFO', 'ORD', '2025-01-01 07:14:00', '2025-01-01 13:24:00', 'A11', 'D8'),
    (14, 'AA', 24, 'SFO', 'JFK', '2025-01-01 07:14:00', '2025-01-01 15:46:00', 'D4', 'C15'),
    (15, 'UA', 72, 'HNL', 'SFO', '2025-01-01 07:28:00', '2025-01-01 14:22:00', 'B33', 'D49'),
    (16, 'AS', 244, 'PDX', 'SFO', '2025-01-01 07:24:00', '2025-01-01 09:04:00', 'E46', 'B8'),
    (17, 'OO', 6307, 'MSP', 'SFO', '2025-01-01 07:29:00', '2025-01-01 09:32:00', 'C49', 'C36'),
    (18, 'OO', 4975, 'STL', 'SFO', '2025-01-01 07:31:00', '2025-01-01 09:58:00', 'C43', 'E16'),
    (19, 'AA', 23, 'DFW', 'SFO', '2025-01-01 08:41:00', '2025-01-01 10:34:00', 'E13', 'E19'),
    (20, 'B6', 434, 'SFO', 'BOS', '2025-01-01 08:00:00', '2025-01-01 16:23:00', 'D16', 'E46'),
    (21, 'F9', 657, 'DEN', 'SFO', '2025-01-01 07:58:00', '2025-01-01 09:13:00', 'E41', 'E28'),
    (22, 'OO', 6464, 'COS', 'SFO', '2025-01-01 07:56:00', '2025-01-01 09:45:00', 'D49', 'E17'),
    (23, 'UA', 1235, 'IAD', 'SFO', '2025-01-01 08:28:00', '2025-01-01 11:15:00', 'A22', 'D19'),
    (24, 'OO', 6527, 'EUG', 'SFO', '2025-01-01 07:54:00', '2025-01-01 09:36:00', 'D9', 'B26'),
    (25, 'AA', 242, 'SFO', 'ORD', '2025-01-01 08:18:00', '2025-01-01 14:26:00', 'E30', 'C1'),
    (26, 'UA', 1739, 'SFO', 'EWR', '2025-01-01 08:22:00', '2025-01-01 16:42:00', 'C13', 'D45'),
    (27, 'B6', 133, 'BOS', 'SFO', '2025-01-01 08:34:00', '2025-01-01 12:30:00', 'D49', 'C16'),
    (28, 'OO', 6336, 'SFO', 'RNO', '2025-01-01 08:32:00', '2025-01-01 09:29:00', 'B45', 'A2'),
    (29, 'UA', 567, 'IAD', 'SFO', '2025-01-01 10:47:00', '2025-01-01 13:39:00', 'C27', 'B24'),
    (30, 'OO', 5223, 'SLC', 'SFO', '2025-01-01 08:50:00', '2025-01-01 09:55:00', 'C38', 'C11'),
    (31, 'US', 1784, 'SFO', 'CLT', '2025-01-01 08:47:00', '2025-01-01 16:32:00', 'C42', 'C31'),
    (32, 'WN', 1579, 'ATL', 'SFO', '2025-01-01 08:48:00', '2025-01-01 10:55:00', 'C47', 'B5'),
    (33, 'OO', 6396, 'BOI', 'SFO', '2025-01-01 08:56:00', '2025-01-01 09:43:00', 'E20', 'A6'),
    (34, 'AA', 2293, 'DFW', 'SFO', '2025-01-01 08:57:00', '2025-01-01 11:08:00', 'D41', 'C37'),
    (35, 'DL', 1106, 'SFO', 'MSP', '2025-01-01 09:06:00', '2025-01-01 14:56:00', 'E38', 'B30'),
    (36, 'DL', 1631, 'SFO', 'ATL', '2025-01-01 09:00:00', '2025-01-01 16:26:00', 'B13', 'B45'),
    (37, 'OO', 5427, 'SFO', 'SMF', '2025-01-01 09:06:00', '2025-01-01 10:00:00', 'C34', 'C25'),
    (38, 'CY', 352, 'SFO', 'BOS', '2025-01-01 09:25:00', '2025-01-01 17:51:00', 'A46', 'E42'),
    (39, 'UA', 1239, 'LAX', 'SFO', '2025-01-01 09:59:00', '2025-01-01 11:19:00', 'A35', 'C2'),
    (40, 'UA', 1285, 'SFO', 'IAH', '2025-01-01 09:41:00', '2025-01-01 15:20:00', 'A39', 'E10'),
    (41, 'CY', 753, 'SEA', 'SFO', '2025-01-01 09:58:00', '2025-01-01 11:53:00', 'C40', 'E35'),
    (42, 'CY', 956, 'SFO', 'SAN', '2025-01-01 09:57:00', '2025-01-01 11:30:00', 'B10', 'E44'),
    (43, 'OO', 5196, 'PDX', 'SFO', '2025-01-01 09:53:00', '2025-01-01 11:39:00', 'D13', 'C34'),
    (44, 'WN', 2346, 'SFO', 'LAX', '2025-01-01 09:59:00', '2025-01-01 11:17:00', 'C24', 'E45'),
    (45, 'OO', 6400, 'EUG', 'SFO', '2025-01-01 10:06:00', '2025-01-01 11:27:00', 'E46', 'D20'),
    (46, 'CY', 23, 'JFK', 'SFO', '2025-01-01 10:21:00', '2025-01-01 14:09:00', 'D2', 'B21'),
    (47, 'B6', 534, 'SFO', 'BOS', '2025-01-01 10:33:00', '2025-01-01 18:55:00', 'A49', 'A5'),
    (48, 'UA', 1727, 'BOS', 'SFO', '2025-01-01 10:34:00', '2025-01-01 13:57:00', 'D43', 'A31'),
    (49, 'WN', 4391, 'LAS', 'SFO', '2025-01-01 10:49:00', '2025-01-01 12:17:00', 'A40', 'C49'),
    (50, 'OO', 5335, 'CEC', 'SFO', '2025-01-01 10:25:00', '2025-01-01 12:04:00', 'C8', 'C6'),
    (51, 'DL', 1680, 'ATL', 'SFO', '2025-01-01 10:43:00', '2025-01-01 12:45:00', 'E49', 'E24'),
    (52, 'OO', 6283, 'SFO', 'MCI', '2025-01-01 10:38:00', '2025-01-01 17:01:00', 'C32', 'B12'),
    (53, 'OO', 5260, 'SFO', 'FAT', '2025-01-01 10:37:00', '2025-01-01 11:39:00', 'B46', 'B43'),
    (54, 'OO', 6249, 'LAS', 'SFO', '2025-01-01 10:48:00', '2025-01-01 12:19:00', 'D11', 'A49'),
    (55, 'AA', 23, 'SFO', 'DFW', '2025-01-01 11:23:00', '2025-01-01 17:00:00', 'B31', 'A11'),
    (56, 'UA', 753, 'SFO', 'AUS', '2025-01-01 10:59:00', '2025-01-01 16:44:00', 'B18', 'D6'),
    (57, 'WN', 4855, 'SFO', 'MKE', '2025-01-01 11:08:00', '2025-01-01 17:33:00', 'D3', 'D40'),
    (58, 'DL', 1845, 'SLC', 'SFO', '2025-01-01 11:29:00', '2025-01-01 12:06:00', 'A22', 'D2'),
    (59, 'UA', 720, 'SFO', 'DEN', '2025-01-01 12:46:00', '2025-01-01 16:19:00', 'C41', 'E39'),
    (60, 'UA', 1257, 'IAH', 'SFO', '2025-01-01 11:30:00', '2025-01-01 13:19:00', 'A26', 'D2'),
    (61, 'WN', 2509, 'SFO', 'LAX', '2025-01-01 11:34:00', '2025-01-01 13:03:00', 'C42', 'D35'),
    (62, 'UA', 73, 'EWR', 'SFO', '2025-01-01 12:11:00', '2025-01-01 15:24:00', 'C5', 'E10'),
    (63, 'UA', 364, 'PHX', 'SFO', '2025-01-01 11:56:00', '2025-01-01 13:11:00', 'A39', 'D26'),
    (64, 'AA', 221, 'DFW', 'SFO', '2025-01-01 11:55:00', '2025-01-01 13:30:00', 'A47', 'B22'),
    (65, 'AA', 2293, 'SFO', 'DFW', '2025-01-01 12:04:00', '2025-01-01 17:36:00', 'C43', 'E30'),
    (66, 'OO', 5199, 'SFO', 'SLC', '2025-01-01 12:09:00', '2025-01-01 14:58:00', 'A40', 'E42'),
    (67, 'OO', 6280, 'SLC', 'SFO', '2025-01-01 12:13:00', '2025-01-01 13:09:00', 'E16', 'E36'),
    (68, 'OO', 6317, 'RNO', 'SFO', '2025-01-01 12:05:00', '2025-01-01 13:09:00', 'C23', 'A31'),
    (69, 'UA', 864, 'SFO', 'EWR', '2025-01-01 12:14:00', '2025-01-01 20:38:00', 'C30', 'E37'),
    (70, 'OO', 5400, 'ONT', 'SFO', '2025-01-01 12:25:00', '2025-01-01 13:40:00', 'A48', 'E19'),
    (71, 'UA', 236, 'HNL', 'SFO', '2025-01-01 12:40:00', '2025-01-01 19:36:00', 'A34', 'C12'),
    (72, 'OO', 6343, 'SNA', 'SFO', '2025-01-01 12:37:00', '2025-01-01 13:57:00', 'A15', 'D26'),
    (73, 'UA', 1148, 'SFO', 'IAD', '2025-01-01 13:10:00', '2025-01-01 21:02:00', 'D24', 'A23'),
    (74, 'DL', 432, 'JFK', 'SFO', '2025-01-01 13:00:00', '2025-01-01 16:28:00', 'E3', 'A35'),
    (75, 'B6', 1435, 'SFO', 'LGB', '2025-01-01 13:33:00', '2025-01-01 14:47:00', 'B24', 'C12'),
    (76, 'OO', 6408, 'SFO', 'EUG', '2025-01-01 13:43:00', '2025-01-01 15:10:00', 'B47', 'A35'),
    (77, 'UA', 645, 'PDX', 'SFO', '2025-01-01 13:27:00', '2025-01-01 15:02:00', 'D7', 'E4'),
    (78, 'WN', 1812, 'SFO', 'SAN', '2025-01-01 14:32:00', '2025-01-01 15:51:00', 'B35', 'D48'),
    (79, 'OO', 6409, 'SFO', 'PDX', '2025-01-01 13:42:00', '2025-01-01 15:27:00', 'C10', 'B17'),
    (80, 'UA', 358, 'AUS', 'SFO', '2025-01-01 14:31:00', '2025-01-01 15:48:00', 'E33', 'D21'),
    (81, 'UA', 1728, 'OGG', 'SFO', '2025-01-01 14:02:00', '2025-01-01 20:38:00', 'A30', 'C29'),
    (82, 'UA', 816, 'SEA', 'SFO', '2025-01-01 13:44:00', '2025-01-01 15:38:00', 'A36', 'A9'),
    (83, 'UA', 724, 'HNL', 'SFO', '2025-01-01 13:59:00', '2025-01-01 20:44:00', 'A14', 'C9'),
    (84, 'UA', 1733, 'SFO', 'DEN', '2025-01-01 14:38:00', '2025-01-01 18:35:00', 'B27', 'E32'),
    (85, 'OO', 5543, 'SFO', 'BOI', '2025-01-01 14:27:00', '2025-01-01 17:10:00', 'E10', 'B16'),
    (86, 'UA', 1761, 'IAH', 'SFO', '2025-01-01 14:55:00', '2025-01-01 16:45:00', 'B2', 'E20'),
    (87, 'UA', 277, 'DEN', 'SFO', '2025-01-01 14:44:00', '2025-01-01 16:04:00', 'B47', 'B13'),
    (88, 'AA', 1213, 'DFW', 'SFO', '2025-01-01 15:14:00', '2025-01-01 16:45:00', 'D11', 'C31'),
    (89, 'US', 407, 'PHX', 'SFO', '2025-01-01 15:03:00', '2025-01-01 16:00:00', 'C7', 'D19'),
    (90, 'OO', 5367, 'SFO', 'RDM', '2025-01-01 14:52:00', '2025-01-01 16:23:00', 'A5', 'E40'),
    (91, 'OO', 5454, 'SMF', 'SFO', '2025-01-01 15:46:00', '2025-01-01 16:33:00', 'A20', 'A40'),
    (92, 'WN', 2687, 'SFO', 'LAX', '2025-01-01 15:15:00', '2025-01-01 16:35:00', 'D41', 'B37'),
    (93, 'CY', 932, 'SFO', 'LAX', '2025-01-01 15:31:00', '2025-01-01 16:51:00', 'A19', 'E46'),
    (94, 'WN', 3236, 'SFO', 'SAN', '2025-01-01 15:56:00', '2025-01-01 17:23:00', 'B29', 'B32'),
    (95, 'UA', 404, 'IAH', 'SFO', '2025-01-01 15:56:00', '2025-01-01 17:52:00', 'B15', 'B42'),
    (96, 'CY', 933, 'LAX', 'SFO', '2025-01-01 15:42:00', '2025-01-01 16:51:00', 'E16', 'E7'),
    (97, 'US', 596, 'SFO', 'PHX', '2025-01-01 15:51:00', '2025-01-01 18:52:00', 'B25', 'E38'),
    (98, 'AA', 379, 'SFO', 'DFW', '2025-01-01 16:04:00', '2025-01-01 21:55:00', 'A30', 'D49'),
    (99, 'OO', 6280, 'SFO', 'SNA', '2025-01-01 16:52:00', '2025-01-01 18:19:00', 'B5', 'C40'),
    (100, 'UA', 735, 'DEN', 'SFO', '2025-01-01 17:34:00', '2025-01-01 19:09:00', 'C31', 'B15'),
    (101, 'OO', 5553, 'PDX', 'SFO', '2025-01-01 16:00:00', '2025-01-01 17:36:00', 'E9', 'D27'),
    (102, 'UA', 73, 'SFO', 'HNL', '2025-01-01 16:13:00', '2025-01-01 19:52:00', 'C37', 'B37'),
    (103, 'UA', 587, 'SEA', 'SFO', '2025-01-01 16:25:00', '2025-01-01 18:27:00', 'B7', 'B23'),
    (104, 'OO', 5363, 'SFO', 'ONT', '2025-01-01 16:09:00', '2025-01-01 17:28:00', 'D21', 'E29'),
    (105, 'OO', 6386, 'SFO', 'BOI', '2025-01-01 16:12:00', '2025-01-01 18:58:00', 'D48', 'E33'),
    (106, 'CY', 27, 'JFK', 'SFO', '2025-01-01 16:28:00', '2025-01-01 20:19:00', 'C6', 'E14'),
    (107, 'OO', 5190, 'SFO', 'ASE', '2025-01-01 16:24:00', '2025-01-01 19:42:00', 'C50', 'B44'),
    (108, 'CY', 746, 'SFO', 'SEA', '2025-01-01 16:40:00', '2025-01-01 18:32:00', 'D35', 'C24'),
    (109, 'UA', 1640, 'SFO', 'ORD', '2025-01-01 17:01:00', '2025-01-01 23:02:00', 'E27', 'C24'),
    (110, 'OO', 6221, 'MCI', 'SFO', '2025-01-01 17:28:00', '2025-01-01 19:08:00', 'E2', 'C36'),
    (111, 'UA', 486, 'SFO', 'LAS', '2025-01-01 16:47:00', '2025-01-01 18:16:00', 'C42', 'B47'),
    (112, 'CY', 2, 'DCA', 'SFO', '2025-01-01 17:01:00', '2025-01-01 20:07:00', 'E43', 'A33'),
    (113, 'B6', 1436, 'LGB', 'SFO', '2025-01-01 16:59:00', '2025-01-01 18:35:00', 'B12', 'B22'),
    (114, 'WN', 694, 'PHX', 'SFO', '2025-01-01 17:11:00', '2025-01-01 18:05:00', 'B44', 'B18'),
    (115, 'OO', 5635, 'MMH', 'SFO', '2025-01-01 17:00:00', '2025-01-01 18:01:00', 'A25', 'D7'),
    (116, 'AS', 341, 'PSP', 'SFO', '2025-01-01 18:25:00', '2025-01-01 19:46:00', 'A11', 'E18'),
    (117, 'OO', 6364, 'PSP', 'SFO', '2025-01-01 18:01:00', '2025-01-01 19:31:00', 'A14', 'B47'),
    (118, 'OO', 5418, 'SBP', 'SFO', '2025-01-01 17:17:00', '2025-01-01 18:15:00', 'B24', 'D17'),
    (119, 'AA', 197, 'SFO', 'ORD', '2025-01-01 17:21:00', '2025-01-01 23:33:00', 'D25', 'E49'),
    (120, 'CY', 720, 'SFO', 'DAL', '2025-01-01 18:25:00', '2025-01-01 23:43:00', 'B24', 'A2'),
    (121, 'OO', 6268, 'FAT', 'SFO', '2025-01-01 17:33:00', '2025-01-01 18:39:00', 'E35', 'D25'),
    (122, 'OO', 5615, 'SFO', 'EUG', '2025-01-01 17:45:00', '2025-01-01 19:18:00', 'B11', 'A34'),
    (123, 'CY', 960, 'SFO', 'SAN', '2025-01-01 17:55:00', '2025-01-01 19:23:00', 'D23', 'E44'),
    (124, 'AA', 1393, 'DFW', 'SFO', '2025-01-01 17:51:00', '2025-01-01 19:25:00', 'E11', 'A41'),
    (125, 'OO', 6352, 'BOI', 'SFO', '2025-01-01 17:58:00', '2025-01-01 18:46:00', 'B31', 'D23'),
    (126, 'DL', 1151, 'SFO', 'HNL', '2025-01-01 17:53:00', '2025-01-01 21:34:00', 'E29', 'B45'),
    (127, 'OO', 5227, 'SFO', 'SAN', '2025-01-01 19:07:00', '2025-01-01 20:31:00', 'A35', 'B38'),
    (128, 'CY', 941, 'LAX', 'SFO', '2025-01-01 18:19:00', '2025-01-01 19:34:00', 'C33', 'B44'),
    (129, 'UA', 1044, 'EWR', 'SFO', '2025-01-01 19:19:00', '2025-01-01 22:53:00', 'D50', 'D26'),
    (130, 'CY', 211, 'ORD', 'SFO', '2025-01-01 18:41:00', '2025-01-01 21:14:00', 'D13', 'E45'),
    (131, 'UA', 685, 'ORD', 'SFO', '2025-01-01 18:51:00', '2025-01-01 21:04:00', 'D26', 'D45'),
    (132, 'UA', 370, 'SFO', 'SEA', '2025-01-01 18:49:00', '2025-01-01 20:41:00', 'B6', 'E20'),
    (133, 'UA', 1433, 'IAH', 'SFO', '2025-01-01 19:45:00', '2025-01-01 21:39:00', 'C42', 'C48'),
    (134, 'UA', 1172, 'DEN', 'SFO', '2025-01-01 20:19:00', '2025-01-01 22:37:00', 'E31', 'D43'),
    (135, 'CY', 593, 'PSP', 'SFO', '2025-01-01 19:09:00', '2025-01-01 20:26:00', 'C23', 'C43'),
    (136, 'UA', 1564, 'SFO', 'ORD', '2025-01-01 19:14:00', '2025-01-02 01:14:00', 'E3', 'C48'),
    (137, 'UA', 741, 'ORD', 'SFO', '2025-01-01 19:18:00', '2025-01-01 21:58:00', 'A25', 'B38'),
    (138, 'WN', 600, 'SFO', 'SNA', '2025-01-01 19:36:00', '2025-01-01 20:53:00', 'D21', 'D11'),
    (139, 'F9', 664, 'SFO', 'DEN', '2025-01-01 19:31:00', '2025-01-01 23:13:00', 'E11', 'E12'),
    (140, 'OO', 4995, 'LAS', 'SFO', '2025-01-01 19:18:00', '2025-01-01 20:58:00', 'B42', 'B49'),
    (141, 'CY', 717, 'DAL', 'SFO', '2025-01-01 19:31:00', '2025-01-01 21:02:00', 'A42', 'D32'),
    (142, 'UA', 1243, 'SFO', 'DEN', '2025-01-01 19:38:00', '2025-01-01 23:15:00', 'B34', 'A3'),
    (143, 'UA', 560, 'IAD', 'SFO', '2025-01-01 19:41:00', '2025-01-01 22:29:00', 'D9', 'D33'),
    (144, 'WN', 2791, 'LAX', 'SFO', '2025-01-01 19:45:00', '2025-01-01 20:57:00', 'C49', 'C22'),
    (145, 'WN', 2079, 'SAN', 'SFO', '2025-01-01 21:03:00', '2025-01-01 22:33:00', 'A25', 'E24'),
    (146, 'OO', 6356, 'SFO', 'SLC', '2025-01-01 19:57:00', '2025-01-01 22:49:00', 'A28', 'C45'),
    (147, 'OO', 6290, 'PSP', 'SFO', '2025-01-01 19:48:00', '2025-01-01 21:12:00', 'A33', 'E27'),
    (148, 'OO', 5363, 'ONT', 'SFO', '2025-01-01 20:27:00', '2025-01-01 21:54:00', 'A14', 'E17'),
    (149, 'OO', 5206, 'SFO', 'PHX', '2025-01-01 20:12:00', '2025-01-01 23:05:00', 'D22', 'E31'),
    (150, 'B6', 1136, 'LGB', 'SFO', '2025-01-01 20:41:00', '2025-01-01 21:56:00', 'B7', 'E45'),
    (151, 'US', 736, 'PHX', 'SFO', '2025-01-01 21:12:00', '2025-01-01 22:11:00', 'B22', 'E33'),
    (152, 'AA', 1611, 'DFW', 'SFO', '2025-01-01 21:42:00', '2025-01-01 23:37:00', 'A13', 'B44'),
    (153, 'UA', 1241, 'SFO', 'EWR', '2025-01-01 21:44:00', '2025-01-02 05:55:00', 'D3', 'A17'),
    (154, 'UA', 1726, 'OGG', 'SFO', '2025-01-01 22:19:00', '2025-01-02 04:56:00', 'C35', 'B23'),
    (155, 'US', 898, 'SFO', 'CLT', '2025-01-01 22:29:00', '2025-01-02 06:18:00', 'D7', 'D15'),
    (156, 'UA', 1669, 'SFO', 'PHL', '2025-01-01 22:26:00', '2025-01-02 06:30:00', 'B13', 'A12'),
    (157, 'UA', 362, 'SFO', 'BWI', '2025-01-01 22:31:00', '2025-01-02 06:36:00', 'E32', 'A36'),
    (158, 'OO', 6220, 'SFO', 'RNO', '2025-01-01 22:25:00', '2025-01-01 23:23:00', 'D18', 'A31'),
    (159, 'OO', 6376, 'SFO', 'MRY', '2025-01-01 22:35:00', '2025-01-01 23:19:00', 'C24', 'E48'),
    (160, 'UA', 522, 'SFO', 'PDX', '2025-01-01 22:36:00', '2025-01-02 00:31:00', 'B17', 'A40'),
    (161, 'UA', 1728, 'SFO', 'BOS', '2025-01-01 22:46:00', '2025-01-02 07:20:00', 'C40', 'E43'),
    (162, 'UA', 724, 'SFO', 'LAX', '2025-01-01 22:45:00', '2025-01-02 00:29:00', 'E43', 'B23'),
    (163, 'UA', 1681, 'SFO', 'CLE', '2025-01-01 23:03:00', '2025-01-02 06:45:00', 'D26', 'E19'),
    (164, 'UA', 1580, 'OGG', 'SFO', '2025-01-01 23:24:00', '2025-01-02 06:07:00', 'E38', 'A35'),
    (165, 'DL', 806, 'SFO', 'MSP', '2025-01-02 00:22:00', '2025-01-02 05:57:00', 'B27', 'A3'),
    (166, 'OO', 5467, 'ONT', 'SFO', '2025-01-02 05:02:00', '2025-01-02 06:18:00', 'E8', 'C32'),
    (167, 'WN', 1989, 'SFO', 'LAS', '2025-01-02 05:35:00', '2025-01-02 06:46:00', 'A21', 'D15'),
    (168, 'UA', 1619, 'BOS', 'SFO', '2025-01-02 05:56:00', '2025-01-02 09:56:00', 'C46', 'E11'),
    (169, 'UA', 1076, 'SFO', 'ORD', '2025-01-02 05:58:00', '2025-01-02 12:00:00', 'E27', 'A31'),
    (170, 'F9', 1056, 'SFO', 'IAH', '2025-01-02 05:59:00', '2025-01-02 12:09:00', 'A29', 'B13'),
    (171, 'WN', 2242, 'LAX', 'SFO', '2025-01-02 06:22:00', '2025-01-02 07:36:00', 'C12', 'D43'),
    (172, 'B6', 735, 'SFO', 'LGB', '2025-01-02 06:01:00', '2025-01-02 07:20:00', 'C2', 'C37'),
    (173, 'WN', 3859, 'SFO', 'MDW', '2025-01-02 06:27:00', '2025-01-02 12:26:00', 'A28', 'E38'),
    (174, 'UA', 1285, 'SFO', 'ORD', '2025-01-02 06:43:00', '2025-01-02 12:47:00', 'B43', 'A35'),
    (175, 'WN', 1726, 'SFO', 'PHX', '2025-01-02 06:32:00', '2025-01-02 09:20:00', 'C16', 'B2'),
    (176, 'OO', 5186, 'SFO', 'SAN', '2025-01-02 06:25:00', '2025-01-02 07:59:00', 'C47', 'C1'),
    (177, 'UA', 566, 'PIT', 'SFO', '2025-01-02 07:15:00', '2025-01-02 09:46:00', 'C39', 'E39'),
    (178, 'UA', 1195, 'MCO', 'SFO', '2025-01-02 07:18:00', '2025-01-02 10:05:00', 'A47', 'C43'),
    (179, 'CY', 925, 'LAX', 'SFO', '2025-01-02 06:57:00', '2025-01-02 08:07:00', 'E29', 'C19'),
    (180, 'UA', 761, 'ORD', 'SFO', '2025-01-02 07:07:00', '2025-01-02 09:34:00', 'B22', 'A3'),
    (181, 'UA', 758, 'SFO', 'JFK', '2025-01-02 07:07:00', '2025-01-02 15:53:00', 'B15', 'B39'),
    (182, 'CY', 67, 'IAD', 'SFO', '2025-01-02 07:07:00', '2025-01-02 10:00:00', 'D24', 'E33'),
    (183, 'UA', 706, 'SEA', 'SFO', '2025-01-02 07:05:00', '2025-01-02 08:59:00', 'D9', 'C28'),
    (184, 'OO', 5441, 'SFO', 'SBP', '2025-01-02 07:06:00', '2025-01-02 08:11:00', 'B3', 'B29'),
    (185, 'CY', 202, 'SFO', 'ORD', '2025-01-02 07:18:00', '2025-01-02 13:23:00', 'C16', 'D10'),
    (186, 'CY', 710, 'SFO', 'DAL', '2025-01-02 07:16:00', '2025-01-02 12:43:00', 'D1', 'E13'),
    (187, 'AA', 24, 'SFO', 'JFK', '2025-01-02 07:06:00', '2025-01-02 15:29:00', 'C38', 'D2'),
    (188, 'UA', 72, 'HNL', 'SFO', '2025-01-02 07:28:00', '2025-01-02 14:20:00', 'C8', 'E18'),
    (189, 'UA', 1278, 'ATL', 'SFO', '2025-01-02 07:30:00', '2025-01-02 09:41:00', 'C16', 'B21'),
    (190, 'UA', 498, 'SFO', 'SAN', '2025-01-02 07:20:00', '2025-01-02 08:38:00', 'B42', 'E44'),
    (191, 'CY', 183, 'EWR', 'SFO', '2025-01-02 07:26:00', '2025-01-02 10:51:00', 'B33', 'A12'),
    (192, 'AA', 76, 'SFO', 'LAX', '2025-01-02 07:22:00', '2025-01-02 08:44:00', 'E28', 'B18'),
    (193, 'WN', 2325, 'DEN', 'SFO', '2025-01-02 07:40:00', '2025-01-02 09:03:00', 'A5', 'A17'),
    (194, 'WN', 1545, 'LAX', 'SFO', '2025-01-02 07:48:00', '2025-01-02 09:02:00', 'D42', 'A25'),
    (195, 'OO', 5196, 'SFO', 'PDX', '2025-01-02 07:46:00', '2025-01-02 09:32:00', 'B12', 'B11'),
    (196, 'UA', 1053, 'DEN', 'SFO', '2025-01-02 08:06:00', '2025-01-02 09:41:00', 'C37', 'E7'),
    (197, 'AA', 23, 'DFW', 'SFO', '2025-01-02 07:53:00', '2025-01-02 09:26:00', 'D24', 'C34'),
    (198, 'UA', 322, 'SNA', 'SFO', '2025-01-02 07:53:00', '2025-01-02 09:17:00', 'E22', 'D32'),
    (199, 'UA', 500, 'IND', 'SFO', '2025-01-02 08:02:00', '2025-01-02 09:46:00', 'D47', 'B25'),
    (200, 'UA', 662, 'SAN', 'SFO', '2025-01-02 08:51:00', '2025-01-02 10:38:00', 'C36', 'B37'),
    (201, 'UA', 834, 'LAS', 'SFO', '2025-01-02 07:51:00', '2025-01-02 09:17:00', 'B24', 'E14'),
    (202, 'CY', 1930, 'SFO', 'LAX', '2025-01-02 07:58:00', '2025-01-02 09:29:00', 'D1', 'E49'),
    (203, 'UA', 1454, 'ORD', 'SFO', '2025-01-02 08:09:00', '2025-01-02 10:54:00', 'C45', 'D18'),
    (204, 'AA', 1368, 'SFO', 'DFW', '2025-01-02 08:11:00', '2025-01-02 13:35:00', 'A24', 'D28'),
    (205, 'UA', 460, 'LAX', 'SFO', '2025-01-02 08:22:00', '2025-01-02 09:53:00', 'E4', 'D14'),
    (206, 'UA', 1547, 'SFO', 'BOS', '2025-01-02 08:30:00', '2025-01-02 16:52:00', 'B24', 'D37'),
    (207, 'AA', 242, 'SFO', 'ORD', '2025-01-02 08:28:00', '2025-01-02 14:41:00', 'D10', 'B31'),
    (208, 'OO', 5538, 'MFR', 'SFO', '2025-01-02 08:15:00', '2025-01-02 09:44:00', 'D3', 'B43'),
    (209, 'UA', 1739, 'SFO', 'EWR', '2025-01-02 08:56:00', '2025-01-02 17:03:00', 'B18', 'A15'),
    (210, 'DL', 745, 'DTW', 'SFO', '2025-01-02 08:36:00', '2025-01-02 10:43:00', 'B2', 'D29'),
    (211, 'OO', 5445, 'SFO', 'SBP', '2025-01-02 10:35:00', '2025-01-02 11:50:00', 'D39', 'A1'),
    (212, 'OO', 5223, 'SLC', 'SFO', '2025-01-02 08:52:00', '2025-01-02 09:50:00', 'C17', 'C19'),
    (213, 'UA', 1615, 'SFO', 'IAD', '2025-01-02 08:44:00', '2025-01-02 16:41:00', 'A19', 'C46'),
    (214, 'AA', 1435, 'ORD', 'SFO', '2025-01-02 08:43:00', '2025-01-02 11:08:00', 'B49', 'B11'),
    (215, 'HA', 11, 'SFO', 'HNL', '2025-01-02 09:28:00', '2025-01-02 13:38:00', 'E28', 'B21'),
    (216, 'OO', 6373, 'SFO', 'DFW', '2025-01-02 08:51:00', '2025-01-02 14:40:00', 'B42', 'D13'),
    (217, 'UA', 479, 'SFO', 'LAX', '2025-01-02 11:15:00', '2025-01-02 12:47:00', 'E32', 'B7'),
    (218, 'UA', 361, 'SFO', 'LIH', '2025-01-02 09:33:00', '2025-01-02 13:10:00', 'A45', 'C26'),
    (219, 'WN', 2074, 'SFO', 'SNA', '2025-01-02 09:08:00', '2025-01-02 10:18:00', 'A6', 'A49'),
    (220, 'CY', 84, 'SFO', 'IAD', '2025-01-02 09:20:00', '2025-01-02 17:25:00', 'A37', 'B4'),
    (221, 'WN', 2969, 'SAN', 'SFO', '2025-01-02 09:35:00', '2025-01-02 10:59:00', 'B50', 'A43'),
    (222, 'UA', 1249, 'SFO', 'MCO', '2025-01-02 10:10:00', '2025-01-02 18:15:00', 'D13', 'E20'),
    (223, 'WN', 1820, 'SFO', 'LAS', '2025-01-02 09:52:00', '2025-01-02 11:19:00', 'B19', 'D16'),
    (224, 'WN', 2414, 'SFO', 'PHX', '2025-01-02 09:57:00', '2025-01-02 12:52:00', 'E1', 'C5'),
    (225, 'UA', 698, 'SEA', 'SFO', '2025-01-02 09:54:00', '2025-01-02 11:59:00', 'E37', 'D34'),
    (226, 'AS', 303, 'SFO', 'SEA', '2025-01-02 10:06:00', '2025-01-02 12:11:00', 'D36', 'C9'),
    (227, 'WN', 3323, 'SFO', 'DEN', '2025-01-02 10:40:00', '2025-01-02 13:59:00', 'A4', 'E35'),
    (228, 'OO', 6286, 'SBA', 'SFO', '2025-01-02 10:23:00', '2025-01-02 11:47:00', 'B46', 'A8'),
    (229, 'OO', 5603, 'PHX', 'SFO', '2025-01-02 10:29:00', '2025-01-02 11:41:00', 'A2', 'C22'),
    (230, 'CY', 218, 'SFO', 'AUS', '2025-01-02 10:28:00', '2025-01-02 15:56:00', 'E49', 'C21'),
    (231, 'OO', 5335, 'CEC', 'SFO', '2025-01-02 10:32:00', '2025-01-02 12:09:00', 'D29', 'D17'),
    (232, 'OO', 5646, 'SFO', 'SBA', '2025-01-02 10:33:00', '2025-01-02 11:54:00', 'A50', 'E40'),
    (233, 'OO', 5635, 'SFO', 'SNA', '2025-01-02 11:22:00', '2025-01-02 12:55:00', 'C17', 'D47'),
    (234, 'OO', 6380, 'SFO', 'BOI', '2025-01-02 10:41:00', '2025-01-02 13:18:00', 'B18', 'C15'),
    (235, 'UA', 662, 'SFO', 'MSP', '2025-01-02 10:52:00', '2025-01-02 16:39:00', 'B6', 'E11'),
    (236, 'OO', 6270, 'SFO', 'RNO', '2025-01-02 10:43:00', '2025-01-02 11:52:00', 'C15', 'E10'),
    (237, 'OO', 4975, 'SFO', 'TUS', '2025-01-02 12:23:00', '2025-01-02 15:18:00', 'D5', 'C12'),
    (238, 'UA', 322, 'SFO', 'SEA', '2025-01-02 10:49:00', '2025-01-02 12:54:00', 'E29', 'D3'),
    (239, 'WN', 1009, 'LAX', 'SFO', '2025-01-02 11:33:00', '2025-01-02 12:45:00', 'E12', 'C30'),
    (240, 'OO', 4966, 'SFO', 'SUN', '2025-01-02 11:11:00', '2025-01-02 14:15:00', 'D39', 'D21'),
    (241, 'UA', 753, 'SFO', 'AUS', '2025-01-02 11:40:00', '2025-01-02 17:33:00', 'B30', 'D47'),
    (242, 'UA', 242, 'SFO', 'SAN', '2025-01-02 12:00:00', '2025-01-02 13:24:00', 'B27', 'E27'),
    (243, 'UA', 390, 'SEA', 'SFO', '2025-01-02 11:10:00', '2025-01-02 13:08:00', 'A17', 'A23'),
    (244, 'OO', 6336, 'RNO', 'SFO', '2025-01-02 11:17:00', '2025-01-02 12:17:00', 'A45', 'E39'),
    (245, 'OO', 5250, 'MRY', 'SFO', '2025-01-02 11:12:00', '2025-01-02 12:28:00', 'B33', 'E11'),
    (246, 'UA', 317, 'SAN', 'SFO', '2025-01-02 12:10:00', '2025-01-02 13:40:00', 'A14', 'A42'),
    (247, 'OO', 5652, 'SFO', 'PSP', '2025-01-02 12:14:00', '2025-01-02 13:50:00', 'E17', 'B17'),
    (248, 'UA', 1195, 'SFO', 'ORD', '2025-01-02 11:56:00', '2025-01-02 18:12:00', 'B39', 'B19'),
    (249, 'CY', 714, 'SFO', 'DAL', '2025-01-02 11:40:00', '2025-01-02 17:22:00', 'C50', 'A19'),
    (250, 'WN', 2779, 'LAX', 'SFO', '2025-01-02 11:58:00', '2025-01-02 13:09:00', 'B19', 'C27'),
    (251, 'UA', 1722, 'SFO', 'IAH', '2025-01-02 11:58:00', '2025-01-02 18:29:00', 'E12', 'D6'),
    (252, 'UA', 766, 'JFK', 'SFO', '2025-01-02 11:55:00', '2025-01-02 15:21:00', 'A16', 'C18'),
    (253, 'UA', 1437, 'ORD', 'SFO', '2025-01-02 12:36:00', '2025-01-02 15:09:00', 'A33', 'E19'),
    (254, 'UA', 1294, 'LAS', 'SFO', '2025-01-02 12:20:00', '2025-01-02 13:45:00', 'E7', 'A10'),
    (255, 'UA', 1266, 'SEA', 'SFO', '2025-01-02 12:13:00', '2025-01-02 14:11:00', 'B32', 'C18'),
    (256, 'UA', 1668, 'SFO', 'EWR', '2025-01-02 12:24:00', '2025-01-02 20:34:00', 'D37', 'A30'),
    (257, 'OO', 6304, 'BUR', 'SFO', '2025-01-02 12:31:00', '2025-01-02 14:01:00', 'C7', 'C4'),
    (258, 'WN', 4391, 'SFO', 'STL', '2025-01-02 12:55:00', '2025-01-02 18:36:00', 'A12', 'D39'),
    (259, 'UA', 236, 'HNL', 'SFO', '2025-01-02 13:19:00', '2025-01-02 20:12:00', 'B25', 'D26'),
    (260, 'UA', 1705, 'SFO', 'IAH', '2025-01-02 12:51:00', '2025-01-02 18:50:00', 'E11', 'E32'),
    (261, 'OO', 6490, 'EUG', 'SFO', '2025-01-02 13:00:00', '2025-01-02 14:21:00', 'D34', 'D29'),
    (262, 'UA', 820, 'SFO', 'ORD', '2025-01-02 13:02:00', '2025-01-02 19:11:00', 'B45', 'E24'),
    (263, 'AS', 381, 'SFO', 'PDX', '2025-01-02 12:52:00', '2025-01-02 14:34:00', 'E11', 'B3'),
    (264, 'AS', 305, 'SFO', 'SEA', '2025-01-02 12:59:00', '2025-01-02 15:10:00', 'C34', 'C41'),
    (265, 'OO', 6273, 'SFO', 'SLC', '2025-01-02 12:57:00', '2025-01-02 15:50:00', 'C3', 'D36'),
    (266, 'DL', 2254, 'SFO', 'DTW', '2025-01-02 13:05:00', '2025-01-02 20:29:00', 'E10', 'D30'),
    (267, 'B6', 1435, 'SFO', 'LGB', '2025-01-02 13:34:00', '2025-01-02 15:02:00', 'B31', 'D48'),
    (268, 'F9', 661, 'DEN', 'SFO', '2025-01-02 13:11:00', '2025-01-02 14:35:00', 'C3', 'E38'),
    (269, 'UA', 1092, 'EWR', 'SFO', '2025-01-02 13:44:00', '2025-01-02 17:05:00', 'D13', 'A41'),
    (270, 'US', 1812, 'SFO', 'CLT', '2025-01-02 13:32:00', '2025-01-02 21:36:00', 'A50', 'A5'),
    (271, 'WN', 2598, 'LAX', 'SFO', '2025-01-02 13:48:00', '2025-01-02 14:54:00', 'E23', 'C38'),
    (272, 'UA', 216, 'SFO', 'SNA', '2025-01-02 15:23:00', '2025-01-02 16:41:00', 'C23', 'E6'),
    (273, 'UA', 1060, 'SFO', 'DCA', '2025-01-02 15:19:00', '2025-01-02 23:18:00', 'D14', 'B18'),
    (274, 'OO', 6326, 'BOI', 'SFO', '2025-01-02 13:51:00', '2025-01-02 14:24:00', 'D49', 'A43'),
    (275, 'UA', 724, 'HNL', 'SFO', '2025-01-02 14:43:00', '2025-01-02 21:33:00', 'B26', 'A50'),
    (276, 'OO', 5463, 'SFO', 'SMF', '2025-01-02 13:50:00', '2025-01-02 14:32:00', 'A13', 'D23'),
    (277, 'UA', 698, 'SFO', 'MCO', '2025-01-02 14:24:00', '2025-01-02 22:33:00', 'D49', 'B2'),
    (278, 'CY', 1178, 'SFO', 'EWR', '2025-01-02 13:58:00', '2025-01-02 22:06:00', 'C27', 'D9'),
    (279, 'OO', 6425, 'TUS', 'SFO', '2025-01-02 14:50:00', '2025-01-02 15:54:00', 'E29', 'D17'),
    (280, 'OO', 5615, 'MFR', 'SFO', '2025-01-02 13:55:00', '2025-01-02 15:15:00', 'B44', 'E27'),
    (281, 'WN', 3450, 'SFO', 'MDW', '2025-01-02 14:50:00', '2025-01-02 20:46:00', 'D39', 'B44'),
    (282, 'UA', 1733, 'SFO', 'DEN', '2025-01-02 15:11:00', '2025-01-02 18:47:00', 'D8', 'D2'),
    (283, 'UA', 1263, 'KOA', 'SFO', '2025-01-02 14:59:00', '2025-01-02 21:49:00', 'C39', 'E3'),
    (284, 'UA', 1568, 'SFO', 'RDU', '2025-01-02 14:55:00', '2025-01-02 23:03:00', 'A30', 'A26'),
    (285, 'UA', 1159, 'SFO', 'BOS', '2025-01-02 14:57:00', '2025-01-02 23:23:00', 'C19', 'A30'),
    (286, 'OO', 5652, 'PSP', 'SFO', '2025-01-02 14:52:00', '2025-01-02 16:46:00', 'D37', 'A5'),
    (287, 'US', 749, 'CLT', 'SFO', '2025-01-02 15:28:00', '2025-01-02 17:47:00', 'D17', 'D38'),
    (288, 'UA', 1491, 'SFO', 'DEN', '2025-01-02 15:09:00', '2025-01-02 18:45:00', 'E31', 'E36'),
    (289, 'WN', 2687, 'SFO', 'LAX', '2025-01-02 15:27:00', '2025-01-02 16:31:00', 'E42', 'B13'),
    (290, 'DL', 2040, 'SFO', 'JFK', '2025-01-02 15:09:00', '2025-01-02 23:20:00', 'E2', 'D33'),
    (291, 'OO', 5549, 'SFO', 'AUS', '2025-01-02 15:24:00', '2025-01-02 21:11:00', 'B42', 'C1'),
    (292, 'AS', 308, 'SEA', 'SFO', '2025-01-02 15:22:00', '2025-01-02 17:18:00', 'C39', 'A46'),
    (293, 'F9', 662, 'SFO', 'DEN', '2025-01-02 15:16:00', '2025-01-02 18:50:00', 'C5', 'A8'),
    (294, 'UA', 735, 'DEN', 'SFO', '2025-01-02 16:00:00', '2025-01-02 17:19:00', 'A32', 'C24'),
    (295, 'UA', 404, 'IAH', 'SFO', '2025-01-02 17:26:00', '2025-01-02 19:32:00', 'C14', 'A7'),
    (296, 'CY', 210, 'SFO', 'ORD', '2025-01-02 15:42:00', '2025-01-02 21:44:00', 'A31', 'B46'),
    (297, 'AS', 315, 'SFO', 'SEA', '2025-01-02 19:47:00', '2025-01-02 22:16:00', 'A8', 'B49'),
    (298, 'WN', 1901, 'SAN', 'SFO', '2025-01-02 15:48:00', '2025-01-02 17:09:00', 'A23', 'B38'),
    (299, 'OO', 5580, 'SFO', 'SNA', '2025-01-02 16:35:00', '2025-01-02 18:03:00', 'D40', 'E43'),
    (300, 'AA', 2265, 'DFW', 'SFO', '2025-01-02 16:07:00', '2025-01-02 17:41:00', 'A4', 'A23'),
    (301, 'OO', 5363, 'SFO', 'ONT', '2025-01-02 16:15:00', '2025-01-02 17:32:00', 'A10', 'A26'),
    (302, 'OO', 5609, 'SFO', 'PDX', '2025-01-02 17:01:00', '2025-01-02 18:42:00', 'E49', 'D4'),
    (303, 'OO', 5645, 'MFR', 'SFO', '2025-01-02 16:15:00', '2025-01-02 17:35:00', 'A25', 'E49'),
    (304, 'OO', 4976, 'SFO', 'STL', '2025-01-02 16:37:00', '2025-01-02 22:27:00', 'C35', 'A2'),
    (305, 'OO', 6217, 'BUR', 'SFO', '2025-01-02 16:44:00', '2025-01-02 17:55:00', 'D50', 'A14'),
    (306, 'WN', 1276, 'SFO', 'LAX', '2025-01-02 16:25:00', '2025-01-02 17:32:00', 'B8', 'D22'),
    (307, 'OO', 6238, 'ACV', 'SFO', '2025-01-02 16:25:00', '2025-01-02 17:36:00', 'E24', 'A39'),
    (308, 'OO', 5190, 'SFO', 'ASE', '2025-01-02 17:08:00', '2025-01-02 20:17:00', 'C9', 'E6'),
    (309, 'OO', 4627, 'SFO', 'SLC', '2025-01-02 16:43:00', '2025-01-02 19:41:00', 'C32', 'C4'),
    (310, 'OO', 6476, 'SFO', 'MFR', '2025-01-02 16:38:00', '2025-01-02 18:16:00', 'B26', 'D33'),
    (311, 'UA', 486, 'SFO', 'LAS', '2025-01-02 16:52:00', '2025-01-02 18:24:00', 'D25', 'E8'),
    (312, 'DL', 214, 'SFO', 'DTW', '2025-01-02 19:09:00', '2025-01-03 02:20:00', 'C6', 'E35'),
    (313, 'CY', 592, 'SFO', 'PSP', '2025-01-02 16:54:00', '2025-01-02 18:14:00', 'B40', 'B9'),
    (314, 'UA', 529, 'SNA', 'SFO', '2025-01-02 17:01:00', '2025-01-02 18:15:00', 'D28', 'B30'),
    (315, 'UA', 372, 'MSP', 'SFO', '2025-01-02 17:39:00', '2025-01-02 19:19:00', 'E28', 'C18'),
    (316, 'AA', 1213, 'SFO', 'DFW', '2025-01-02 18:20:00', '2025-01-03 00:19:00', 'E39', 'A42'),
    (317, 'AS', 341, 'PSP', 'SFO', '2025-01-02 17:15:00', '2025-01-02 18:37:00', 'D40', 'D15'),
    (318, 'OO', 5418, 'SBP', 'SFO', '2025-01-02 18:11:00', '2025-01-02 19:09:00', 'C31', 'D39'),
    (319, 'CY', 193, 'EWR', 'SFO', '2025-01-02 17:29:00', '2025-01-02 20:59:00', 'E20', 'E40'),
    (320, 'UA', 1723, 'SFO', 'KOA', '2025-01-02 17:26:00', '2025-01-02 21:02:00', 'E18', 'D41'),
    (321, 'UA', 1762, 'SFO', 'IAH', '2025-01-02 17:37:00', '2025-01-02 23:11:00', 'C40', 'C37'),
    (322, 'CY', 720, 'SFO', 'DAL', '2025-01-02 17:23:00', '2025-01-02 22:47:00', 'D13', 'D23'),
    (323, 'CY', 963, 'SAN', 'SFO', '2025-01-02 17:36:00', '2025-01-02 19:05:00', 'B1', 'E32'),
    (324, 'OO', 5255, 'SFO', 'EUG', '2025-01-02 17:27:00', '2025-01-02 19:02:00', 'C7', 'B2'),
    (325, 'CY', 960, 'SFO', 'SAN', '2025-01-02 17:32:00', '2025-01-02 18:50:00', 'C3', 'C4'),
    (326, 'UA', 1419, 'MSY', 'SFO', '2025-01-02 17:46:00', '2025-01-02 20:07:00', 'C26', 'E49'),
    (327, 'WN', 1585, 'SFO', 'LAX', '2025-01-02 17:46:00', '2025-01-02 19:31:00', 'A12', 'A12'),
    (328, 'DL', 1505, 'MSP', 'SFO', '2025-01-02 17:47:00', '2025-01-02 19:48:00', 'A33', 'B37'),
    (329, 'US', 686, 'PHX', 'SFO', '2025-01-02 17:59:00', '2025-01-02 19:05:00', 'C28', 'C11'),
    (330, 'AS', 302, 'SEA', 'SFO', '2025-01-02 18:00:00', '2025-01-02 20:04:00', 'B28', 'E17'),
    (331, 'UA', 1552, 'RDU', 'SFO', '2025-01-02 18:08:00', '2025-01-02 20:51:00', 'B15', 'D48'),
    (332, 'UA', 697, 'LAS', 'SFO', '2025-01-02 17:55:00', '2025-01-02 19:14:00', 'E37', 'B31'),
    (333, 'AA', 219, 'LAX', 'SFO', '2025-01-02 17:54:00', '2025-01-02 19:05:00', 'C27', 'E34'),
    (334, 'OO', 5425, 'ONT', 'SFO', '2025-01-02 18:04:00', '2025-01-02 19:23:00', 'C46', 'A14'),
    (335, 'WN', 1861, 'SFO', 'PHX', '2025-01-02 20:49:00', '2025-01-02 23:31:00', 'C31', 'B42'),
    (336, 'OO', 5631, 'SFO', 'BUR', '2025-01-02 18:10:00', '2025-01-02 19:29:00', 'D26', 'E29'),
    (337, 'UA', 720, 'SFO', 'AUS', '2025-01-02 18:13:00', '2025-01-02 23:41:00', 'E10', 'D1'),
    (338, 'UA', 1581, 'SFO', 'OGG', '2025-01-02 18:15:00', '2025-01-02 22:06:00', 'B40', 'E18'),
    (339, 'OO', 5227, 'SFO', 'SAN', '2025-01-02 18:26:00', '2025-01-02 19:59:00', 'E33', 'E50'),
    (340, 'CY', 941, 'LAX', 'SFO', '2025-01-02 18:24:00', '2025-01-02 19:39:00', 'D4', 'A22'),
    (341, 'AA', 1459, 'ORD', 'SFO', '2025-01-02 18:21:00', '2025-01-02 21:11:00', 'D42', 'B25'),
    (342, 'OO', 5442, 'SFO', 'RDD', '2025-01-02 18:19:00', '2025-01-02 19:35:00', 'E21', 'B9'),
    (343, 'AA', 69, 'MIA', 'SFO', '2025-01-02 18:31:00', '2025-01-02 21:33:00', 'B11', 'B15'),
    (344, 'OO', 6220, 'DFW', 'SFO', '2025-01-02 19:01:00', '2025-01-02 20:40:00', 'D35', 'E4'),
    (345, 'UA', 1670, 'SFO', 'HNL', '2025-01-02 18:42:00', '2025-01-02 22:52:00', 'A15', 'C18'),
    (346, 'DL', 435, 'JFK', 'SFO', '2025-01-02 18:57:00', '2025-01-02 22:25:00', 'D35', 'C33'),
    (347, 'UA', 508, 'SEA', 'SFO', '2025-01-02 19:01:00', '2025-01-02 21:05:00', 'C16', 'B8'),
    (348, 'UA', 1523, 'BOS', 'SFO', '2025-01-02 21:19:00', '2025-01-03 01:19:00', 'E7', 'B32'),
    (349, 'CY', 593, 'PSP', 'SFO', '2025-01-02 19:03:00', '2025-01-02 20:21:00', 'C13', 'A47'),
    (350, 'UA', 741, 'ORD', 'SFO', '2025-01-02 19:44:00', '2025-01-02 22:14:00', 'A16', 'C32'),
    (351, 'WN', 3414, 'SFO', 'SAN', '2025-01-02 19:43:00', '2025-01-02 21:10:00', 'E3', 'D46'),
    (352, 'UA', 404, 'SFO', 'PDX', '2025-01-02 19:42:00', '2025-01-02 21:24:00', 'B48', 'A30'),
    (353, 'UA', 637, 'SFO', 'SNA', '2025-01-02 19:32:00', '2025-01-02 20:49:00', 'D40', 'A23'),
    (354, 'OO', 5573, 'SAT', 'SFO', '2025-01-02 21:13:00', '2025-01-02 22:57:00', 'D34', 'E33'),
    (355, 'AA', 219, 'SFO', 'LAX', '2025-01-02 20:03:00', '2025-01-02 22:00:00', 'E27', 'D38'),
    (356, 'OO', 5600, 'BUR', 'SFO', '2025-01-02 19:55:00', '2025-01-02 21:21:00', 'A7', 'A27'),
    (357, 'CY', 947, 'LAX', 'SFO', '2025-01-02 20:38:00', '2025-01-02 22:06:00', 'D28', 'C15'),
    (358, 'WN', 138, 'SFO', 'LAS', '2025-01-02 20:26:00', '2025-01-02 21:49:00', 'B35', 'E31'),
    (359, 'OO', 5302, 'SFO', 'PSC', '2025-01-02 20:29:00', '2025-01-02 22:28:00', 'C9', 'A15'),
    (360, 'CY', 748, 'SFO', 'SEA', '2025-01-02 20:46:00', '2025-01-02 23:17:00', 'D7', 'E25'),
    (361, 'UA', 236, 'SFO', 'EWR', '2025-01-02 20:41:00', '2025-01-03 05:04:00', 'D12', 'B47'),
    (362, 'OO', 6503, 'SFO', 'EUG', '2025-01-02 20:54:00', '2025-01-02 22:27:00', 'A50', 'E49'),
    (363, 'WN', 1174, 'LAS', 'SFO', '2025-01-02 20:58:00', '2025-01-02 22:19:00', 'C42', 'D33'),
    (364, 'AS', 744, 'SFO', 'SLC', '2025-01-02 20:52:00', '2025-01-02 23:33:00', 'B11', 'A39'),
    (365, 'WN', 4992, 'SFO', 'PHX', '2025-01-02 21:49:00', '2025-01-03 00:32:00', 'D19', 'E15'),
    (366, 'UA', 469, 'EWR', 'SFO', '2025-01-02 23:07:00', '2025-01-03 02:27:00', 'E3', 'D19'),
    (367, 'UA', 1163, 'SFO', 'EWR', '2025-01-02 21:44:00', '2025-01-03 05:45:00', 'D19', 'C17'),
    (368, 'DL', 2337, 'SFO', 'ATL', '2025-01-02 21:44:00', '2025-01-03 05:23:00', 'B4', 'A39'),
    (369, 'OO', 6432, 'SFO', 'EUG', '2025-01-02 22:24:00', '2025-01-02 23:57:00', 'E11', 'A4'),
    (370, 'US', 898, 'SFO', 'CLT', '2025-01-02 22:24:00', '2025-01-03 06:16:00', 'A45', 'A24'),
    (371, 'UA', 292, 'SFO', 'MCO', '2025-01-02 22:30:00', '2025-01-03 06:37:00', 'C7', 'C5'),
    (372, 'OO', 5600, 'SFO', 'MFR', '2025-01-02 22:36:00', '2025-01-03 00:01:00', 'E42', 'E38'),
    (373, 'UA', 263, 'LAX', 'SFO', '2025-01-02 22:48:00', '2025-01-03 00:04:00', 'B22', 'E27'),
    (374, 'OO', 5414, 'SFO', 'RDM', '2025-01-02 22:32:00', '2025-01-03 00:12:00', 'A46', 'C18'),
    (375, 'UA', 1241, 'SFO', 'EWR', '2025-01-02 23:22:00', '2025-01-03 07:44:00', 'A28', 'E27'),
    (376, 'UA', 522, 'SFO', 'PDX', '2025-01-02 22:55:00', '2025-01-03 00:42:00', 'A10', 'E39'),
    (377, 'UA', 724, 'SFO', 'ORD', '2025-01-02 23:05:00', '2025-01-03 05:20:00', 'B40', 'D23'),
    (378, 'UA', 156, 'SFO', 'MSY', '2025-01-02 23:22:00', '2025-01-03 05:24:00', 'C40', 'A9'),
    (379, 'CY', 358, 'SFO', 'BOS', '2025-01-02 22:52:00', '2025-01-03 07:31:00', 'B23', 'A44'),
    (380, 'DL', 1370, 'SFO', 'JFK', '2025-01-02 23:04:00', '2025-01-03 07:36:00', 'E10', 'A49'),
    (381, 'UA', 1729, 'SFO', 'IAD', '2025-01-02 23:10:00', '2025-01-03 07:35:00', 'D32', 'E42'),
    (382, 'CY', 34, 'SFO', 'JFK', '2025-01-02 23:15:00', '2025-01-03 07:54:00', 'D28', 'E25'),
    (383, 'DL', 1880, 'SFO', 'ATL', '2025-01-02 00:53:00', '2025-01-02 08:27:00', 'E2', 'A29'),
    (384, 'UA', 1523, 'SFO', 'EWR', '2025-01-02 23:39:00', '2025-01-03 07:43:00', 'B9', 'D15'),
    (385, 'US', 426, 'PHX', 'SFO', '2025-01-02 00:18:00', '2025-01-02 01:16:00', 'D47', 'C42');
    ```

3.  Run the following query to verify that the tables are populated:
    
    ``` text
    SELECT * FROM flights LIMIT 10;
    SELECT * FROM airports LIMIT 10;
    ```

## Create a data agent

In this section, you create a data agent named `  flights-assistant  ` . This agent doesn't include any agent context uploaded to it.

1.  In the **Explorer pane** , next to **Data Agents** , click **View actions** .
2.  Click **Create agent** .
3.  In **Name your agent** , enter `  flights-assistant  ` .
4.  Click **Create** .

## Inspect the agent in Studio

In this section, you ask the `  flights-assistant  ` agent a natural language question and it generates a SQL query. Since the agent doesn't have any context, even after asking a question with context such as `  nighttime traffic  ` , the agent generates a sub-optimal query.

1.  In the **Explorer pane** , next to your data agent, click **View actions** .

2.  Click **Inspect agent** .

3.  In the query editor, click **Generate SQL using agent: flights-assistant** .

4.  Enter the following natural language question to generate a SQL query, and click **Generate** .
    
    ``` text
    Find flights from SFO to JFK.
    ```
    
    Review the SQL query. Notice that the agent generates the correct SQL for this unambiguous question.
    
    ``` text
      SELECT
        *
      FROM
        flights
      WHERE
        departure_airport = 'SFO'
        AND arrival_airport = 'JFK';
    ```

5.  In the **Generate SQL using agent: flights-assistant** window, click **Edit** .

6.  Enter the following natural language question to generate a SQL query, and click **Update** .
    
    ``` text
    Tell me flights that can help me beat nighttime traffic if traveling from New York
    ```
    
    The database fails to understand the term `  nighttime  ` traffic. This might prevent it from generating a SQL query or cause it to generate a query that ignores the term, as the following query shows.
    
    **Note:** GenAI models are nondeterministic, meaning the same prompt may yield different responses across separate calls due to the probabilistic nature of the output generation.
    
    ``` text
    -- The database schema does not contain information about traffic.
    -- Returning all flights departing from New York airports.
    SELECT
      f.airline,
      f.flight_number,
      a.name AS departure_airport_name,
      f.departure_time,
      b.name AS arrival_airport_name,
      f.arrival_time
    FROM
      flights AS f
    JOIN
      airports AS a
      ON f.departure_airport = a.iata
    JOIN
      airports AS b
      ON f.arrival_airport = b.iata
    WHERE
      a.city = 'New York'
    ORDER BY
      f.departure_time;
    ```

## Generate context for the agent

In this section, you create a context file that helps improve the agent's querying capabilities. To address the issue from the previous section where the agent didn't recognize the term `  nighttime traffic  ` , define the term in the agent context as traffic occurring between `  5:00 PM  ` and `  7:00 PM  ` .

To generate agent context, perform the following steps:

1.  In your local directory, install Gemini CLI. For more information, see [Gemini CLI quickstart](https://geminicli.com/docs/get-started/) .

2.  [Install the Google Cloud CLI](https://cloud.google.com/sdk/docs/install) and [set up Application Default Credentials (ADC)](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

3.  Install the [MCP Toolbox Gemini CLI](https://github.com/gemini-cli-extensions/mcp-toolbox) extension, which connects to the database.
    
    ``` text
    gemini extensions install https://github.com/gemini-cli-extensions/mcp-toolbox
    ```

4.  Create a `  tools.yaml  ` configuration file in the same directory where you install the MCP toolbox to configure the database connection:
    
    ``` text
    sources:
      flight-sql-source:
        kind: spanner
        project: PROJECT_ID
        instance: INSTANCE_ID
        database: DATABASE_ID
    
    tools:
      # (Optional) Fetches database schemas for context generation in the bulk generation (/generate_bulk_templates) phase.
      list_flight_schemas_tool:
        kind: spanner-list-tables
        source: flight-spanner-source
        description: Use this tool to list all tables and their schemas in the flight database.
      # (Optional) Executes generated SQL for validation in the bulk generation (/generate_bulk_templates) phase.
      execute_sql_tool:
        kind: spanner-execute-sql
        source: flight-spanner-source
        description: Use this tool to execute SQL against the flight database.
    ```
    
    Replace the following:
    
      - `  PROJECT_ID  ` : Your Google Cloud project ID.
      - `  INSTANCE_ID  ` : The ID of your Spanner instance.
      - `  DATABASE_ID  ` : The name of the database to connect to.

5.  Install the `  uv  ` Python package installer by following [official installation guide](https://docs.astral.sh/uv/getting-started/installation/) and verify the installation succeeds by running:
    
    ``` text
    uv --version
    ```

6.  Install the DB Context Enrichment MCP Server, which includes workflows for context generation.
    
    ``` text
    gemini extensions install https://github.com/GoogleCloudPlatform/db-context-enrichment
    ```

7.  Export the Gemini API key as an environment variable. For more information about how to find the API key, see [Using Gemini API keys](https://ai.google.dev/gemini-api/docs/api-key) .
    
    ``` text
    export GEMINI_API_KEY="YOUR_API_KEY"
    ```
    
    Replace `  YOUR_API_KEY  ` with your Gemini API key.

8.  In the same directory where you created the `  tools.yaml  ` file, start Gemini:
    
    ``` text
    gemini
    ```

9.  Complete the [Gemini CLI Authentication Setup](https://geminicli.com/docs/get-started/authentication/) .

10. Verify that the MCP toolbox and the database enrichment extension are connected and ready to use.
    
    ``` text
    /mcp list
    ```

11. Run the `  /generate_targeted_templates  ` command and follow the workflow:
    
    ``` text
    /generate_targeted_templates
    ```

12. Provide the natural language query that you want to add to the query template in the terminal.
    
    ``` text
    Tell me flights that can help me beat nighttime traffic if traveling from New York
    ```

13. Provide a corresponding SQL query that you want to add to the query template. This query template defines the term `  nighttime  ` as occurring between `  5:00 PM  ` and `  7:00 PM  ` .
    
    ``` text
    SELECT
      f.airline,
      f.flight_number,
      a.name AS airport_name,
      f.departure_time
    FROM
      flights f
    JOIN
      airports a
      ON f.departure_airport = a.iata
    WHERE
      a.city = 'New York'
      AND (
        EXTRACT(HOUR FROM f.departure_time) < 17
        OR EXTRACT(HOUR FROM f.departure_time) >= 19
      )
    ORDER BY
      f.departure_time;
    ```

14. Press **Enter** . Gemini converts your input into a specific format that refines the agent's performance across a wide range of user queries. For more information, see [Agent context](/spanner/docs/data-agent-overview#agent-context) .
    
    Optionally, run the `  /generate_bulk_templates  ` workflow to let Gemini CLI generate more context by scanning your database schema and suggesting related context. Ensure you add both the `  list_flight_schemas_tool  ` and the `  execute_sql_tool  ` to the `  tools.yaml  ` configuration you created in step 4.
    
    **Note:** The `  /generate_bulk_templates  ` workflow requires the list schema tool for fetching database schemas and the execute SQL tool for SQL validation, as the previous `  tools.yaml  ` configuration shows.

15. Review the generated query template. You can either save the query template as a new agent context file or append it to an existing agent context file.

16. Select the option to create a new agent context file. Gemini creates a filename `  INSTANCE_ID _ DATABASE_ID _context_set_ TIMESTAMP .json  ` in the same directory, with the following content:
    
    ``` text
    {
      "templates": [
        {
          "nl_query": "Tell me flights that can help me beat nighttime traffic if traveling from New York",
          "sql": "SELECT f.airline, f.flight_number, a.name AS airport_name, f.departure_time FROM flights f JOIN airports a ON f.departure_airport = a.iata WHERE a.city = 'New York' AND (EXTRACT(HOUR FROM f.departure_time) < 17 OR EXTRACT(HOUR FROM f.departure_time) >= 19) ORDER BY f.departure_time;",
          "intent": "Tell me flights that can help me beat nighttime traffic if traveling from New York",
          "manifest": "Tell me flights that can help me beat nighttime traffic if traveling from a given city",
          "parameterized": {
            "parameterized_sql": "SELECT f.airline, f.flight_number, a.name AS airport_name, f.departure_time FROM flights f JOIN airports a ON f.departure_airport = a.iata WHERE a.city = ? AND (EXTRACT(HOUR FROM f.departure_time) < 17 OR EXTRACT(HOUR FROM f.departure_time) >= 19) ORDER BY f.departure_time;",
            "parameterized_intent": "Tell me flights that can help me beat nighttime traffic if traveling from ?"
          }
        }
      ]
    }
    ```

## Upload context to the agent

In this section, you upload the agent context file to the data agent, so that it improves the agent's SQL generation capabilities on your database.

To upload the context, perform the following steps:

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , next to **Data Agents** , click **View actions** .

5.  Click **Edit agent** .

6.  Optional: Edit **Agent description** .

7.  Click **Browse** in the **Upload agent context file** section, and select the agent context file generated earlier.

8.  Click **Save** .

## Generate SQL query using agent context

In this section, you use the agent context file you uploaded to ask natural language questions. This lets you verify that the agent correctly understands and applies definitions for terms like `  nighttime traffic  ` and other related phrases.

**Note:** GenAI models are nondeterministic, meaning the same prompt may yield different responses across separate calls due to the probabilistic nature of the output generation.

To generate SQL queries, perform the following steps:

1.  In the **Explorer** pane, next to your data agent, click **View actions** .

2.  Click **Inspect agent** .

3.  In the query editor, click **Generate SQL using agent: flights-assistant** .

4.  Enter the following natural language question to generate a SQL query, and click **Generate** .
    
    ``` text
    Tell me flights that can help me beat nighttime traffic if traveling from New York
    ```
    
    The generated SQL query looks similar to the following:
    
    ``` text
    SELECT
      f.airline,
      f.flight_number,
      a.name AS airport_name,
      f.departure_time
    FROM
      flights f
    JOIN
      airports a ON f.departure_airport = a.iata
    WHERE
      a.city = 'New York'
      AND (
        EXTRACT(HOUR FROM f.departure_time) < 17
        OR EXTRACT(HOUR FROM f.departure_time) >= 19
      )
    ORDER BY
      f.departure_time;
    ```
    
    This is the same question you added to the data agent's context. Observe that the agent can now accurately interpret the term `  nighttime traffic  ` .
    
    Although the context originates from one particular question, the agent uses it to enhance SQL generation for a wide range of similar questions.

5.  In the **Generate SQL using agent: flights-assistant** window, click **Edit** .

6.  Enter the following similar question to generate a SQL query, and click **Update** .
    
    ``` text
    What are the flights that can help me avoid evening traffic if departing from Boston
    ```
    
    Since the question replaces the term `  nighttime traffic  ` with a similar term, `  evening traffic  ` , the agent provides a consistent answer to this question by applying the same interpretation.
    
    The generated SQL query looks similar to the following:
    
    ``` text
    -- What are the flights that can help me avoid evening traffic if departing from Boston
    SELECT
      f.airline,
      f.flight_number,
      a.name AS airport_name,
      f.departure_time
    FROM
      flights f
    JOIN
      airports a
    ON
      f.departure_airport = a.iata
    WHERE
      a.city = 'Boston'
      AND (
        EXTRACT(HOUR FROM f.departure_time) < 17
        OR EXTRACT(HOUR FROM f.departure_time) >= 19
      )
    ORDER BY
      f.departure_time;
    ```

## Integrate the agent with your application

In this section, you create a data agent for a flight-finding application. This data agent provides a conversational interface to the `  flights  ` and `  airports  ` table you created earlier. It also explains how to create and integrate this agent into your application using [Agent Development Kit (ADK)](https://github.com/google/adk-python) , the Gemini Data Analytics QueryData MCP tool, and agent context to improve quality of the responses.

**Note:** GenAI models are nondeterministic, meaning the same prompt may yield different responses across separate calls due to the probabilistic nature of the output generation.

1.  Download [MCP Toolbox](https://googleapis.github.io/genai-toolbox/getting-started/introduction/#installing-the-server) version 0.24.0 or later. MCP toolbox exposes the data agent as a tool for applications to connect with. The MCP toolbox differs from the MCP Toolbox Gemini CLI extension you installed earlier, which generates context.

2.  In the terminal, set the project that you are using.
    
    ``` text
    gcloud config set project [PROJECT_ID]
    ```

3.  Set up [Application Default Credentials (ADC)](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .
    
    ``` text
    gcloud auth application-default login
    ```

4.  Find the agent context ID. For more information about how to find the context set ID, see [Find the agent context ID](/spanner/docs/data-agent-overview#agent-context) .

5.  Create the `  tools.yaml  ` configuration to connect to the data agent using the MCP toolbox. For more information, see [Gemini Data Analytics Source](https://googleapis.github.io/genai-toolbox/resources/sources/cloud-gda/) and Gemini Data Analytics QueryData Tool.
    
    ``` text
    sources:
      gda-api-source:
        kind: cloud-gemini-data-analytics
        projectId: "PROJECT_ID"
    
    tools:
      cloud_gda_query_tool:
        kind: cloud-gemini-data-analytics-query
        source: gda-api-source
        description: Use this tool to send natural language queries to the Gemini Data Analytics API and receive SQL, natural language answers, and explanations.
        location: "REGION_ID"
        context:
          datasourceReferences:
            spannerReference:
              databaseReference:
                engine: "GOOGLE_SQL"
                projectId: "PROJECT_ID"
                region: "REGION_ID"
                instanceId: "INSTANCE_ID"
                databaseId: "DATABASE_ID"
              agentContextReference:
                contextSetId: "DATA_AGENT_CONTEXT_SET_ID"
        generationOptions:
          generateQueryResult: true
          generateNaturalLanguageAnswer: true
          generateExplanation: true
          generateDisambiguationQuestion: true
    ```
    
    Replace the following:
    
      - `  PROJECT_ID  ` : Your Google Cloud project ID.
      - `  REGION_ID  ` : The region of your Spanner instance (e.g., us-central1).
      - `  INSTANCE_ID  ` : The ID of your Spanner instance.
      - `  DATABASE_ID  ` : The name of the database to connect to.
      - `  DATA_AGENT_CONTEXT_SET_ID  ` : The data agent context set ID.

6.  Run the MCP Toolbox server with the `  tools.yaml  ` file.
    
    ``` text
    ./toolbox --tools-file "tools.yaml"
    ```

7.  Create an ADK application that invokes [Gemini Data Analytics QueryData tool](https://googleapis.github.io/genai-toolbox/resources/tools/cloudgda/cloud-gda-query/) using the MCP Toolbox's Python SDK. For more information about how to use the MCP Toolbox's Python SDK, see the [quickstart for Toolbox](https://googleapis.github.io/genai-toolbox/getting-started/local_quickstart/) and for Python ADK, see the [quickstart for ADK](https://google.github.io/adk-docs/get-started/python/) .
    
    1.  Create a directory to store the application, for example `  flight-assistant-app  ` .
    
    2.  Change directory to the `  flight-assistant-app  ` directory.
        
        ``` text
        mkdir flight-assistant-app
        cd flight-assistant-app
        ```
    
    3.  Run the following commands under the `  flight-assistant-app  ` directory to create a virtual environment and install required components.
        
        ``` text
        python3 -m venv .venv
        source .venv/bin/activate
        pip install toolbox-core
        pip install google-genai
        pip install google-adk
        ```
    
    4.  Set up an ADK agent.
        
        1.  Create an ADK agent.
            
            ``` text
            adk create my_agent
            ```
        
        2.  Select the `  gemini-2.5-flash  ` model.
        
        3.  Select **Google AI** , and enter your Gemini API key. For more information about how to find your API key, see [Using Gemini API keys](https://ai.google.dev/gemini-api/docs/api-key) .
    
    5.  Replace the contents of the `  agent.py  ` file with the following Flight Data Assistant sample application code.
        
        ``` text
        from typing import cast
        
        from google.adk.agents.llm_agent import Agent
        from google.adk.agents.llm_agent import ToolUnion
        
        from toolbox_core import ToolboxSyncClient
        
        TOOLBOX_URL = "http://127.0.0.1:5000"
        
        INSTRUCTION = """
        # ROLE
        You are a friendly and factual flight data assistant. Your goal is to help users find the best flights for their needs by providing accurate information with a helpful, professional tone.
        - use the Query Data Tool to answer the user's question, if the tool fails to generate a valid query, ask the user to clarify their question.
        
        # OPERATIONAL CONSTRAINTS
        - TOOL LIMITATION: You only have access to the Query Data Tool. Do not claim to have capabilities beyond what this tool provides.
        - TRANSPARENCY POLICY: Maintain a seamless user experience. Never mention that you are using a tool, querying a database, or generating SQL. Frame all responses as your own direct assistance.
        - SCOPE MANAGEMENT: If a user asks for something beyond your capabilities, politely state that you cannot perform that specific task. Guide the user towards what you can help with.
        
        # COMMUNICATION STYLE
        - Be concise and scannable when listing answers.
        - Maintain a helpful, professional persona.
        
        =====
        
        # QUERY DATA TOOL
        
        Inputs:
        1. query: A natural language formulation of a database query.
        
        Outputs: (all optional)
        1. disambiguation_question: Clarification questions or comments where the tool needs the users' input.
        2. generated_query: The generated query for the user query.
        3. intent_explanation: An explanation for why the tool produced `generated_query`.
        4. query_result: The result of executing `generated_query`.
        5. natural_language_answer: The natural language answer that summarizes the `query` and `query_result`.
        
        Usage guidance:
        1. If `disambiguation_question` is produced, then solicit the needed inputs from the user and try the tool with a new `query` that has the needed clarification.
        2. If `natural_language_answer` is produced, use `intent_explanation` and `generated_query` to see if you need to clarify any assumptions for the user.
        3. If the tool output indicates failure or empty results, explain that clearly using the provided reasoning.
        """
        
        client = ToolboxSyncClient(TOOLBOX_URL)
        
        mcp_tool = client.load_tool("cloud_gda_query_tool")
        
        root_agent = Agent(
            model="gemini-2.5-flash",
            name="root_agent",
            instruction=INSTRUCTION,
            tools=cast(list[ToolUnion], [mcp_tool]),
        )
        ```
    
    **Note:** The instruction contains a system prompt that sets up a flight assistant agent. It also instructs the agent how to use the Gemini Data Analytics QueryData tool you set up in the `  tools.yaml  ` file you created earlier in step 5.

8.  Run the following commands under the `  flight-assistant-app  ` directory to start the application and access the ADK web server at `  http://127.0.0.1:8000  ` .
    
    ``` text
    adk web --port 8000
    ```

9.  Enter any text, such as `  hello  ` , to start interacting with the agent.
    
    The ADK agent answers general questions and calls the required MCP tools.

10. Enter the following flight-related question.
    
    ``` text
    How many flights depart from the west side?
    ```
    
    The MCP tool is called to answer this question. However, since the term `  the west  ` is ambiguous and doesn't specify any airports, the MCP tool returns a disambiguation question which the agent uses to construct a response.
    
    ``` text
    I cannot determine how many flights depart from the 'west side' as the database does not contain information about which airports are considered to be on the 'west side'. However, I can help you with questions like:
    
    1. How many flights depart from a specific airport?
    
    2. What are the departure airports for all flights?
    
    3. How many flights depart from each airport? Would you like to rephrase your question based on these options?
    ```

11. Enter a question similar to that in the query template generated for the agent.
    
    ``` text
    Help me find flights from San Francisco that avoid the evening rush hour.
    ```
    
    Based on the agent context added earlier, the MCP tool understands that `  evening traffic  ` occurs between 5 PM and 7 PM. The MCP tool returns the associated data for the agent to use in constructing its response.
    
    ``` text
    Here are the flights departing from San Francisco that avoid the evening rush hour (defined as 5 PM to 7 PM):
    
    * UA 1532 departing at 05:50:00
    * UA 1158 departing at 05:57:00
    * CY 922 departing at 06:38:00
    * OO 5441 departing at 07:08:00
    * UA 616 departing at 07:14:00
    * AA 24 departing at 07:14:00
    * B6 434 departing at 08:00:00
    * AA 242 departing at 08:18:00
    * UA 1739 departing at 08:22:00
    * OO 6336 departing at 08:32:00
    * US 1784 departing at 08:47:00
    * DL 1631 departing at 09:00:00
    * DL 1106 departing at 09:06:00
    * OO 5427 departing at 09:06:00
    * CY 352 departing at 09:25:00
    ```

## Iterate agent performance

The ADK web UI lets you inspect the request and response from the Gemini Data Analytics QueryData MCP tool. You can use this response to observe the tool responses such as generated SQL query, result set, intent explanation, disambiguation question, and natural language answer, to help you confirm the correctness of your agent's responses.

For example, for the input text `  How many flights depart from the west side?  ` you entered earlier, click the agent bubble. In the **Event** tab in the left navigation, expand the `  functionResponse  ` to see the following response.

``` text
"{"disambiguationQuestion": ["[NOT_ENOUGH_INFO] The database schema does not
contain information about which airports are on the 'west side'. Therefore, I
cannot determine how many flights depart from the west side.Possible alternative
questions: 1. How many flights depart from a specific airport? 2. What are the
departure airports for all flights? 3. How many flights depart from each
airport?"]}"
```

### Refine response accuracy

You can continuously refine the accuracy of responses from the Gemini Data Analytics QueryData tool by adding additional context. Use the Gemini CLI to generate context, and then upload the updated agent context to the existing `  flights-assistant  ` agent. For more information, see [Build contexts using Gemini CLI](/alloydb/docs/ai/build-context-gemini-cli) . The console immediately ingests new context after you upload it, enabling you to enhance the agent's accuracy without any application downtime.

### Multiple agents

In your development environment, you can perform A/B testing on multiple agent contexts by assigning distinct names to tools in your `  tools.yaml  ` file. For example, you can create unique `  tools.yaml  ` configurations by defining two `  cloud-gemini-data-analytics-query  ` tools with different names, such as `  cloud_gda_query_tool_v1  ` and `  cloud_gda_query_tool_v2  ` . This setup lets you implement application logic that programmatically selects the required agent context version by choosing the corresponding tool name.

The following example `  tools.yaml  ` shows how to set up multiple agents for a database source:

``` text
sources:
  gda-api-source:
    kind: cloud-gemini-data-analytics
    projectId: "<var>PROJECT_ID</var>"
tools:
  cloud_gda_query_tool_v1:
    kind: cloud-gemini-data-analytics-query
    source: gda-api-source
    context:
      datasourceReferences:
        <var>DB_SOURCE</var>:
          databaseReference: ...
          agentContextReference:
            contextSetId: "V1_YOUR_DATA_AGENT_CONTEXT_SET_ID"
    generationOptions: ...
  cloud_gda_query_tool_v2:
    kind: cloud-gemini-data-analytics-query
    source: gda-api-source
    context:
      datasourceReferences:
        <var>DB_SOURCE</var>:
          databaseReference: ...
          agentContextReference:
            contextSetId: "V2_YOUR_DATA_AGENT_CONTEXT_SET_ID"
    generationOptions: ...
```

Replace the following:

  - `  PROJECT_ID  ` : Your Google Cloud project ID.
  - `  V1_YOUR_DATA_AGENT_CONTEXT_SET_ID  ` : The data agent context set ID for version 1.
  - `  V2_YOUR_DATA_AGENT_CONTEXT_SET_ID  ` : The data agent context set ID for version 2.

## Clean up

The following sections describe how to delete these resources and objects.

### Delete the agent

Before you delete the instance, delete the agent that you created.

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , next to your data agent, click **View actions** .

5.  In the **Delete agent** window, enter `  flight-assistant  ` in the confirmation box.

6.  Click **Confirm** .

### Delete the instance

When you delete the instance that you created in the [before you begin](/alloydb/docs/ai/use-natural-language-generate-sql-queries#create-and-connect-to-database) section, you also delete all of the objects you created.

1.  In the Google Cloud console, go to the Spanner page.

2.  Click the name of the instance that you want to delete, for example, **Test Instance** .

3.  Click **Delete instance** .

4.  Confirm that you want to delete the instance by entering the instance name and clicking **Delete** .

## What's next

  - Learn more about [Data agents overview](/alloydb/docs/ai/data-agent-overview) .
  - Learn how to [define data agent context for database data sources](/gemini/docs/conversational-analytics-api/data-agent-authored-context-databases#cloud-sql) .
