# Stream data to a downstream database

[Reference Blog Post](https://debezium.io/blog/2017/09/25/streaming-to-another-database/)

This example shows a simple streaming data pipeline to continuously capture changes in a Azure SQL Database and replicate in to another Azure SQL Database.

> The CDC feature is available for Azure databases higher than the S3 (Standard 3) tier. [Read more](https://techcommunity.microsoft.com/t5/azure-sql/introducing-change-data-capture-for-azure-sql-databases-public/ba-p/2429842)

## Configuration

The docker compose file consists of the following docker images:

- Apache Zookeeper
- Apache Kafka
- Apache Kafka Connect
  - Added the Kafka Connect JDBC Connector and MSSQL JDBC Driver

### Source Connector Configuration

```json
{
  "name": "user-connector", 
  "config": {
      "connector.class": "io.debezium.connector.sqlserver.SqlServerConnector", 
      "database.hostname": "<hostname>", 
      "database.port": "1433", 
      "database.user": "<username>", 
      "database.password": "<password>", 
      "database.dbname": "<db-name>", 
      "database.server.name": "<server-name>", 
      "table.include.list": "dbo.<your-table>", 
      "database.history.kafka.bootstrap.servers": "kafka:9092", 
      "database.history.kafka.topic": "dbhistory.<your-table>" 
  }
}
```

### Sink Connector Configuration

```json
{
  "name": "sql-sink-connector",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
    "tasks.max": "1",
    "topics": "<topics>",
    "connection.url": "<get connection string e.g. from azure portal>",
    "transforms": "dropPrefix,unwrap",
    "transforms.dropPrefix.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.dropPrefix.regex": "<dbserver>\\.dbo\\.(.*)",
    "transforms.dropPrefix.replacement": "$1",
    "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
    "transforms.unwrap.drop.tombstones": false,
    "auto.create": true,
    "insert.mode": "upsert",
    "delete.enabled": true,
    "pk.mode": "record_key"
  }
}
```

## Example

### With Docker Compose

- Start the environment: ```docker compose up```
- Add the connectors with the specific POST-Requests (see [here](debezium-requests.http) 

Try DML-Statements in your source database and check the logs if the changes were also replicated in your downstream database.

## Manually start containers

### start zookeeper

```docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.6```

### start kafka

```docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:1.6```

### start kafka connect

```docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka debezium/connect:1.6```

### watch for events

```docker run -it --rm --name watcher --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:1.6 watch-topic -a -k <your-topic>```