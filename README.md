
------

## Source from PostgreSQL to Kafka Topic by Kafka Connect

### 1. Preconditions

- Install PostgreSQL Server
- Install Confluent Platform

### 2. Create Database
```
CREATE DATABASE demo;
```

### 3. Create table

```
CREATE TABLE users
(
    id          SERIAL NOT NULL,
    name        VARCHAR(100) NOT NULL,
    age         INTEGER,
    updated_at  TIMESTAMP NOT NULL DEFAULT NOW(),
    PRIMARY KEY(id)
);
```

### 4. Insert data

```sql
INSERT INTO users (name, age) VALUES ('a', 26);
INSERT INTO users (name, age) VALUES ('b', 24);
INSERT INTO users (name, age) VALUES ('c', 25);
```

### 5. Start Confluent server (kafka, zookeeper, schema-registry)

```
$ ./confluent start schema-registry
```

### 7. Create Kafka Topic
```
$ ./kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic postgres_users
```

### 8. List all Kafka topics
```
$ ./kafka-topics --lost --zookeeper localhost:2181
```

### 9. create source-postgres.properties file
```
cp {/path/to/confluent}/etc/kafka-connect-jdbc/source-quickstart-sqlite.properties {/path/to/confluent}/etc/kafka-connect-jdbc/source-postgres.properties
```

#### source-postgres.properties file content:
```
#name for the connector
name=source-postgres

connector.class=io.confluent.connect.jdbc.JdbcSourceConnector

tasks.max=1

# JDBC endpoint for the PostgreSQL database
connection.url=jdbc:postgresql://localhost:5432/demo?user=postgres&password=password

#Incremental Query Modes, do not use 'timestamp+incrementing' mode
mode=incrementing

#the column name which has the timestamps
timestamp.column.name=updated_at

#the column which has incremental IDs
incrementing.column.name=id

#to identify the Kafka topics ingested from PostgreSQL
topic.prefix=postgres_
```

### 10. Start Standalone Kafka Connect
```
$ ./connect-standalone ../etc/schema-registry/connect-avro-standalone.properties ../etc/kafka-connect-jdbc/source-postgres.properties
```

### 11. Start Kafka Consumer
```
$ ./kafka-avro-console-consumer --new-consumer --bootstrap-server localhost:9092 --topic postgres_users --from-beginning
```

PS. How to check Kafka messages length towards a Topic
```
./kafka-run-class kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic postgres_users --time -1
```

#### Reference
[Syncing Redshift & PostgreSQL in real-time with Kafka Connect][1]

  [1]: https://blog.insightdatascience.com/from-postgresql-to-redshift-with-kafka-connect-111c44954a6a
