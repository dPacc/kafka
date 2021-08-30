# Kafka

## Installation

- Go to <https://kafka.apache.org/downloads> and download the **binary**

- Next run `mkdir kafka && cd kafka`

- Enter the command: `tar -xvzf ~/Downloads/kafka.tgz --strip 1`

## Commands for Running Kafka Server

- To start the kafka server (broker): `bin/kafka-server-start.sh config/server.properties`

**NOTE**: The above command will **fail as kafka needs zookeeper** to run too.

- Start zookeeper server using: `bin/zookeeper-server-start.sh config/zookeeper.properties` and **then start the kafka server**.

- The Zookeeper Server runs on **localhost:2181** and Kafka Server (broker) runs on **localhost:9092**

## Creating and Exploring Kafka Topic

- **Create** a Topic: `bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic TOPIC_NAME`

You can check the log of creation in the `/kafka/logs/server.log` file

**What happens after creation of the topic?** If you take a look inside `ls /tmp/kafka-logs` there will be a **cities-0** log folder, inside these there are different log files which will be **initially empty** as we have not sent any messages to this topic. As soon as we start sending messages, **kafka will store new messages inside these files**.

**Why does the folder have weird name like "cities-0"?** Inside the `/config/server.properties` you will notice that the default **number of partitions is 1**. Inside of every topic, **messages can be spread among several partitions** and every partition will have its own folder **cities-0, cities-1**

- **List** Topics in a Server: `bin/kafka-topics.sh --list --bootstrap-server localhost:9092` or `bin/kafka-topics.sh --list --zookeeper localhost:2181`

- **Read** details of a Topic: `bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic TOPIC_NAME`

## Producing & Consuming in Kafka

Producers **send messages** to the Kafka cluster. Consumers **receive messages** from the Kafka cluster.

We can use any producer to send messages to the Topic. But let us use the producer that is shipped with the Kafka installation located in `/bin/kafka-console-producer.sh`

- **Produce** to a topic: `bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic TOPIC_NAME`

Once the above command is run, it will take you to the **console**, where you can enter the messages.

- **Consume** a topic/s: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --WHITE_LIST/TOPIC TOPIC_NAME`

**NOTE**: WHITE_LIST is for connecting to a set of topics and read from **multiple topics** using wildcard syntax. TOPIC is for reading message from a **single topic**.

The above command will open like console interface and any new messages produced will show up in the consumer.

If you had noticed, only the **latest messages seem to be consumed by the** consumer. How do we **consume all the messages from the beginning**? Use the **--from-beginning** flag

- **Consume** a topic/s: `in/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic TOPIC_NAME --from-beginning`

**IMPORTANT**: Kafka cluster **stores messages** even if they were **already consumed by one of the consumers**. Therefore, the **same messages** may be **read multiple times** by **multiple consumers**.

Producers and Consumers may appear and disappear. But Kafka doesn't care about that. It's **job is to store messages and receive or send them on demand**.

#### **Where has Kafka broker stored those messages?**

If you open the `/config/server.properties` file you will notice the `log.dirs=/tmp/kafka-logs`. Inside this folder `00000000.log` file is where the messages are stored.

**NOTE**: Kafka **doesn't store all messages forever** and after a specific amount of time (or when size of log exceeds configured max size) messages are deleted.
The **default** log retention period is **7 days (168 hours)**

**What are offsets in a topic?** Every **message** inside a topic has a **unique number** called offset. **First message in each topic has an offset 0**. Consumers start reading messages starting from specific offset.

## Topic with Multiple Partitions

Delete the `/tmp/kafka-logs` folder and restart the Zookeeper and Kafka servers.

- **Create** a topic with **3 partitions** of name **animals**: `bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --replication-factor 1 --partitions 3 --topic animals`

Now inside the `/tmp/kafka-logs/` folder you will find **3 animals folder**, one for each partition.

- Start the **producer**: `bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic animals` an produce a few messages

- Start the **consumer**: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic animals --from-beginning`

**IMPORTANT**: If you notice, the **messages in the consumer are in a different order**. This is because we have created a **topic with several partitions** and every consumers that connects to the topic will have to read from multiple partitions, it won't read in any round-robin fashion. It **can read in a different order**. A thing to keep in mind is that **consumers may not always read messages in the same order that they were produced**.

- **Consumer** from a **specific partition**: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --partition 1 --topic animals --from-beginning`

- **Consumer** from **specific offset** from **specific partition**: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --partition 2 --topic animals --offset 0`

- **Consumer** from **specific offset** from **all partitions**: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic animals --offset 1` - This will throw an error as **partition is required while specifying offset**. Remember, it is **not possible to read from a specific offset across the entire topic partitions**.
  
- Topic **details**: `bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic animals`

## Cluster with Multiple Brokers

Now that we understand the basics, let us now **run 3 different brokers** on the same machine. Start by deleting the `/tmp/kafka-logs` and `/tmp/zookeeper` folders.

**NOTE**: To run 3 different broker, you would need to create 3 different `server.properties` file with **unique port**, **broker ID** and **log directory**.

- In the `kafka/config` folder, create `server0.properties, server1.properties and server2.properties`

- Modify the **broker. id**, uncomment the **listeners://:909X** and change the **log.dirs=/tmp/kafka-logs-x**

- Start **Zookeeper**: `bin/zookeeper-server-start.sh config/zookeeper.properties`

- Start **3 different brokers**:
`bin/kafka-server-start.sh config/server0.properties`
`bin/kafka-server-start.sh config/server1.properties`
`bin/kafka-server-start.sh config/server2.properties`

How do we verify which brokers are active in the cluster? We can use **Zookeeper to get cluster info (active brokers)**.

- Get info from Zookeeper about **active broker IDs**: `bin/zookeeper-shell.sh localhost:2181 ls /brokers/ids`

- Get info from Zookeeper about **specific broker by ID**: `bin/zookeeper-shell.sh localhost:2181 get /brokers/ids/ID_NUMBER`

Let us now create a **topic** with **replication factor 1** and **partition count 5**.

- **Create** topic: `bin/kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --create --replication-factor 1 --partitions 5 --topic cars`

The **5 partitions will be randomly created on each of the brokers**. To understand better, explore the `/tmp/kafka-logs-x` folder.

Let us now start **producing** to the topic

- **Produce** to topic: `bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093,localhost:9094 --topic cars`

- **Consume** the topic: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --topic cars`

- **List** topics: `bin/kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --list`

- **Topic details**: `bin/kafka-topics.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --describe --topic cars`

- **Consume from beginning**: `bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093,localhost:9094 --topic cars --from-beginning`

Now let us **simulate a broker failure** by shutting down one of the brokers. If you **restart the consumer** now, you will notice a warning message saying that **all the messages could not be read as the partitions in the broker was not available**.

If you list the **Topic details**, you will notice that 2 leaders are none. Then, if you restart the broker, everything will be back to normal. To avoid this, we should use a **replication factor of more than 1**.
