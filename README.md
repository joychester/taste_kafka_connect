
------

## Source from PostgreSQL to Kafka Topic by Kafka Connect

### 1. Preconditions

- Install PostgreSQL Server (v10.1)
- Install Confluent Platform (v3.3.1)
- Install Elasticsearch (v5.6.4)

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

### 5. Start Confluent server (kafka, zookeeper, schema-registry, connect, kafka-rest)

```
$ ./confluent start
```

### 7. Create Kafka Topic
```
$ ./kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic postgres-users
```

### 8. List all Kafka topics
```
$ ./kafka-topics --list --zookeeper localhost:2181
```

### 9. create source-postgres.properties file
```
cp {/path/to/confluent}/etc/kafka-connect-jdbc/source-quickstart-sqlite.properties {/path/to/confluent}/etc/kafka-connect-jdbc/source-postgres.properties
```

#### source-postgres.properties file content:
```
#name for the connector
name=source-postgres-test

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
topic.prefix=postgres-
```

### 10. Load Kafka Connect for Sourcing from PostgreSQL 
```
$ ./confluent load source-postgres-test -d ../etc/kafka-connect-jdbc/source-postgres.properties
```

### 11. Start Kafka Consumer
```
$ ./kafka-avro-console-consumer --new-consumer --bootstrap-server localhost:9092 --topic postgres-users --from-beginning
```
### 12. Start Elasticsearch server (v5.6.4 instead of v6.0.0)
```
$ ./elasticsearch
```

### 13. Create sink-es-test.properties file under /etc/kafka-connect-elasticsearch/
```
name=elasticsearch-sink-test
connector.class=io.confluent.connect.elasticsearch.ElasticsearchSinkConnector
tasks.max=1
topics=postgres-users
key.ignore=true
connection.url=http://localhost:9200
type.name=kafka-connect
```
### 14. Load Kafka Connect for Sinking to Elasticsearch 
```
./confluent load elasticsearch-sink-test -d ../etc/kafka-connect-elasticsearch/sink-es-test.properties
```

### 15. Verify the data return from Elasticsearch Rest API
```
GET http://localhost:9200/postgres-users/_search?q=name:<someone>
```

#### How to check Kafka messages length towards a Topic
```
./kafka-run-class kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic postgres-users --time -1
```
#### How to Delete a Topic

First, edit ./etc/kafka/server.properties
```
delete.topic.enable=true
```
Then, remove the Topic by CLI
```
./kafka-topics --zookeeper localhost:2181 --delete --topic <topic_name>
```
#### How to check kafka connect status
```
./confluent status elasticsearch-sink-test
```

#### Reference
[Syncing Redshift & PostgreSQL in real-time with Kafka Connect][1]  
[The Simplest Useful Kafka Connect Data Pipeline In The World][2]  

  [1]: https://blog.insightdatascience.com/from-postgresql-to-redshift-with-kafka-connect-111c44954a6a 
  [2]: https://www.confluent.io/blog/simplest-useful-kafka-connect-data-pipeline-world-thereabouts-part-1/
