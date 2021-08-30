# Kafka

### Installation

- Go to <https://kafka.apache.org/downloads> and download the **binary**

- Next run `mkdir kafka && cd kafka`

- Enter the command: `tar -xvzf ~/Downloads/kafka.tgz --strip 1`

### Commands

- To start the Kafka server: `bin/kafka-server-start.sh config/server.properties`

The above command will **fail as kafka needs zookeeper** to run too.

- Start zookeeper using: `bin/zookeeper-server-start.sh config/zookeeper.properties` and then start the kafka server
