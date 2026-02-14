This page shows how to connect PGAdapter to the [Spanner emulator](/spanner/docs/emulator) . The emulator runs locally, and you can use to it to develop and test your applications without creating a Google Cloud project or a billing account. As the emulator stores data only in memory, all state, including data, schema, and configs, is lost on restart. The emulator offers the same APIs as the Spanner production service and is intended for local development and testing, not for production deployments.

You can connect PGAdapter to the emulator in three different ways:

  - Run a prebuilt combined Docker container with both PGAdapter and the emulator.
  - Run both the emulator and PGAdapter on your local machine.
  - Run both the emulator and PGAdapter in a Docker network.

## Combined Docker container

Run the prebuilt Docker image that contains both PGAdapter and the emulator. The PGAdapter instance in the Docker container is automatically configured to connect to the emulator in the container.

Start the Docker container with the following command.

``` text
docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter-emulator
docker run -d \
  -p 5432:5432 \
  -p 9010:9010 \
  -p 9020:9020 \
  gcr.io/cloud-spanner-pg-adapter/pgadapter-emulator
```

You don't need to specify a project, instance, or database name when starting the container. It will by default use the project name `  emulator-project  ` and instance name `  test-instance  ` . Any database that you connect to is automatically created by PGAdapter.

You can connect to PGAdapter and execute statements with `  psql  ` with the following command.

``` text
psql -h localhost -p 5432 -d test-database
```

The database `  test-database  ` is automatically created. Execute these statements to verify that you are connected to a PostgreSQL-dialect database database:

``` text
create table my_table (
  id bigint not null primary key,
  value varchar
);
insert into my_table (id, value) values (1, 'One');
select * from my_table;
```

You can also connect directly to the emulator inside the Docker container, for example with the gcloud CLI.

``` text
gcloud config configurations create emulator
gcloud config set auth/disable_credentials true
gcloud config set project emulator-project
gcloud config set api_endpoint_overrides/spanner http://localhost:9020/
gcloud spanner instances list
gcloud spanner databases execute-sql test-database \
  --instance=test-instance \
  --sql="select * from my_table"
```

## Run the emulator and PGAdapter on your local machine

You can run both the emulator and PGAdapter on your local machine, and connect PGAdapter to the emulator with the following commands.

First start the emulator.

``` text
gcloud emulators spanner start
```

Then start PGAdapter and connect it to the emulator. Note that you must start PGAdapter as a Java application on your local machine for it to be able to access the emulator. If you start PGAdapter in a Docker container, it isn't able to access the emulator that is running on your local host.

``` text
wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter.tar.gz \
  && tar -xzvf pgadapter.tar.gz
java -jar pgadapter.jar -p emulator-project -i test-instance \
  -c "" \
  -r autoConfigEmulator=true
```

The additional command-line arguments for PGAdapter that are used to connect to the emulator are:

  - `  -c ""  ` : This instructs PGAdapter to not use any credentials.
  - `  -r autoConfigEmulator=true  ` : This instructs PGAdapter to connect to `  localhost:9010  ` , which is the default emulator host and port. It also instructs PGAdapter to automatically create any database that a user connects to. This means that you don't need to create a database before connecting to it.

You can connect to PGAdapter and execute statements with `  psql  ` with the following command.

``` text
psql -h localhost -p 5432 -d test-database
```

The database `  test-database  ` is automatically created. Execute these statements to verify that you are connected to a PostgreSQL-dialect database database:

``` text
create table my_table (
  id bigint not null primary key,
  value varchar
);
insert into my_table (id, value) values (1, 'One');
select * from my_table;
```

## Run the emulator and PGAdapter in a Docker network

You can run both the emulator and PGAdapter in a Docker network and connect PGAdapter to the emulator with the following commands.

``` text
cat <<EOT > docker-compose.yml
version: "3.9"
services:
  emulator:
    image: "gcr.io/cloud-spanner-emulator/emulator"
    pull_policy: always
    container_name: spanner-emulator
    ports:
      - "9010:9010"
      - "9020:9020"
  pgadapter:
    depends_on:
      emulator:
        condition: service_started
    image: "gcr.io/cloud-spanner-pg-adapter/pgadapter"
    pull_policy: always
    container_name: pgadapter-connected-to-emulator
    command:
      - "-p emulator-project"
      - "-i test-instance"
      - "-r autoConfigEmulator=true"
      - "-e emulator:9010"
      - "-c \"\""
      - "-x"
    ports:
      - "5432:5432"
EOT
docker compose up -d
```

Both PGAdapter and the emulator are started in the same Docker network, and PGAdapter is configured to connect to the emulator. The additional command-line arguments for PGAdapter that are used to connect to the emulator are:

  - `  -c ""  ` : This instructs PGAdapter to not use any credentials.
  - `  -r autoConfigEmulator=true  ` : This instructs PGAdapter to automatically create any database that a user connects to. This means that you don't need to create a database before connecting to it.
  - `  -e emulator:9010  ` : `  -e  ` specifies the endpoint that PGAdapter should connect to. `  emulator:9010  ` is the name and port number of the emulator in the same Docker network.
  - `  -x  ` : This allows connections to PGAdapter from your local machine.

You can connect to PGAdapter and execute statements with `  psql  ` with the following command.

``` text
psql -h localhost -p 5432 -d test-database
```

The database `  test-database  ` is automatically created. Execute these statements to verify that you are connected to a PostgreSQL-dialect database database:

``` text
create table my_table (
  id bigint not null primary key,
  value varchar
);
insert into my_table (id, value) values (1, 'One');
select * from my_table;
```

You can also connect directly to the emulator in the Docker network, for example with the gcloud CLI.

``` text
gcloud config configurations create emulator
gcloud config set auth/disable_credentials true
gcloud config set project emulator-project
gcloud config set api_endpoint_overrides/spanner http://localhost:9020/
gcloud spanner instances list
gcloud spanner databases execute-sql test-database \
  --instance=test-instance \
  --sql="select * from my_table"
```
