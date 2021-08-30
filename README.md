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

- **Produce** to a topic: `in/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic TOPIC_NAME`

Once the above command is run, it will take you to the **console**, where you can enter the messages.

- **Consume** a topic/s: `in/kafka-console-consumer.sh --bootstrap-server localhost:9092 --WHITE_LIST/TOPIC TOPIC_NAME`

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
