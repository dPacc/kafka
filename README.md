# Kafka

### Installation

- Go to <https://kafka.apache.org/downloads> and download the **binary**

- Next run `mkdir kafka && cd kafka`

- Enter the command: `tar -xvzf ~/Downloads/kafka.tgz --strip 1`

### Commands for Running Kafka Server

- To start the kafka server (broker): `bin/kafka-server-start.sh config/server.properties`

**NOTE**: The above command will **fail as kafka needs zookeeper** to run too.

- Start zookeeper server using: `bin/zookeeper-server-start.sh config/zookeeper.properties` and **then start the kafka server**.

- The Zookeeper Server runs on **localhost:2181** and Kafka Server (broker) runs on **localhost:9092**

### Creating and Exploring Kafka Topic

- **Create** a Topic: `bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic TOPIC_NAME`

You can check the log of creation in the `/kafka/logs/server.log` file

**What happens after creation of the topic?** If you take a look inside `ls /tmp/kafka-logs` there will be a **cities-0** log folder, inside these there are different log files which will be **initially empty** as we have not sent any messages to this topic. As soon as we start sending messages, **kafka will store new messages inside these files**.

**Why does the folder have weird name like "cities-0"?** Inside the `/config/server.properties` you will notice that the default **number of partitions is 1**. Inside of every topic, **messages can be spread among several partitions** and every partition will have its own folder **cities-0, cities-1**

- **List** Topics in a Server: `bin/kafka-topics.sh --list --bootstrap-server localhost:9092` or `bin/kafka-topics.sh --list --zookeeper localhost:2181`

- **Read** details of a Topic: `bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic cities`
