# Metabase Athena Driver

💥*Note:* This project is under active development

[![Latest Release](https://img.shields.io/github/v/release/dacort/metabase-athena-driver.svg?label=latest%20release&include_prereleases)](https://github.com/dacort/metabase-athena-driver/releases)
[![GitHub license](https://img.shields.io/github/license/dacort/metabase-athena-driver)](https://raw.githubusercontent.com/dacort/metabase-athena-driver/master/LICENSE)

## Installation

Beginning with Metabase 0.32, drivers must be stored in a `plugins` directory in the same directory where `metabase.jar` is, or you can specify the directory by setting the environment variable `MB_PLUGINS_DIR`.

### Download Metabase Jar and Run

1. Download a fairly recent Metabase binary release (jar file) from the [Metabase distribution page](https://metabase.com/start/jar.html).
2. Download the Athena driver jar from this repository's "Releases" page
3. Create a directory and copy the `metabase.jar` to it.
4. In that directory create a sub-directory called `plugins`.
5. Copy the Athena driver jar to the `plugins` directory.
6. Make sure you are the in the directory where your `metabase.jar` lives.
7. Run `java -jar metabase.jar`.

### Build from source

I'm not familiar enough with `lein` to know if there is a better way to include a jar from a static URL, so for the time being we download it manually.

1. Download a fairly recent Metabase binary release (jar file) from the [Metabase distribution page](https://metabase.com/start/jar.html).

2. Download the Athena driver into your local Maven repo
   ```shell
   mkdir -p ~/.m2/repository/athena/athena-jdbc/2.0.7/
   wget -O ~/.m2/repository/athena/athena-jdbc/2.0.7/athena-jdbc-2.0.7.jar https://s3.amazonaws.com/athena-downloads/drivers/JDBC/SimbaAthenaJDBC_2.0.7/AthenaJDBC42_2.0.7.jar 
   ```

3. Clone this repo
   ```shell
   git clone https://github.com/dacort/metabase-athena-driver
   ```

4. Build the jar
   ```shell
   cd metabase-athena-driver/
   DEBUG=1 LEIN_SNAPSHOTS_IN_RELEASE=true lein uberjar
   ```

5. Let's assume we download `metabase.jar` to `~/metabae/` and we built the project above. Copy the built jar to the Metabase plugins directly and run Metabase from there!
   ```shell
   TARGET_DIR=~/metabae
   mkdir ${TARGET_DIR}/plugins/
   cp target/uberjar/athena.metabase-driver.jar ${TARGET_DIR}/plugins/
   cd ${TARGET_DIR}/
   java -jar metabase.jar
   ```

You should see a message on startup similar to:

```
2019-05-07 23:27:32 INFO plugins.lazy-loaded-driver :: Registering lazy loading driver :athena...
2019-05-07 23:27:32 INFO metabase.driver :: Registered driver :athena (parents: #{:sql-jdbc}) 🚚
```

## Configuring

Once you've started up Metabase, go to add a database and select "Athena".

You'll need to provide the AWS region, an access key and secret key, and an S3 bucket and prefix where query results will be written to.

Please note:
- The provided bucket must be in the same region you specify.
- If you do _not_ provide an access key, the [default credentials chain](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/DefaultAWSCredentialsProviderChain.html).
- The initial sync can take some time depending on how many databases and tables you have.

## Testing

Only one test exists at the moment, but can easily be run:

```shell
lein test
```

## Known Issues

- Cannot specify a single database to sync
- ~~Only native SQL queries are supported~~
  - Native SQL Queries must not end with a semi-colon (`;`)
  - Basic aggregations seem to work in the query builder
  - ~~Parameterized queries are not supported~~
- Sometimes, the initial database verification can time out
  - If this happens, configure a higher timeout value with the `MB_DB_CONNECTION_TIMEOUT_MS` environment variable
- ~~Heavily nested fields can result in a `StackOverflowError`~~
  - ~~If this happens, increase the `-Xss` JVM parameter~~