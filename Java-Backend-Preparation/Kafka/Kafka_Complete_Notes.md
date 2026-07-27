# Apache Kafka — Complete Tutorial Notes
### From Basics to Advanced — With Code Examples

---

## Table of Contents

1. [Introduction to Kafka](#1-introduction-to-kafka)
2. [Core Concepts & Terminology](#2-core-concepts--terminology)
3. [Kafka Architecture](#3-kafka-architecture)
4. [Installing & Running Kafka](#4-installing--running-kafka)
5. [Topics, Partitions & Offsets](#5-topics-partitions--offsets)
6. [CLI Tools Deep Dive](#6-cli-tools-deep-dive)
7. [Producers](#7-producers)
8. [Consumers](#8-consumers)
9. [Consumer Groups & Rebalancing](#9-consumer-groups--rebalancing)
10. [Serialization & Deserialization](#10-serialization--deserialization)
11. [Schema Registry & Avro](#11-schema-registry--avro)
12. [Replication, Leaders & ISR](#12-replication-leaders--isr)
13. [Delivery Semantics & Idempotence](#13-delivery-semantics--idempotence)
14. [Kafka Transactions & Exactly-Once Semantics](#14-kafka-transactions--exactly-once-semantics)
15. [Log Compaction & Retention](#15-log-compaction--retention)
16. [Kafka Connect](#16-kafka-connect)
17. [Kafka Streams](#17-kafka-streams)
18. [ksqlDB](#18-ksqldb)
19. [Partitioning Strategies & Custom Partitioners](#19-partitioning-strategies--custom-partitioners)
20. [Consumer/Producer Configuration Deep Dive](#20-consumerproducer-configuration-deep-dive)
21. [Security in Kafka](#21-security-in-kafka)
22. [Monitoring & Metrics](#22-monitoring--metrics)
23. [Performance Tuning](#23-performance-tuning)
24. [KRaft Mode (ZooKeeper-less Kafka)](#24-kraft-mode-zookeeper-less-kafka)
25. [Multi-Cluster Replication (MirrorMaker)](#25-multi-cluster-replication-mirrormaker)
26. [Tiered Storage](#26-tiered-storage)
27. [Testing Kafka Applications](#27-testing-kafka-applications)
28. [Deploying Kafka (Docker & Kubernetes)](#28-deploying-kafka-docker--kubernetes)
29. [Client Libraries Across Languages](#29-client-libraries-across-languages)
30. [Common Design Patterns & Use Cases](#30-common-design-patterns--use-cases)
31. [Error Handling & Dead Letter Queues](#31-error-handling--dead-letter-queues)
32. [Kafka vs Other Messaging Systems](#32-kafka-vs-other-messaging-systems)
33. [Best Practices & Cheat Sheet](#33-best-practices--cheat-sheet)

---

## 1. Introduction to Kafka

### 1.1 What is Kafka?
Apache Kafka is a **distributed event streaming platform** used to publish, subscribe to, store, and process streams of records in real time. Think of it as a highly scalable, durable, publish-subscribe messaging system combined with a distributed commit log.

### 1.2 Why Kafka Exists
Traditional message queues (like RabbitMQ) struggled with:
- High-throughput, high-volume event streams
- Replaying old messages
- Multiple independent consumers reading the same data at their own pace

Kafka was built at LinkedIn to solve exactly these problems — it treats data as an **immutable, ordered, replayable log**.

### 1.3 Key Characteristics
- **Durable:** Messages are persisted to disk and replicated.
- **Scalable:** Scales horizontally by adding brokers/partitions.
- **Fast:** Sequential disk I/O + zero-copy transfer gives very high throughput.
- **Fault-tolerant:** Data replicated across multiple brokers.
- **Real-time:** Sub-second latency for publish and consume.

### 1.4 Common Use Cases
- Real-time analytics pipelines
- Log aggregation
- Event-driven microservices communication
- Change Data Capture (CDC)
- Metrics/monitoring pipelines
- Stream processing (fraud detection, recommendation engines)
- Messaging backbone between systems (decoupling producers/consumers)

### 1.5 The Kafka Ecosystem
| Component | Purpose |
|---|---|
| Kafka Broker | Core server storing & serving data |
| Producer API | Publish records to topics |
| Consumer API | Subscribe & process records |
| Kafka Connect | Integrate with external systems (DBs, S3, etc.) |
| Kafka Streams | Stream processing library |
| ksqlDB | SQL-like stream processing engine |
| Schema Registry | Manage/enforce message schemas |
| ZooKeeper / KRaft | Cluster metadata & coordination |

---

## 2. Core Concepts & Terminology

| Term | Meaning |
|---|---|
| **Topic** | A named stream/category of records (like a table or folder) |
| **Partition** | An ordered, immutable sequence of records within a topic; unit of parallelism |
| **Offset** | A unique, sequential ID identifying a record's position within a partition |
| **Broker** | A single Kafka server that stores data and serves client requests |
| **Cluster** | A group of brokers working together |
| **Producer** | Client application that publishes (writes) records to topics |
| **Consumer** | Client application that reads records from topics |
| **Consumer Group** | A set of consumers cooperating to consume a topic, sharing partitions |
| **Replica** | A copy of a partition stored on another broker for fault tolerance |
| **Leader / Follower** | Each partition has one leader replica (handles reads/writes) and follower replicas that copy it |
| **ISR (In-Sync Replicas)** | Set of replicas fully caught up with the leader |
| **Zookeeper / KRaft controller** | Manages cluster metadata, leader election, configuration |
| **Record (Message)** | Key-value pair + timestamp + headers, the basic unit of data |

### 2.1 Analogy for Beginners
Think of a Kafka **topic** like a YouTube channel's playlist:
- The **partition** is like splitting that playlist across multiple physical shelves for faster access.
- The **offset** is the video's position number in that shelf.
- **Producers** are content uploaders.
- **Consumers** are viewers, each remembering where they left off (their offset).
- Multiple viewers can watch independently, at their own pace, and can even rewind — unlike a traditional queue where once a message is picked up, it's gone.

---

## 3. Kafka Architecture

### 3.1 High-Level Diagram (conceptual)
```
Producers -> [ Kafka Broker 1 | Broker 2 | Broker 3 ] -> Consumers
                     |
              Controller / KRaft or ZooKeeper (metadata & coordination)
```

### 3.2 Brokers
Each broker:
- Stores a subset of partitions for various topics
- Handles produce/fetch requests from clients
- Participates in replication

A cluster typically has 3+ brokers for fault tolerance.

### 3.3 Partitions & Parallelism
A topic can have many partitions distributed across brokers, enabling:
- Parallel writes (multiple producers writing to different partitions)
- Parallel reads (multiple consumers in a group, one per partition)

### 3.4 Controller
One broker acts as the **controller**, responsible for:
- Partition leader election
- Propagating metadata changes to other brokers

### 3.5 Storage Model — The Commit Log
Each partition is stored as an append-only log on disk, split into **segments** (files). Kafka never modifies old data in-place (except during compaction) — it only appends.

```
Partition 0 log segments:
00000000000000000000.log
00000000000000100000.log
00000000000000200000.log
```

### 3.6 Request Flow
1. Producer sends record → determines partition (via key hash or round-robin) → sends to partition leader's broker.
2. Broker appends to log, replicates to followers.
3. Broker acknowledges producer based on `acks` setting.
4. Consumer polls broker for new records starting from its committed offset.

---

## 4. Installing & Running Kafka

### 4.1 Using Docker Compose (Recommended for Learning)
```yaml
# docker-compose.yml
version: "3.8"
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```
```bash
docker-compose up -d
```

### 4.2 Manual Installation (Binary)
```bash
# Download and extract Kafka
tar -xzf kafka_2.13-3.7.0.tgz
cd kafka_2.13-3.7.0

# Start ZooKeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka broker (in a new terminal)
bin/kafka-server-start.sh config/server.properties
```

### 4.3 KRaft Mode (No ZooKeeper — Modern Approach)
```bash
# Generate a cluster UUID
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"

# Format storage directories
bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties

# Start the broker (acts as broker + controller)
bin/kafka-server-start.sh config/kraft/server.properties
```

---

## 5. Topics, Partitions & Offsets

### 5.1 Creating a Topic
```bash
bin/kafka-topics.sh --create \
  --bootstrap-server localhost:9092 \
  --topic orders \
  --partitions 3 \
  --replication-factor 1
```

### 5.2 Listing & Describing Topics
```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092

bin/kafka-topics.sh --describe --topic orders --bootstrap-server localhost:9092
```
Sample output:
```
Topic: orders   PartitionCount: 3   ReplicationFactor: 1
    Partition: 0  Leader: 1  Replicas: 1  Isr: 1
    Partition: 1  Leader: 1  Replicas: 1  Isr: 1
    Partition: 2  Leader: 1  Replicas: 1  Isr: 1
```

### 5.3 Altering Topics
```bash
# Increase partitions (can only increase, never decrease)
bin/kafka-topics.sh --alter --topic orders --partitions 6 --bootstrap-server localhost:9092
```

### 5.4 Deleting a Topic
```bash
bin/kafka-topics.sh --delete --topic orders --bootstrap-server localhost:9092
```

### 5.5 Offsets Explained
Every record in a partition gets a monotonically increasing **offset**, starting at 0. Offsets are **local to a partition** — partition 0's offset 5 has nothing to do with partition 1's offset 5.

```
Partition 0: [0][1][2][3][4][5] <- next write here
Partition 1: [0][1][2][3]       <- next write here
```

### 5.6 How Many Partitions Should a Topic Have?
- More partitions = more parallelism (throughput) but also more overhead (open file handles, more memory on brokers, longer leader elections).
- Rule of thumb: start with `partitions = expected peak throughput / per-partition throughput`, commonly 6-12 partitions for moderate workloads.

---

## 6. CLI Tools Deep Dive

### 6.1 Console Producer
```bash
bin/kafka-console-producer.sh --topic orders --bootstrap-server localhost:9092
> {"orderId": 1, "amount": 250}
> {"orderId": 2, "amount": 99}
```

### 6.2 Console Consumer
```bash
# Read new messages only
bin/kafka-console-consumer.sh --topic orders --bootstrap-server localhost:9092

# Read from the beginning
bin/kafka-console-consumer.sh --topic orders --from-beginning --bootstrap-server localhost:9092

# With keys and formatting
bin/kafka-console-consumer.sh --topic orders --from-beginning \
  --property print.key=true --property key.separator="," \
  --bootstrap-server localhost:9092
```

### 6.3 Consumer Group Tools
```bash
# List consumer groups
bin/kafka-consumer-groups.sh --list --bootstrap-server localhost:9092

# Describe a group (shows lag!)
bin/kafka-consumer-groups.sh --describe --group order-service-group --bootstrap-server localhost:9092

# Reset offsets to earliest
bin/kafka-consumer-groups.sh --group order-service-group --topic orders \
  --reset-offsets --to-earliest --execute --bootstrap-server localhost:9092
```

### 6.4 Checking Log Segment Content (Low-Level Debugging)
```bash
bin/kafka-dump-log.sh --files /tmp/kafka-logs/orders-0/00000000000000000000.log --print-data-log
```

---

## 7. Producers

### 7.1 Producer Fundamentals
A producer sends records to a topic; Kafka decides which partition using:
1. **Explicit partition** if specified.
2. **Key hash** (`hash(key) % numPartitions`) if a key is given (ensures same key → same partition → ordering per key).
3. **Sticky/round-robin** partitioner if no key is given.

### 7.2 Java Producer Example
```java
import org.apache.kafka.clients.producer.*;
import java.util.Properties;

public class SimpleProducer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("acks", "all");
        props.put("retries", 3);

        Producer<String, String> producer = new KafkaProducer<>(props);

        for (int i = 0; i < 10; i++) {
            String key = "order-" + i;
            String value = "{\"orderId\":" + i + ", \"amount\": " + (i * 10) + "}";

            ProducerRecord<String, String> record = new ProducerRecord<>("orders", key, value);

            producer.send(record, (metadata, exception) -> {
                if (exception != null) {
                    exception.printStackTrace();
                } else {
                    System.out.printf("Sent to partition %d, offset %d%n",
                            metadata.partition(), metadata.offset());
                }
            });
        }

        producer.flush();
        producer.close();
    }
}
```

### 7.3 Python Producer Example (using `kafka-python` or `confluent-kafka`)
```python
from confluent_kafka import Producer
import json

conf = {'bootstrap.servers': 'localhost:9092'}
producer = Producer(conf)

def delivery_report(err, msg):
    if err is not None:
        print(f"Delivery failed: {err}")
    else:
        print(f"Delivered to {msg.topic()} [{msg.partition()}] at offset {msg.offset()}")

for i in range(10):
    order = {"orderId": i, "amount": i * 10}
    producer.produce(
        topic="orders",
        key=f"order-{i}",
        value=json.dumps(order),
        callback=delivery_report
    )
    producer.poll(0)  # trigger delivery callbacks

producer.flush()
```

### 7.4 Node.js Producer Example (using `kafkajs`)
```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ clientId: 'order-service', brokers: ['localhost:9092'] });
const producer = kafka.producer();

async function run() {
  await producer.connect();
  for (let i = 0; i < 10; i++) {
    await producer.send({
      topic: 'orders',
      messages: [
        { key: `order-${i}`, value: JSON.stringify({ orderId: i, amount: i * 10 }) }
      ]
    });
  }
  await producer.disconnect();
}

run().catch(console.error);
```

### 7.5 Synchronous vs Asynchronous Sends
```java
// Synchronous - blocks until ack received (safer, slower)
RecordMetadata metadata = producer.send(record).get();

// Asynchronous - fire and forget with callback (faster, default pattern)
producer.send(record, callback);
```

### 7.6 Key Producer Configs
| Config | Purpose |
|---|---|
| `acks` | `0` (no wait), `1` (leader only), `all`/`-1` (all ISR) |
| `retries` | Number of retry attempts on transient failure |
| `batch.size` | Bytes to batch per partition before sending |
| `linger.ms` | Time to wait to batch more records before sending |
| `compression.type` | `none`, `gzip`, `snappy`, `lz4`, `zstd` |
| `max.in.flight.requests.per.connection` | Concurrent unacknowledged requests (set to 1 for strict ordering with retries) |
| `enable.idempotence` | Prevents duplicate writes on retries |

---

## 8. Consumers

### 8.1 Java Consumer Example
```java
import org.apache.kafka.clients.consumer.*;
import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class SimpleConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "order-service-group");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("auto.offset.reset", "earliest");
        props.put("enable.auto.commit", "false");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("orders"));

        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("key=%s value=%s partition=%d offset=%d%n",
                            record.key(), record.value(), record.partition(), record.offset());
                }
                consumer.commitSync();  // manually commit offsets after processing
            }
        } finally {
            consumer.close();
        }
    }
}
```

### 8.2 Python Consumer Example
```python
from confluent_kafka import Consumer

conf = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'order-service-group',
    'auto.offset.reset': 'earliest',
    'enable.auto.commit': False
}

consumer = Consumer(conf)
consumer.subscribe(['orders'])

try:
    while True:
        msg = consumer.poll(1.0)
        if msg is None:
            continue
        if msg.error():
            print(f"Error: {msg.error()}")
            continue
        print(f"key={msg.key()} value={msg.value()} partition={msg.partition()} offset={msg.offset()}")
        consumer.commit(msg)  # manual commit
finally:
    consumer.close()
```

### 8.3 Node.js Consumer Example
```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ clientId: 'order-service', brokers: ['localhost:9092'] });
const consumer = kafka.consumer({ groupId: 'order-service-group' });

async function run() {
  await consumer.connect();
  await consumer.subscribe({ topic: 'orders', fromBeginning: true });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      console.log(`partition=${partition} offset=${message.offset} value=${message.value.toString()}`);
    },
  });
}

run().catch(console.error);
```

### 8.4 Auto-Commit vs Manual Commit
```java
// Auto-commit (simplest, risk of message loss/duplication)
props.put("enable.auto.commit", "true");
props.put("auto.commit.interval.ms", "5000");

// Manual commit (safer, gives you control)
consumer.commitSync();          // blocking, retries on failure
consumer.commitAsync();         // non-blocking, no retry (use callback)
```

### 8.5 Seeking to a Specific Offset
```java
TopicPartition tp = new TopicPartition("orders", 0);
consumer.assign(Collections.singletonList(tp));
consumer.seek(tp, 10);  // start reading from offset 10
```

### 8.6 `auto.offset.reset` Explained
Controls behavior when there's **no committed offset** (new consumer group) or the committed offset is invalid:
- `earliest`: start from the beginning of the partition
- `latest`: start from new messages only (default)
- `none`: throw an exception if no offset found

---

## 9. Consumer Groups & Rebalancing

### 9.1 What is a Consumer Group?
A consumer group is a set of consumers that jointly consume a topic's partitions — **each partition is assigned to exactly one consumer within the group** at a time, enabling horizontal scaling of processing.

```
Topic "orders" (3 partitions), Consumer Group "order-service-group" (3 consumers)

Partition 0 -> Consumer A
Partition 1 -> Consumer B
Partition 2 -> Consumer C
```

If you add a 4th consumer, it sits idle (more consumers than partitions = wasted resources).
If a consumer dies, its partitions are reassigned to the remaining consumers — this is called **rebalancing**.

### 9.2 Independent Consumption
Multiple consumer groups can read the **same topic independently**, each maintaining its own offsets:
```
Group "analytics-group"   -> reads all of "orders" from its own offset
Group "order-service-group" -> reads all of "orders" from its own offset
```

### 9.3 Partition Assignment Strategies
- **Range:** Assigns contiguous partition ranges per consumer (default, can be uneven).
- **RoundRobin:** Distributes partitions evenly across all consumers.
- **Sticky:** Minimizes partition movement during rebalances.
- **CooperativeSticky:** Incremental rebalancing — avoids "stop-the-world" pause (recommended in modern Kafka).

```java
props.put("partition.assignment.strategy",
    "org.apache.kafka.clients.consumer.CooperativeStickyAssignor");
```

### 9.4 Rebalancing Triggers
- A consumer joins or leaves the group
- A consumer is considered dead (missed heartbeat / `session.timeout.ms` exceeded)
- Topic partition count changes

### 9.5 Rebalance Listener Example
```java
consumer.subscribe(Collections.singletonList("orders"), new ConsumerRebalanceListener() {
    @Override
    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
        System.out.println("Partitions revoked: " + partitions);
        consumer.commitSync(); // commit before losing partitions
    }

    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        System.out.println("Partitions assigned: " + partitions);
    }
});
```

### 9.6 Static Group Membership (Avoiding Unnecessary Rebalances)
```java
props.put("group.instance.id", "consumer-instance-1"); // survives short restarts without rebalance
```

---

## 10. Serialization & Deserialization

### 10.1 Why It Matters
Kafka stores and transmits records as raw bytes. Producers **serialize** objects to bytes; consumers **deserialize** bytes back to objects.

### 10.2 Built-in Serializers
`StringSerializer`, `IntegerSerializer`, `LongSerializer`, `ByteArraySerializer`

### 10.3 Custom JSON Serializer (Java, using Jackson)
```java
public class JsonSerializer<T> implements Serializer<T> {
    private final ObjectMapper mapper = new ObjectMapper();

    @Override
    public byte[] serialize(String topic, T data) {
        try {
            return mapper.writeValueAsBytes(data);
        } catch (Exception e) {
            throw new SerializationException("Error serializing JSON", e);
        }
    }
}
```

### 10.4 Custom JSON Deserializer
```java
public class JsonDeserializer<T> implements Deserializer<T> {
    private final ObjectMapper mapper = new ObjectMapper();
    private Class<T> targetType;

    public JsonDeserializer(Class<T> targetType) {
        this.targetType = targetType;
    }

    @Override
    public T deserialize(String topic, byte[] data) {
        try {
            if (data == null) return null;
            return mapper.readValue(data, targetType);
        } catch (Exception e) {
            throw new SerializationException("Error deserializing JSON", e);
        }
    }
}
```

### 10.5 Avro Serialization (Preview — full detail in Section 11)
Avro requires a schema and is far more compact and safely evolvable than raw JSON.

---

## 11. Schema Registry & Avro

### 11.1 Why a Schema Registry?
Without enforced schemas, producers can send malformed or incompatible data, breaking downstream consumers. Confluent's **Schema Registry** stores schemas centrally and enforces **compatibility rules** on changes.

### 11.2 Defining an Avro Schema
```json
{
  "type": "record",
  "name": "Order",
  "fields": [
    { "name": "orderId", "type": "int" },
    { "name": "amount", "type": "double" },
    { "name": "customerName", "type": "string" }
  ]
}
```

### 11.3 Producing Avro Records (Java)
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://localhost:8081");

Producer<String, Order> producer = new KafkaProducer<>(props);

Order order = new Order(1, 250.0, "Alice");
producer.send(new ProducerRecord<>("orders-avro", "order-1", order));
```

### 11.4 Schema Compatibility Modes
| Mode | Meaning |
|---|---|
| `BACKWARD` | New schema can read data written with the previous schema |
| `FORWARD` | Old schema can read data written with the new schema |
| `FULL` | Both backward and forward compatible |
| `NONE` | No compatibility checks (not recommended) |

### 11.5 Registering & Checking Schemas via REST
```bash
# Register a new schema
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  --data '{"schema": "{\"type\":\"record\",\"name\":\"Order\",\"fields\":[...]}"}' \
  http://localhost:8081/subjects/orders-avro-value/versions

# Get latest schema
curl http://localhost:8081/subjects/orders-avro-value/versions/latest
```

### 11.6 Alternatives to Avro
- **Protobuf:** compact, strongly typed, widely used outside Kafka too.
- **JSON Schema:** human-readable, easier debugging, less compact than Avro.

---

## 12. Replication, Leaders & ISR

### 12.1 Why Replicate?
If a broker holding the only copy of a partition fails, that data is lost. Replication copies each partition to multiple brokers.

### 12.2 Leader & Followers
Each partition has:
- **One leader replica** — handles all reads and writes for that partition.
- **Zero or more follower replicas** — passively replicate the leader's log.

```
Topic: orders, Partition 0, Replication Factor 3
Broker 1 (Leader) --> Broker 2 (Follower) --> Broker 3 (Follower)
```

### 12.3 In-Sync Replicas (ISR)
The ISR is the set of replicas that are fully caught up with the leader within `replica.lag.time.max.ms`. Only ISR members are eligible to become leader if the current leader fails.

### 12.4 Setting Replication Factor
```bash
bin/kafka-topics.sh --create --topic orders \
  --partitions 3 --replication-factor 3 \
  --bootstrap-server localhost:9092
```

### 12.5 `min.insync.replicas`
Combined with `acks=all`, this guarantees a minimum durability level:
```properties
min.insync.replicas=2
```
If fewer than `min.insync.replicas` are in sync, producers with `acks=all` will get an error rather than silently risking data loss.

### 12.6 Unclean Leader Election
```properties
unclean.leader.election.enable=false   # default & recommended; prevents data loss
```
If `true`, an out-of-sync replica could become leader after a failure — trading durability for availability.

---

## 13. Delivery Semantics & Idempotence

### 13.1 The Three Semantics
- **At-most-once:** Messages may be lost, never redelivered (fire-and-forget, no retries).
- **At-least-once:** Messages are never lost but may be redelivered/duplicated (default with retries + manual commit after processing).
- **Exactly-once:** Every message is processed exactly one time (requires idempotent producer + transactions, see Section 14).

### 13.2 Idempotent Producer
Prevents duplicate writes from producer retries (e.g., a network blip causing a retry of an already-successful send).
```java
props.put("enable.idempotence", "true");
// automatically sets acks=all, retries=Integer.MAX_VALUE, max.in.flight.requests.per.connection<=5
```

### 13.3 How Idempotence Works Internally
Each producer gets a unique **Producer ID (PID)**; each message gets a sequence number per partition. The broker deduplicates based on `(PID, partition, sequence number)`.

### 13.4 At-Least-Once Consumer Pattern
```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
    for (ConsumerRecord<String, String> record : records) {
        process(record);  // must be idempotent on the consumer side too!
    }
    consumer.commitSync(); // commit AFTER processing
}
```

---

## 14. Kafka Transactions & Exactly-Once Semantics

### 14.1 Why Transactions?
For pipelines like **consume -> process -> produce** (common in Kafka Streams), you want the read, the processing, and the write to be atomic — either all happen, or none do.

### 14.2 Transactional Producer Example
```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("transactional.id", "order-processor-1");
props.put("enable.idempotence", "true");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

Producer<String, String> producer = new KafkaProducer<>(props);
producer.initTransactions();

try {
    producer.beginTransaction();
    producer.send(new ProducerRecord<>("processed-orders", "order-1", "processed"));
    producer.send(new ProducerRecord<>("audit-log", "order-1", "logged"));
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

### 14.3 Read-Process-Write with Consumer Offsets in the Transaction
```java
producer.beginTransaction();
try {
    producer.send(new ProducerRecord<>("processed-orders", key, value));

    Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
    offsets.put(new TopicPartition("orders", partition),
                new OffsetAndMetadata(consumedOffset + 1));

    producer.sendOffsetsToTransaction(offsets, consumerGroupId);
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

### 14.4 Consumer Isolation Level
```java
props.put("isolation.level", "read_committed"); // ignores aborted transaction records
// default is "read_uncommitted"
```

---

## 15. Log Compaction & Retention

### 15.1 Time/Size-Based Retention (Default)
Kafka deletes old segments once they exceed age or size limits — good for event streams where old data becomes irrelevant.
```properties
log.retention.hours=168        # 7 days
log.retention.bytes=1073741824 # 1 GB per partition
```

### 15.2 Log Compaction (Key-Based Retention)
Instead of deleting by age, compaction keeps **only the latest value for each key**, forever — ideal for "current state" topics (e.g., a changelog of user profile updates).
```properties
cleanup.policy=compact
```
```
Before compaction: (k1,v1) (k2,v2) (k1,v3) (k3,v4) (k1,v5)
After compaction:              (k2,v2)       (k3,v4) (k1,v5)
```

### 15.3 Combining Both Policies
```properties
cleanup.policy=compact,delete
```

### 15.4 Tombstones (Deleting a Key)
Sending a record with a `null` value for a key marks it for deletion during the next compaction cycle.
```java
producer.send(new ProducerRecord<>("user-profiles", "user-42", null)); // tombstone
```

### 15.5 Per-Topic Configuration
```bash
bin/kafka-configs.sh --alter --entity-type topics --entity-name user-profiles \
  --add-config cleanup.policy=compact --bootstrap-server localhost:9092
```

---

## 16. Kafka Connect

### 16.1 What is Kafka Connect?
A framework for reliably streaming data **between Kafka and external systems** (databases, S3, Elasticsearch, etc.) without writing custom producer/consumer code — using pre-built, configurable **connectors**.

### 16.2 Source vs Sink Connectors
- **Source Connector:** Pulls data FROM an external system INTO Kafka (e.g., MySQL → Kafka via Debezium CDC).
- **Sink Connector:** Pushes data FROM Kafka TO an external system (e.g., Kafka → Elasticsearch).

### 16.3 Standalone vs Distributed Mode
- **Standalone:** Single process, good for development/testing.
- **Distributed:** Multiple workers coordinate via Kafka itself — scalable, fault-tolerant, used in production.

### 16.4 Example: JDBC Source Connector Config
```json
{
  "name": "jdbc-source-orders",
  "config": {
    "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
    "connection.url": "jdbc:postgresql://localhost:5432/mydb",
    "connection.user": "postgres",
    "connection.password": "postgres",
    "table.whitelist": "orders",
    "mode": "incrementing",
    "incrementing.column.name": "id",
    "topic.prefix": "postgres-",
    "poll.interval.ms": 5000
  }
}
```

### 16.5 Example: Elasticsearch Sink Connector Config
```json
{
  "name": "es-sink-orders",
  "config": {
    "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
    "topics": "orders",
    "connection.url": "http://localhost:9200",
    "type.name": "_doc",
    "key.ignore": "true"
  }
}
```

### 16.6 Submitting a Connector via REST API
```bash
curl -X POST -H "Content-Type: application/json" \
  --data @jdbc-source-orders.json \
  http://localhost:8083/connectors

# Check status
curl http://localhost:8083/connectors/jdbc-source-orders/status
```

### 16.7 Debezium for Change Data Capture (CDC)
Debezium is a popular set of source connectors that stream **row-level database changes** (inserts/updates/deletes) into Kafka in near real-time — commonly used for event-driven architectures and cache invalidation.

---

## 17. Kafka Streams

### 17.1 What is Kafka Streams?
A Java library for building stream-processing applications directly on top of Kafka — no separate cluster needed (unlike Spark/Flink), it's just a library embedded in your app.

### 17.2 Core Abstractions
- **KStream:** An unbounded stream of records (like an event log).
- **KTable:** A changelog/table view — represents the *latest* value per key (like a compacted topic).
- **GlobalKTable:** A KTable replicated fully to every application instance.

### 17.3 Simple Word Count Example
```java
import org.apache.kafka.streams.*;
import org.apache.kafka.streams.kstream.*;
import org.apache.kafka.common.serialization.Serdes;
import java.util.Arrays;
import java.util.Properties;

public class WordCountApp {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "word-count-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> textLines = builder.stream("text-input");

        KTable<String, Long> wordCounts = textLines
            .flatMapValues(line -> Arrays.asList(line.toLowerCase().split("\\W+")))
            .groupBy((key, word) -> word)
            .count();

        wordCounts.toStream().to("word-count-output", Produced.with(Serdes.String(), Serdes.Long()));

        KafkaStreams streams = new KafkaStreams(builder.build(), props);
        streams.start();

        Runtime.getRuntime().addShutdownHook(new Thread(streams::close));
    }
}
```

### 17.4 Stateless Operations
```java
KStream<String, Order> orders = builder.stream("orders");

// filter
KStream<String, Order> bigOrders = orders.filter((key, order) -> order.getAmount() > 100);

// map / mapValues
KStream<String, Double> amounts = orders.mapValues(Order::getAmount);

// branch (split into multiple streams by predicate)
Map<String, KStream<String, Order>> branches = orders.split(Named.as("branch-"))
    .branch((key, order) -> order.getAmount() > 500, Named.as("large"))
    .branch((key, order) -> order.getAmount() <= 500, Named.as("small"))
    .noDefaultBranch();
```

### 17.5 Stateful Operations — Aggregations
```java
KTable<String, Double> totalPerCustomer = orders
    .groupBy((key, order) -> order.getCustomerId())
    .aggregate(
        () -> 0.0,
        (customerId, order, total) -> total + order.getAmount(),
        Materialized.with(Serdes.String(), Serdes.Double())
    );
```

### 17.6 Windowed Aggregations
```java
KTable<Windowed<String>, Long> ordersPerCustomerPerHour = orders
    .groupBy((key, order) -> order.getCustomerId())
    .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofHours(1)))
    .count();
```

### 17.7 Joins in Kafka Streams
```java
// Stream-Table join (enrich order stream with customer table)
KStream<String, EnrichedOrder> enriched = orders.join(
    customersTable,
    (order, customer) -> new EnrichedOrder(order, customer)
);

// Stream-Stream join (windowed)
KStream<String, PaymentMatch> matched = orders.join(
    payments,
    (order, payment) -> new PaymentMatch(order, payment),
    JoinWindows.ofTimeDifferenceWithNoGrace(Duration.ofMinutes(5))
);
```

### 17.8 State Stores & Fault Tolerance
Kafka Streams persists state locally (RocksDB by default) and backs it up to an internal **changelog topic** in Kafka — so state can be rebuilt automatically if an instance crashes.

---

## 18. ksqlDB

### 18.1 What is ksqlDB?
A SQL engine built on top of Kafka Streams, letting you build stream-processing pipelines using **SQL syntax** instead of Java code.

### 18.2 Creating a Stream
```sql
CREATE STREAM orders_stream (
    orderId INT,
    customerId VARCHAR,
    amount DOUBLE
) WITH (
    KAFKA_TOPIC = 'orders',
    VALUE_FORMAT = 'JSON'
);
```

### 18.3 Creating a Table
```sql
CREATE TABLE customer_totals AS
SELECT customerId, SUM(amount) AS total_spent
FROM orders_stream
GROUP BY customerId
EMIT CHANGES;
```

### 18.4 Filtering & Continuous Queries
```sql
CREATE STREAM big_orders AS
SELECT * FROM orders_stream
WHERE amount > 500
EMIT CHANGES;

-- Ad-hoc push query (keeps streaming results to your terminal)
SELECT * FROM orders_stream WHERE amount > 500 EMIT CHANGES;

-- Pull query (point-in-time lookup on a table, like a normal SQL query)
SELECT * FROM customer_totals WHERE customerId = 'cust-42';
```

### 18.5 Windowed Aggregation in ksqlDB
```sql
CREATE TABLE hourly_order_counts AS
SELECT customerId, COUNT(*) AS order_count
FROM orders_stream
WINDOW TUMBLING (SIZE 1 HOUR)
GROUP BY customerId
EMIT CHANGES;
```

---

## 19. Partitioning Strategies & Custom Partitioners

### 19.1 Default Partitioning Logic
```
If key is null       -> sticky/round-robin across partitions
If key is provided    -> partition = murmur2_hash(key) % numPartitions
If partition is set   -> use it explicitly, ignoring key
```

### 19.2 Why Keys Matter for Ordering
Kafka only guarantees ordering **within a partition** — so if you need all events for a given entity (e.g., a specific `orderId` or `userId`) processed in order, always use that entity as the key.

### 19.3 Custom Partitioner (Java)
```java
public class VipCustomerPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                          Object value, byte[] valueBytes, Cluster cluster) {
        int numPartitions = cluster.partitionCountForTopic(topic);
        String customerId = (String) key;

        if (customerId.startsWith("VIP-")) {
            return 0; // always route VIP customers to partition 0
        }
        return Math.abs(customerId.hashCode()) % (numPartitions - 1) + 1;
    }

    @Override public void close() {}
    @Override public void configure(Map<String, ?> configs) {}
}
```
```java
props.put("partitioner.class", "com.example.VipCustomerPartitioner");
```

---

## 20. Consumer/Producer Configuration Deep Dive

### 20.1 Important Producer Configs
```properties
acks=all
retries=2147483647
max.in.flight.requests.per.connection=5
enable.idempotence=true
compression.type=snappy
linger.ms=10
batch.size=32768
buffer.memory=33554432
delivery.timeout.ms=120000
```

### 20.2 Important Consumer Configs
```properties
group.id=order-service-group
enable.auto.commit=false
auto.offset.reset=earliest
max.poll.records=500
max.poll.interval.ms=300000
session.timeout.ms=45000
heartbeat.interval.ms=3000
fetch.min.bytes=1
fetch.max.wait.ms=500
```

### 20.3 Broker/Topic Configs
```properties
num.partitions=6
default.replication.factor=3
min.insync.replicas=2
log.retention.hours=168
message.max.bytes=1048588
num.network.threads=8
num.io.threads=8
```

---

## 21. Security in Kafka

### 21.1 Encryption in Transit (SSL/TLS)
```properties
security.protocol=SSL
ssl.truststore.location=/path/to/truststore.jks
ssl.truststore.password=changeit
ssl.keystore.location=/path/to/keystore.jks
ssl.keystore.password=changeit
```

### 21.2 Authentication (SASL)
```properties
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
  username="user" password="pass";
```
Other SASL mechanisms: `SCRAM-SHA-256`, `SCRAM-SHA-512`, `GSSAPI` (Kerberos), `OAUTHBEARER`.

### 21.3 Authorization (ACLs)
```bash
bin/kafka-acls.sh --bootstrap-server localhost:9092 \
  --add --allow-principal User:alice \
  --operation Read --operation Write \
  --topic orders
```

### 21.4 Encryption at Rest
Handled at the storage/disk level (e.g., LUKS, cloud provider disk encryption) — Kafka itself doesn't natively encrypt data on disk.

---

## 22. Monitoring & Metrics

### 22.1 Key Metrics to Watch
| Metric | Why It Matters |
|---|---|
| **Consumer Lag** | Gap between latest offset and consumer's committed offset — high lag = falling behind |
| **Under-replicated partitions** | Indicates replication problems |
| **Request latency** (produce/fetch) | Broker performance health |
| **Broker disk usage** | Prevent out-of-disk failures |
| **Active controller count** | Should always be exactly 1 |
| **ISR shrink/expand rate** | Frequent shrinking indicates broker instability |

### 22.2 Checking Consumer Lag via CLI
```bash
bin/kafka-consumer-groups.sh --describe --group order-service-group --bootstrap-server localhost:9092
```
```
GROUP                TOPIC    PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
order-service-group   orders   0          1000            1050            50
```

### 22.3 JMX Metrics
Kafka exposes extensive metrics via JMX, commonly scraped by **Prometheus** (via `jmx_exporter`) and visualized in **Grafana**.

### 22.4 Common Monitoring Stack
```
Kafka Brokers -> JMX Exporter -> Prometheus -> Grafana Dashboards
                                       |
                                  Alertmanager -> Slack/PagerDuty
```

---

## 23. Performance Tuning

### 23.1 Producer-Side Tuning
- Increase `batch.size` and `linger.ms` to improve throughput via better batching.
- Use `compression.type=lz4` or `zstd` for high-volume topics.
- Tune `buffer.memory` if producers are being blocked waiting for buffer space.

### 23.2 Consumer-Side Tuning
- Increase `fetch.min.bytes` and `fetch.max.wait.ms` to reduce request overhead (batch fetching).
- Increase `max.poll.records` for higher per-poll throughput, but watch out for `max.poll.interval.ms` timeouts if processing is slow.
- Scale out consumers (up to the partition count) for more parallelism.

### 23.3 Broker-Side Tuning
- Use fast disks (NVMe/SSD) — Kafka is very disk I/O sensitive.
- Tune `num.io.threads` and `num.network.threads` based on CPU cores.
- Separate OS page cache from JVM heap — keep Kafka's JVM heap modest (6-8GB) and let the OS cache handle most of the I/O buffering.

### 23.4 Reducing GC Pauses
```bash
export KAFKA_HEAP_OPTS="-Xms6g -Xmx6g"
export KAFKA_JVM_PERFORMANCE_OPTS="-XX:+UseG1GC -XX:MaxGCPauseMillis=20"
```

---

## 24. KRaft Mode (ZooKeeper-less Kafka)

### 24.1 Why KRaft?
Since Kafka 3.3+ (production-ready), Kafka replaces ZooKeeper with its own **Raft-based consensus protocol**, simplifying deployment (one less system to operate), improving scalability (millions of partitions), and speeding up metadata operations and failovers.

### 24.2 Combined vs Separate Controller Mode
```properties
# Combined mode (small clusters/dev)
process.roles=broker,controller

# Separate mode (large production clusters)
process.roles=controller   # on dedicated controller nodes
process.roles=broker       # on broker nodes
```

### 24.3 Sample KRaft `server.properties`
```properties
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
controller.listener.names=CONTROLLER
log.dirs=/tmp/kraft-combined-logs
```

---

## 25. Multi-Cluster Replication (MirrorMaker)

### 25.1 Why Replicate Across Clusters?
- Disaster recovery (DR) across data centers/regions
- Data locality for geographically distributed consumers
- Aggregating data from multiple regional clusters into a central analytics cluster

### 25.2 MirrorMaker 2 (MM2) Configuration
```properties
clusters = source, target
source.bootstrap.servers = source-broker:9092
target.bootstrap.servers = target-broker:9092

source->target.enabled = true
source->target.topics = orders.*

replication.factor = 3
sync.topic.acls.enabled = false
```

### 25.3 Running MirrorMaker 2
```bash
bin/connect-mirror-maker.sh mm2.properties
```

---

## 26. Tiered Storage

### 26.1 The Problem
Keeping years of data on local broker disks is expensive and limits retention.

### 26.2 The Solution
Tiered storage (available in newer Kafka/Confluent versions) offloads older log segments to cheaper object storage (like S3/GCS/Azure Blob) while keeping recent data on fast local disks — transparently to producers/consumers.

```properties
remote.log.storage.system.enable=true
remote.log.storage.manager.class.name=org.apache.kafka.server.log.remote.storage.RemoteLogManager
```

---

## 27. Testing Kafka Applications

### 27.1 Embedded Kafka for Unit/Integration Tests (Java, Spring)
```java
@EmbeddedKafka(partitions = 1, topics = { "orders" })
class OrderServiceTest {

    @Autowired
    private EmbeddedKafkaBroker embeddedKafkaBroker;

    @Test
    void shouldConsumeOrderMessage() {
        // produce a test message and assert consumer behavior
    }
}
```

### 27.2 Testcontainers (Real Kafka in Docker for Tests)
```java
@Testcontainers
class KafkaIntegrationTest {

    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.6.0"));

    @Test
    void testProduceConsume() {
        String bootstrapServers = kafka.getBootstrapServers();
        // build producer/consumer using bootstrapServers and run assertions
    }
}
```

### 27.3 Kafka Streams `TopologyTestDriver`
```java
TopologyTestDriver testDriver = new TopologyTestDriver(topology, props);
TestInputTopic<String, String> input = testDriver.createInputTopic(
    "text-input", Serdes.String().serializer(), Serdes.String().serializer());
TestOutputTopic<String, Long> output = testDriver.createOutputTopic(
    "word-count-output", Serdes.String().deserializer(), Serdes.Long().deserializer());

input.pipeInput("hello kafka streams");
assertEquals(1L, output.readValue());
```

---

## 28. Deploying Kafka (Docker & Kubernetes)

### 28.1 Production-Style Docker Compose (3-Broker Cluster, KRaft)
```yaml
version: "3.8"
services:
  kafka1:
    image: confluentinc/cp-kafka:7.6.0
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      KAFKA_CONTROLLER_QUORUM_VOTERS: "1@kafka1:9093,2@kafka2:9093,3@kafka3:9093"
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka1:9092
      KAFKA_CONTROLLER_LISTENER_NAMES: CONTROLLER
    ports:
      - "9092:9092"
  # kafka2, kafka3 similarly configured with different node IDs...
```

### 28.2 Kubernetes with Strimzi Operator
```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
spec:
  kafka:
    version: 3.7.0
    replicas: 3
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
    config:
      offsets.topic.replication.factor: 3
      default.replication.factor: 3
      min.insync.replicas: 2
    storage:
      type: persistent-claim
      size: 100Gi
  zookeeper:
    replicas: 3
    storage:
      type: persistent-claim
      size: 20Gi
```
```bash
kubectl apply -f kafka-cluster.yaml
```

---

## 29. Client Libraries Across Languages

| Language | Popular Libraries |
|---|---|
| Java | Official `kafka-clients`, Spring Kafka |
| Python | `confluent-kafka-python`, `kafka-python`, `aiokafka` |
| Node.js | `kafkajs`, `node-rdkafka` |
| Go | `segmentio/kafka-go`, `confluent-kafka-go` |
| .NET | `Confluent.Kafka` |
| Rust | `rdkafka` |

### 29.1 Go Producer Example
```go
package main

import (
    "fmt"
    "github.com/segmentio/kafka-go"
    "context"
)

func main() {
    writer := kafka.NewWriter(kafka.WriterConfig{
        Brokers: []string{"localhost:9092"},
        Topic:   "orders",
    })
    defer writer.Close()

    err := writer.WriteMessages(context.Background(),
        kafka.Message{
            Key:   []byte("order-1"),
            Value: []byte(`{"orderId":1,"amount":250}`),
        },
    )
    if err != nil {
        fmt.Println("write error:", err)
    }
}
```

---

## 30. Common Design Patterns & Use Cases

### 30.1 Event Sourcing
Store every state change as an immutable event in Kafka; rebuild application state by replaying events from the beginning.

### 30.2 CQRS (Command Query Responsibility Segregation)
Write model publishes events to Kafka; separate read models (materialized views) are built by consumers optimized for querying.

### 30.3 Outbox Pattern
Solves the "dual write" problem (DB write + Kafka publish must be atomic) by writing events to an "outbox" table in the same DB transaction, then using CDC (e.g., Debezium) to publish them to Kafka reliably.

### 30.4 Saga Pattern for Distributed Transactions
Coordinate a sequence of local transactions across microservices via Kafka events, with compensating events to roll back on failure.

### 30.5 Change Data Capture (CDC)
Stream database row changes into Kafka in real time (via Debezium) for cache invalidation, search index updates, or analytics.

### 30.6 Log Aggregation
Centralize logs from many services into Kafka topics, then ship to Elasticsearch/Splunk for search and analysis.

---

## 31. Error Handling & Dead Letter Queues

### 31.1 Why DLQs?
When a message repeatedly fails processing (e.g., malformed data, downstream service down), you don't want to block the whole partition — route it to a separate "dead letter" topic for later inspection/reprocessing.

### 31.2 Simple DLQ Pattern (Java)
```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
    for (ConsumerRecord<String, String> record : records) {
        try {
            process(record);
        } catch (Exception e) {
            producer.send(new ProducerRecord<>("orders-dlq", record.key(), record.value()));
        }
    }
    consumer.commitSync();
}
```

### 31.3 Retry with Backoff Before DLQ
Many frameworks (e.g., Spring Kafka's `DefaultErrorHandler`) support configurable retry attempts with exponential backoff before finally routing to a DLQ topic.

```java
DefaultErrorHandler errorHandler = new DefaultErrorHandler(
    new DeadLetterPublishingRecoverer(kafkaTemplate),
    new FixedBackOff(1000L, 3) // retry 3 times, 1 second apart
);
```

---

## 32. Kafka vs Other Messaging Systems

| Feature | Kafka | RabbitMQ | AWS SQS |
|---|---|---|---|
| Model | Distributed log | Traditional message broker/queue | Managed queue service |
| Message retention | Configurable (time/size/compaction), replayable | Deleted once consumed (unless configured otherwise) | Deleted once consumed (with visibility timeout) |
| Throughput | Very high (millions of msgs/sec) | Moderate to high | Moderate |
| Ordering | Per-partition ordering | Per-queue ordering (with caveats) | FIFO queues support ordering |
| Multiple independent consumers | Yes (via consumer groups) | Possible but less natural | Possible via SNS fan-out |
| Best for | Event streaming, high-throughput pipelines, replay-based architectures | Complex routing, RPC-style messaging | Simple decoupled queueing in AWS-native apps |

---

## 33. Best Practices & Cheat Sheet

### 33.1 Design Best Practices
- Choose keys carefully — they determine both partitioning and ordering guarantees.
- Size partitions for your target throughput and consumer parallelism upfront; growing partitions later can disrupt key-based ordering.
- Use Avro/Protobuf with a Schema Registry in any non-trivial production system — raw JSON offers no schema safety.
- Model "current state" topics with log compaction; model "event history" topics with time-based retention.

### 33.2 Reliability Best Practices
- Use `acks=all` + `min.insync.replicas=2` + `enable.idempotence=true` for durable, exactly-once-friendly writes.
- Always commit consumer offsets *after* successful processing, not before.
- Implement dead-letter queues for poison-pill messages.
- Monitor consumer lag continuously — it's the single most important health signal for a consumer application.

### 33.3 Operational Best Practices
- Run at least 3 brokers in production, with replication factor ≥ 3 for critical topics.
- Prefer KRaft mode for new clusters (Kafka 3.3+) to simplify operations.
- Separate high-throughput and low-latency workloads onto different clusters if requirements conflict.
- Automate topic/ACL provisioning via GitOps rather than manual CLI changes.

### 33.4 Quick Command Reference
| Task | Command |
|---|---|
| Create topic | `kafka-topics.sh --create ...` |
| List topics | `kafka-topics.sh --list ...` |
| Describe topic | `kafka-topics.sh --describe ...` |
| Produce (console) | `kafka-console-producer.sh ...` |
| Consume (console) | `kafka-console-consumer.sh --from-beginning ...` |
| List consumer groups | `kafka-consumer-groups.sh --list ...` |
| Check consumer lag | `kafka-consumer-groups.sh --describe --group ... ` |
| Reset offsets | `kafka-consumer-groups.sh --reset-offsets --to-earliest --execute ...` |
| View ACLs | `kafka-acls.sh --list ...` |

---

## Suggested Learning Path
1. Sections 1–6: Concepts, architecture, and hands-on CLI practice — get comfortable creating topics and producing/consuming manually.
2. Sections 7–10: Write your first real producer/consumer app in your language of choice.
3. Sections 9, 12–15: Consumer groups, replication, delivery semantics, compaction — the durability/consistency core.
4. Sections 16–18: Kafka Connect, Kafka Streams, ksqlDB — build real pipelines and stream processors.
5. Sections 19–26: Partitioning strategy, security, monitoring, tuning, KRaft, multi-cluster — production-readiness topics.
6. Sections 27–33: Testing, deployment, patterns, and best practices — tie it all together into a real system.

**Practice tip:** Spin up the Docker Compose cluster from Section 4, create a topic, and manually walk through producing, consuming with a group, killing a consumer mid-stream to watch rebalancing happen, and inspecting consumer lag — seeing these mechanics live is what makes Kafka concepts click.
