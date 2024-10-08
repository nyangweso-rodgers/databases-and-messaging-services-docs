# Kafka Connect

## Table Of Contents

# What is Kafka Connect

- **Kafka Connect** is a component of **Apache Kafka** that’s used to perform streaming integration between **Kafka** and other systems such as **databases**, **cloud services**, **search indexes**, **file systems**, and **key-value stores**. It makes it simple to quickly define **connectors** that move large data sets in and out of **Kafka**.

# Configuring Converters (`key` and `value` Converters)

- **Converters** are responsible for the **serialization** and **deserialization** of data flowing between **Kafka Connect** and **Kafka** itself.
- There are a ton of different **converters** available, but some common ones include:
  1. **Avro** - `io.confluent.connect.avro.AvroConverter`: use with **Schema Registry**
  2. **Protobuf** – `io.confluent.connect.protobuf.ProtobufConverter`: use with **Schema Registry**
  3. **String** – `org.apache.kafka.connect.storage.StringConverter`: simple string format
  4. **JSON** – `org.apache.kafka.connect.json.JsonConverter`: (without Schema Registry): use with structured data
  5. **JSON Schema** – `io.confluent.connect.json.JsonSchemaConverter`: use with **Schema Registry**
  6. **ByteArray** – `org.apache.kafka.connect.converters.ByteArrayConverter`: provides a “pass-through” option that does no conversion
- **Remark**:
  - Remember, **Kafka** messages are just pairs of **key/value** bytes, and you need to specify the **converter** for both **keys** and **value**, using the `key.converter` and `value.converter` configuration setting. In some situations, you may use different **converters** for the **key** and the **value**.
  - The `key.converter` and `value.converter` properties are where you specify the type of **converter** to use.

# JSON Schema

- Example configuration using `JsonSchemaConverter` for the value converter and using `StringConverter` for the key:
  ```json
    key.converter=org.apache.kafka.connect.storage.StringConverter
    value.converter=io.confluent.connect.json.JsonSchemaConverter
    value.converter.schema.registry.url=http://localhost:8081
  ```



# Resources And Further Reading

1. [docs.confluent.io - Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html)
