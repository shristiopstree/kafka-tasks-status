# Kafka Connect Integration with PostgreSQL and S3

This repository provides a detailed guide on how to set up a Kafka Connect pipeline using the JDBC Source Connector to pull data from PostgreSQL and the S3 Sink Connector to write the data to an S3 bucket. The steps include setting up PostgreSQL, configuring Kafka Connect connectors, and verifying data in Kafka and S3.

## Prerequisites

Make sure you have the following tools and services set up:

- **Docker**: To run PostgreSQL, Kafka, Kafka Connect, and KSQLDB containers.
- **Kafka and Kafka Connect**: Running in Docker or your own environment.
- **PostgreSQL Database**: Running and accessible.
- **S3 Bucket**: For storing data from Kafka.
- **ksqlDB**: Optional, for querying data in Kafka topics.
- **kafkacat**: To consume messages from Kafka topics for verification.

## Setup Instructions


```
git clone https://github.com/shristiopstree/kafka-tasks-status.git

cd kafka-tasks-status/kafka-task-1

docker-compose up -d

CONTAINER ID   IMAGE                                      COMMAND                   CREATED          STATUS                    PORTS                                                 NAMES
5561931056c0   confluentinc/ksqldb-server:0.9.0           "/usr/bin/docker/run"     18 minutes ago   Up 18 minutes             0.0.0.0:8088->8088/tcp, :::8088->8088/tcp             ksqldb
fa08551f5300   edenhill/kafkacat:1.5.0                    "/bin/sh -c 'apk add…"    18 minutes ago   Up 18 minutes                                                                   kafkacat
b52775b7182b   confluentinc/cp-kafka-connect-base:5.5.0   "bash -c 'echo \"Inst…"   18 minutes ago   Up 18 minutes (healthy)   0.0.0.0:8083->8083/tcp, :::8083->8083/tcp, 9092/tcp   kafka-connect
c49b22e479a2   confluentinc/cp-schema-registry:5.5.0      "/etc/confluent/dock…"    18 minutes ago   Up 18 minutes             0.0.0.0:8081->8081/tcp, :::8081->8081/tcp             schema-registry
7b8ea6b70d1f   confluentinc/cp-kafka:5.5.0                "/etc/confluent/dock…"    18 minutes ago   Up 18 minutes             0.0.0.0:9092->9092/tcp, :::9092->9092/tcp             kafka
96e414934e2c   confluentinc/cp-zookeeper:5.5.0            "/etc/confluent/dock…"    18 minutes ago   Up 18 minutes             2181/tcp, 2888/tcp, 3888/tcp                          zookeeper
1876d244b9ff   postgres:12                                "docker-entrypoint.s…"    18 minutes ago   Up 18 minutes             0.0.0.0:5432->5432/tcp, :::5432->5432/tcp             postgres


AWS CONSOLE FOR S3 ACCESS

Attach a policy granting access to the S3 bucket (mykafkabucketconnector) to the user.

Specify the required permissions for S3 (read/write access).


```

### Step 1: Configure the JDBC Source Connector

We will configure a JDBC Source Connector in Kafka Connect to pull data from PostgreSQL and publish it to Kafka.

Run the following `curl` command to configure the JDBC Source Connector:

```
curl -X POST \
  http://localhost:8083/connectors \
  -H "Content-Type: application/json" \
  -d '{
    "name": "jdbc-source",
    "config": {
      "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
      "tasks.max": "1",
      "connection.url": "jdbc:postgresql://postgres:5432/postgres",
      "connection.user": "postgres",
      "connection.password": "postgres",
      "mode": "incrementing",
      "incrementing.column.name": "id",
      "topic.prefix": "postgres-"
    }
  }'

```

This configures the JDBC connector to pull data from the opstree_employees table in PostgreSQL and stream it to Kafka topics with the prefix postgres-.

### **Step 2:** Create and Populate PostgreSQL Table
Next, create the opstree_employees table in PostgreSQL and insert data into it.

Access the PostgreSQL container:

```
docker exec --tty --interactive postgres bash -c 'psql -U $POSTGRES_USER $POSTGRES_DB'


CREATE TABLE opstree_employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    created_at TIMESTAMP
);

CREATE TABLE
postgres=# INSERT INTO opstree_employees (name, email, created_at)
SELECT
    'Name_' || gs AS name,
    'email_' || gs || '@example.com' AS email,
    NOW() AS created_at
FROM generate_series(1, 10000) gs;  -- Generates 10,000 rows
INSERT 0 10000

```


### **Step 3:** Verify the Source Connector in KSQLDB
To verify that the source connector is working, check the connectors in KSQLDB:

```
docker exec -it ksqldb ksql http://ksqldb:8088
Then run the following KSQL query:

show connectors;

show topics;

```
This will list the available connectors, including the JDBC Source Connector you created. Along with the topic you have created in postgres.

### **Step 4:** Configure the S3 Sink Connector
Now, set up the S3 Sink Connector to push the data from Kafka topics to an S3 bucket in Avro format. Use the following curl command to configure the S3 Sink Connector:

```
curl -i -X PUT -H "Accept:application/json" \
  -H "Content-Type:application/json" \
  http://localhost:8083/connectors/sink-s3-voluble/config \
  -d '{
      "connector.class": "io.confluent.connect.s3.S3SinkConnector",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "tasks.max": "1",
      "topics": "postgres-opstree_employees",
      "s3.region": "ap-south-1",
      "s3.bucket.name": "mykafkabucketconnector",
      "flush.size": "100",
      "storage.class": "io.confluent.connect.s3.storage.S3Storage",
      "format.class": "io.confluent.connect.s3.format.avro.AvroFormat",
      "schema.generator.class": "io.confluent.connect.storage.hive.schema.DefaultSchemaGenerator",
      "schema.compatibility": "NONE",
      "partitioner.class": "io.confluent.connect.storage.partitioner.DefaultPartitioner",
      "transforms": "AddMetadata",
      "transforms.AddMetadata.type": "org.apache.kafka.connect.transforms.InsertField$Value",
      "transforms.AddMetadata.offset.field": "_offset",
      "transforms.AddMetadata.partition.field": "_partition"
  }'

```
This will configure the S3 Sink connector to write the data from the specified Kafka topics into an S3 bucket, using Avro format.

### **Step 5:** Verify the S3 Sink Connector in KSQLDB
After configuring the S3 Sink Connector, verify it by running the following command in ksqlDB:

```
docker exec -it ksqldb ksql http://ksqldb:8088
Then run the following KSQL query:

show connectors;

```

This will show the active connectors, including the S3 Sink connector.

### **Step 6:** Consume Data from Kafka Topic
To check if data is flowing correctly from PostgreSQL through Kafka and is being written into the S3 bucket, use the kafkacat tool to consume data from the Kafka topic.

Run the following command to consume messages from the postgres-opstree_employees topic:

```
kafkacat -b localhost:9092 -t postgres-opstree_employees -C -o beginning -f '%o %s\n'
```

This will display the messages in the Kafka topic from the beginning, allowing you to verify the data.


Detailed demo is given in the ppt attached.