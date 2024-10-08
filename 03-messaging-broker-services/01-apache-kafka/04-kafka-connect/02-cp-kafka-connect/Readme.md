# cp-kafka-connect

## Table Of Contents

# What is Kafka Connect

- **Kafka Connect** is a tool for scalably and reliably streaming data between **Apache Kafka** and other data systems. It makes it simple to quickly define **connectors** that move large data sets in and out of **Kafka**. **Kafka Connect** can ingest entire databases or collect metrics from all your application servers into **Kafka topics**, making the data available for stream processing with low latency. An export **connector** can deliver data from **Kafka topics** into secondary indexes like Elasticsearch, or into batch systems–such as Hadoop for offline analysis.

# Benefits of cp-kafka-connect

# Modes Of Execution

- **Kafka Connect** currently supports two modes of execution:
  1. standalone (single process) and
  2. distributed.
- In **standalone mode**, all work is performed in a single process it's simpler to get started with and may be useful in situations where only one worker makes sense (e.g. collecting log files), but it does not benefit from some of the features of **Kafka Connect** such as fault tolerance.
- **Distributed mode** handles automatically the balancing of work, allows you to scale up (or down) dynamically, and offers fault tolerance both in the active tasks and for configuration and offset commit data.

# Kafka Connect Concepts

## 1. Connectors

- **Kafka Connect** includes two types of connectors:
  1. **Source Connector**: ingest entire databases and stream table updates to **Kafka topics**. **Source connectors** can also collect metrics from all your application servers and store the data in Kafka topics–making the data available for stream processing with low latency.
  2. **Sink Connector**: deliver data from **Kafka topics** to secondary indexes, such as Elasticsearch, or batch systems such as Hadoop for offline analysis.
- Remark:
  - Confluent offers several [pre-built connectors](https://www.confluent.io/product/connectors/?session_ref=https://www.google.com/&_ga=2.266203727.1350022208.1723832910-234518971.1709664712&_gl=1*ss8h9o*_gcl_au*OTIxNjA2MDMuMTcxNzYxMTg4NA..*_ga*MjM0NTE4OTcxLjE3MDk2NjQ3MTI.*_ga_D2D3EGKSGD*MTcyMzk5MDkzNi4xNDQuMS4xNzIzOTkxOTE1LjQ4LjAuMA..) that can be used to stream data to or from commonly used systems.

## 2. Task

- **Tasks** are independent units of work that enable parallel data processing in **Kafka Connect**. When a **connector** is created, **Kafka Connect** divides its work into multiple **tasks** based on the configured level of parallelism.

### Task Assignment

- Each **task** processes a subset of the data for a **connector**. For example, if the source system is a database table, you may assign each task a partition of the table data based on some criteria like a column value.
- Some ways in which **tasks** can be assigned partitions:
  1. Round-robin for non-partitioned tables
  2. Hash partitioning based on the hash of the primary key column
  3. Time-based date or timestamp ranges
  4. Custom like any application-specific logic
- If there are no natural partitions, **Kafka Connect** distributes partitions across tasks randomly using round-robin.

### Task Configuration

- **Tasks** inherit most configurations from their parent connector but also allow some custom settings such as:
  1. **transforms** - Data manipulation logic
  2. **converters** - Serialization formats
- Examples:

  - MySQL Connector (This MySQL source connector with four tasks inserts a "topic" field to records using a transformation before publishing to Kafka)

    ```json
    {
      "connector.class": "MySqlSourceConnector",

      "tasks.max": "4",

      "transforms": "insertTopic",

      "transforms.insertTopic.type": "org.apache.kafka.connect.transforms.InsertField$Value",

      "transforms.insertTopic.topic.field": "topic"
    }
    ```

## 3. Workers

- A **worker** denotes a server capable of executing **connectors** and **tasks**, overseeing their lifecycle, and offering scalability.

## 4. Converters

- **Converters** are required to have a **Kafka Connect** deployment support a particular data format when writing to, or reading from **Kafka**. Tasks use **converters** to change the format of data from bytes to a Connect internal data format and vice versa.
- By default, **Confluent Platform** provides the following converters:
  1. **AvroConverter** `io.confluent.connect.avro.AvroConverter`: use with [Schema Registry]()
  2. **ProtobufConverter**: `io.confluent.connect.protobuf.ProtobufConverter`: use with [Schema Registry]()
  3. **JsonSchemaConverter** `io.confluent.connect.json.JsonSchemaConverter`: use with [Schema Registry]()
  4. **JsonConverter** `org.apache.kafka.connect.json.JsonConverter` (without Schema Registry): use with structured data
  5. **StringConverter** `org.apache.kafka.connect.storage.StringConverter`: simple string format
  6. **ByteArrayConverter** `org.apache.kafka.connect.converters.ByteArrayConverter`: provides a “pass-through” option that does no conversion
- Remark:
  - **Converters** are decoupled from **connectors** themselves to allow for the reuse of **converters** between **connectors**. For example, using the same **Avro converter**, the **JDBC Source Connector** can write **Avro** data to **Kafka**, and the HDFS Sink Connector can read Avro data from Kafka

### 4.1: Using String Converter (`StringConverter`)

- Here’s an example of using the **String converter**. Since it’s just a **string**, there’s no **schema** to the data, and thus it’s not so useful to use for the **value**:
  ```json
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
  ```

### 4.2: Using JSON Converter (`JsonConverter`)

- For **JSON**, you need to specify if you want **Kafka Connect** to embed the **schema** in the **JSON** itself.
- Whilst **JSON** does not by default support carrying a **schema**, **Kafka Connect** supports two ways that you can still have a declared **schema** and use **JSON**:
  - The first is to use **JSON Schema** with the **Confluent Schema Registry**.
  - If you cannot use the **Schema Registry** then your second (less optimal option) is to use **Kafka Connect’s** support of a particular structure of **JSON** in which the **schema** is embedded. The resulting data size can get large as the **schema** is included in every single message along with the **schema**.
- If you’re setting up a **Kafka Connect source** and want **Kafka Connect** to include the **schema** in the message it writes to **Kafka**, you’d set:
  ```json
    "value.converter":"org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable":true
  ```
- The resulting message to **Kafka** would look like the example below, with **schema** and **payload** top-level elements in the **JSON**:
  ```json
  {
    "schema": {
      "type": "struct",
      "fields": [
        {
          "type": "int64",
          "optional": false,
          "field": "registertime"
        },
        {
          "type": "string",
          "optional": false,
          "field": "userid"
        },
        {
          "type": "string",
          "optional": false,
          "field": "regionid"
        },
        {
          "type": "string",
          "optional": false,
          "field": "gender"
        }
      ],
      "optional": false,
      "name": "ksql.users"
    },
    "payload": {
      "registertime": 1493819497170,
      "userid": "User_1",
      "regionid": "Region_5",
      "gender": "MALE"
    }
  }
  ```
- **Note** the size of the message, as well as the proportion of it that is made up of the **payload** vs. the **schema**. Considering that this is repeated in every message, you can see why a serialisation format like **JSON Schema** or **Avro** makes a lot of sense, as the **schema** is stored separately and the message holds just the payload (and is compressed at that).
- **Remark**:
  - If you’re consuming **JSON** data from a **Kafka topic** into a **Kafka Connect sink**, you need to understand how the **JSON** was serialised. If it was with **JSON Schema serialiser**, then you need to set **Kafka Connect** to use the **JSON Schema converter** (`io.confluent.connect.json.JsonSchemaConverter`). If the **JSON** data was written as a plain **string**, then you need to determine if the data includes a nested **schema**. If it does—and it’s in the same format as above, not some arbitrary schema-inclusion format—then you’d set:
    ```json
      "value.converter":"org.apache.kafka.connect.json.JsonConverter",
      "value.converter.schemas.enable":true
    ```
  - However, if you’re consuming **JSON** data and it doesn’t have the **schema/payload** construct, such as this sample:
    ```json
    {
      "registertime": 1489869013625,
      "userid": "User_1",
      "regionid": "Region_2",
      "gender": "OTHER"
    }
    ```
  - …you must tell **Kafka Connect** not to look for a **schema** by setting `schemas.enable:false`:
    ```json
      "value.converter":"org.apache.kafka.connect.json.JsonConverter",
      "value.converter.schemas.enable":false
    ```
  - As before, remember that the **converter** configuration option (here, `schemas.enable`) needs the prefix of `key.converter` or `value.converter` as appropriate.

### 4.3: Using Avro Converter (`AvroConverter`)

- Some **converters** have additional configuration. For **Avro**, you need to specify the **Schema Registry**.
- When you specify converter-specific configurations, always use the `key.converter.` or `value.converter.` prefix. For example, to use **Avro** for the message payload, you’d specify the following:
  ```json
    "value.converter": "io.confluent.connect.avro.AvroConverter",
    "value.converter.schema.registry.url": "http://schema-registry:8081"
  ```

## 5. Transforms

- **Connectors** can be configured with **transformations** to make simple and lightweight modifications to individual messages.
- A **transform** is a simple function that accepts one record as an input and outputs a modified record. All transforms provided by Kafka Connect perform simple but commonly useful modificationss

### 5.1: Single Message Transforms (SMTs)

- **SMTs** allow you to modify the records in real-time before they are sent to the **Kafka topic**.
- Common **SMTs** include:
  1. `ExtractField`: Extracts a specific field from the **Key** or **Value**.
  2. `ReplaceField`: Removes or renames fields within a record.
  3. `ValueToKey`: Promotes one or more fields from the Value to the Key.
  4. `Cast`: Converts a field’s data type.
  5. `InsertField`: Adds a field with a specified value to the record.
  6. `MaskField`: Replaces a field’s value with a masked value (useful for sensitive data).
  7. `Filter`: Filters records based on conditions.
- Examles:
  1. `transforms": "InsertKey, ExtractId, CastLong"`
  2. `"transforms.InsertKey.type":`
  3. `"transforms.InsertKey.fields": "id"`
  4. `"transforms.ExtractId.type":`
  5. `"transforms.ExtractId.field": "id"`
  6. `"transforms.CastLong.type":`
  7. `"transforms.CastLong.spec": "int64"`
- Examle Scenario:
  1. Promoting a Field from Value to Key
     - You have a `customers` table, and each record has a `customer_id` in the **Value**. You want to use `customer_id` as the **Key** in **Kafka**.
     - SMT: `ValueToKey`
     - Configuration:
       ```json
       {
         "transforms": "InsertKey",
         "transforms.InsertKey.type": "org.apache.kafka.connect.transforms.ValueToKey",
         "transforms.InsertKey.fields": "customer_id"
       }
       ```
     - Outcome: The `customer_id` field is promoted from the Value to the Key of the record.
  2. Removing Unnecessary Fields
     - You want to remove sensitive fields like `ssn` from the **Value** before the data is published to **Kafka**.
     - SMT: `ReplaceField`
     - Configuration:
       ```json
       {
         "transforms": "RemoveSSN",
         "transforms.RemoveSSN.type": "org.apache.kafka.connect.transforms.ReplaceField$Value",
         "transforms.RemoveSSN.blacklist": "ssn"
       }
       ```
     - Outcome: The ssn field is removed from the Value.
  3. Converting a Field’s Data Type
     - You have a `timestamp` field stored as a `string`, and you want to convert it to an `integer` (epoch time).
     - SMT: `Cast`
     - Configuration
       ```json
       {
         "transforms": "CastTimestamp",
         "transforms.CastTimestamp.type": "org.apache.kafka.connect.transforms.Cast$Value",
         "transforms.CastTimestamp.spec": "timestamp:int64"
       }
       ```
     - Outcome: The timestamp field is converted to a long integer representing epoch time.
  4. Filtering Out Records
     - You only want to send records where the status field is set to `active`.
     - SMT: `Filter`
     - Configuration
       ```json
       {
         "transforms": "FilterInactive",
         "transforms.FilterInactive.type": "org.apache.kafka.connect.transforms.Filter$Value",
         "transforms.FilterInactive.condition": "status == 'active'"
       }
       ```
     - Outcome: Only records with status set to active will be passed through; others will be filtered out.

## 6. Dead Letter Queues

- **Dead Letter Queues** (DLQs) are only applicable for **sink connectors**.
- An invalid record may occur for a number of reasons. One example is when a record arrives at a **sink connector** serialized in **JSON** format, but the **sink connector** configuration is expecting **Avro** format. When an invalid record can’t be processed by the **sink connector**, the error is handled based on the connector `errors.tolerance` configuration property.
- To create a **DLQ**, add the following configuration properties to your **sink connector** configuration:
  ```json
    errors.tolerance = all
    errors.deadletterqueue.topic.name = <dead-letter-topic-name>
  ```
- Even if the **DQL topic** contains the records that failed, it does not show why. You can add the following configuration property to include failed record header information.
  ```json
    errors.deadletterqueue.context.headers.enable=true
  ```

## 7. Kafka Connect Plugin

- A **Kafka Connect plugin** is a set of **JAR files** containing the implementation of one or more **connectors**, **transforms**, or **converters**. **Connect** isolates each **plugin** from one another so libraries in one **plugin** are not affected by the libraries in any other **plugins**. This is very important when mixing and matching **connectors** from multiple providers.
- A **Kafka Connect plugin** can be any one of the following:
  1. A directory on the file system that contains all required **JAR files** and third-party dependencies for the **plugin**. This is most common and is preferred.
  2. A single uber JAR containing all the class files for a plugin and its third-party dependencies.
- To install a **plugin**, you must place the **plugin directory** in a directory already listed in the plugin path.
- To find the components that fit your needs, check out the [Confluent Hub](https://www.confluent.io/hub/?session_ref=https://www.google.com/&_gl=1*1sogvi8*_gcl_au*OTIxNjA2MDMuMTcxNzYxMTg4NA..*_ga*MjM0NTE4OTcxLjE3MDk2NjQ3MTI.*_ga_D2D3EGKSGD*MTcyMzk5MDkzNi4xNDQuMS4xNzIzOTkzMjEzLjYwLjAuMA..&_ga=2.28774524.1350022208.1723832910-234518971.1709664712) page-it has an ecosystem of **connectors**,**transforms**, and **converters**. For a full list of supported **connectors**, see [Supported Self-Managed Connectors](https://docs.confluent.io/platform/current/connect/supported.html)

### 7.1: Avro Plugin

- The **Avro converter** is a **plugin** that allows **Kafka Connect** to handle data in the **Avro format**. It's typically included in pre-built **Kafka Connect** images like `confluentinc/cp-kafka-connect`.
- Remark:
  - If you're using a pre-built image, you generally don't need to install the **Avro plugin** separately. The image should already contain the necessary components.

# Setup

- We will set up:
  1. A postgres database
  2. A local Kafka cluster using Docker Compose
     - with Confluent Schema Registry to support Avro (de)serialization
  3. **Kafka Connect** instance, in distributed mode
     - with AKHQ to more easily create and manage connectors
  4. **Source connector** from the **Postgres** table to a **Kafka topic**
     - with database connection password as environment variable
  5. JMX metrics exported and scraped by Prometheus for monitoring
  6. **Grafana** dashboard to visualize & alert Connector status

# Running Kafka Connect in Docker

- [Confluent]() maintains its own image for **Kafka Connect**,[confluentinc/cp-kafka-connect](https://hub.docker.com/r/confluentinc/cp-kafka-connect), which provides a basic Connect worker to which you can add your desired **JAR files** for **sink** and **source connectors**, single message **transforms**, and **converters**.
- **Note**:

  - Starting with Confluent Platform version 6.0 release, many **connectors** previously bundled with Confluent Platform are now available for download from [Confluent Hub](https://www.confluent.io/hub/?_ga=2.262009677.1350022208.1723832910-234518971.1709664712&_gl=1*11la3pl*_gcl_au*OTIxNjA2MDMuMTcxNzYxMTg4NA..*_ga*MjM0NTE4OTcxLjE3MDk2NjQ3MTI.*_ga_D2D3EGKSGD*MTcyNDE1OTU2My4xNDcuMS4xNzI0MTYwNTU1LjM1LjAuMA..). For more information, see the [6.0 Connector Release Notes](https://docs.confluent.io/platform/current/release-notes/index.html#connectors).

- Configure **Kafka Connect**:

# Adding Connectors to a Docker Container

- You can use [Confluent Hub]() to add your desired **JARs**, either by installing them at runtime or by creating a new Docker image. Of course, there are pros and cons to either of these options, and you should choose based on your individual needs.

## Create a Docker Image Containing Confluent Hub Connectors

- Add connectors from [Confluent Hub](https://www.confluent.io/hub/?_ga=2.263155916.1350022208.1723832910-234518971.1709664712&_gl=1*1ttjnnz*_gcl_au*OTIxNjA2MDMuMTcxNzYxMTg4NA..*_ga*MjM0NTE4OTcxLjE3MDk2NjQ3MTI.*_ga_D2D3EGKSGD*MTcyNDE1OTU2My4xNDcuMS4xNzI0MTYyNjA2LjEuMC4w)

- Write a `Dockerfile` with the connectors as follows:

  ```Dockerfile
    FROM confluentinc/cp-server-connect-base:7.7.0
    # FROM confluentinc/cp-kafka-connect:7.1.0-1-ubi8

    ENV CONNECT_PLUGIN_PATH: "/usr/share/java,/usr/share/confluent-hub-components"

    RUN confluent-hub install --no-prompt hpgrahsl/kafka-connect-mongodb:1.1.0
    RUN confluent-hub install --no-prompt microsoft/kafka-connect-iothub:0.6
    RUN confluent-hub install --no-prompt wepay/kafka-connect-bigquery:1.1.0
    RUN confluent-hub install --no-prompt neo4j/kafka-connect-neo4j:2.0.2
  ```

- **Remarks**:
  - The `CONNECT_PLUGIN_PATH` environment variable is set to `/usr/share/java,/usr/share/confluent-hub-components`, which are the paths where **Kafka Connect** looks for **plugins**.
- Build a `Dockerfile`

## Get Available Connector Plugins

- Get available connectro plugins by:

  ```sh
    curl localhost:8083/connector-plugins | json_pp
  ```

- If you need to check the list of available **plugins** you should hit `localhost:8083/connector-plugins`

  ```sh
      curl localhost:8083/connector-plugins
  ```

- The `curl` command you ran is querying the **Kafka Connect** REST API to list the available **connector plugins** on your **Kafka Connect** instance.
- Examle Output:

  ```json
  [
    {
      "class": "io.confluent.connect.jdbc.JdbcSinkConnector",
      "type": "sink",
      "version": "10.7.6"
    },
    {
      "class": "io.confluent.connect.jdbc.JdbcSourceConnector",
      "type": "source",
      "version": "10.7.6"
    },
    {
      "class": "org.apache.kafka.connect.mirror.MirrorCheckpointConnector",
      "type": "source",
      "version": "1"
    },
    {
      "class": "org.apache.kafka.connect.mirror.MirrorHeartbeatConnector",
      "type": "source",
      "version": "1"
    },
    {
      "class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
      "type": "source",
      "version": "1"
    }
  ]
  ```

- Here, the response indicates that several **connector plugins** are available, each identified by its class, type, and version.

  1. `JdbcSinkConnector` and `JdbcSourceConnector`:
     - `io.confluent.connect.jdbc.JdbcSinkConnector` (version 10.7.6): A **sink connector** that allows data to be written from **Kafka topics** into a relational database using **JDBC**.
     - `io.confluent.connect.jdbc.JdbcSourceConnector` (version 10.7.6): A **source connector** that allows data to be ingested from a relational database into **Kafka topics** using **JDBC**
  2. MirrorMaker 2 Connectors:
     - `org.apache.kafka.connect.mirror.MirrorCheckpointConnector` (version 1): Part of **MirrorMaker 2**, this **connector** helps manage the **offsets** in the target cluster, allowing **consumers** to pick up where they left off after a failover.
     - `org.apache.kafka.connect.mirror.MirrorHeartbeatConnector` (version 1): Also part of **MirrorMaker 2**, this **connector** is used for monitoring and ensuring the health and consistency of the data replication process.
     - `org.apache.kafka.connect.mirror.MirrorSourceConnector` (version 1): This **connector** is responsible for replicating data from one **Kafka cluster** to another (cross-cluster mirroring).

- `Dockerfile` for **Kafka Connect** with **plugins**:

  ```Dockerfile
    FROM confluentinc/cp-kafka-connect-base:6.2.1

    # Install Avro & JDBC plugins
    RUN confluent-hub install --no-prompt confluentinc/kafka-connect-avro-converter:5.5.4
    RUN confluent-hub install --no-prompt confluentinc/kafka-connect-jdbc:10.1.1
  ```

- To (de)serialize messages using **Avro** by default, we add the following environment variables.
  ```yml
  # Default converter configuration
  CONNECT_KEY_CONVERTER: "org.apache.kafka.connect.storage.StringConverter"
  CONNECT_VALUE_CONVERTER: "io.confluent.connect.avro.AvroConverter"
  CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL: "http://schema-registry:8081/"
  ```
- Example:
  ```Dockerfile
    # Dockerfile
    # Install Avro plugins
    RUN confluent-hub install --no-prompt confluentinc/kafka-connect-avro-converter:5.5.4
  ```

# Bonus

# JMX metrics exporter

- We add the [Prometheus JMX Exporter agent]() to our **Kafka Connect image**.
  ```Dockerfile
    # Dockerfile
    # Install and configure JMX Exporter
    COPY jmx_prometheus_javaagent-0.15.0.jar /opt/
    COPY kafka-connect.yml /opt/
  ```
- Update `docker-compose.yml` with the following:
  ```yml
  # docker-compose.yml
  # Export JMX metrics to :9876/metrics for Prometheus
  KAFKA_JMX_PORT: "9875"
  KAFKA_OPTS: "-javaagent:/opt/jmx_prometheus_javaagent-0.15.0.jar=9876:/opt/kafka-connect.yml"
  ```

# Resources and Further Reading

1. [docs.confluent.io - Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html)
2. [docs.confluent.ion - How to Use Kafka Connect - Get Started](https://docs.confluent.io/platform/current/connect/userguide.html)
3. [redpanda - Understanding Apache Kafka](https://www.redpanda.com/guides/kafka-tutorial-what-is-kafka-connect)
4. [cloud.google.com/iam/docs/keys-create-delete#console](https://cloud.google.com/iam/docs/keys-create-delete#console)
