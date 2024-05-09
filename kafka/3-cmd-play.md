# Using apache kafka
* go to kafka folder and open a terminal from the kafka folder
* start the zookeeper followed by the properties file
```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```
* open another terminal form the kafka folder and start the kafka server followed by the properties file
```bash
bin/kafka-server-start.sh config/server.properties
```

## Default ports
* zookeeper: 2181
* kafka: 9092

## Create a topic
* open another terminal from root kafka
* run the command followed by the kafka server address and create topic and partition count and replication factor args. The default partition count is 1 if we don't define it
```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic test-topic1 --partitions 3 --replication-factor 1
```

## useful command: list of topics
```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

## useful command: full description of a topic
```bash
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic test-topic1
```

# Producers and consumers
* Open Kafka offset exploere. Give a cluster name and zookeeper host/port etc and create a connection.

## Run Producer

* run the console-producer command in another terminal with all the broker server list(all the avilable servers) followed by the topic name.

```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic1
```

* after running the command we need to put the message we want to deliver.
* run the console-consumer command in another terminal. define the boker server list and which topic to listen except from where we want to listen.

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic1 --from-beginning
```

# Send a bulk message using csv file
```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic1 < d/test.csv
```

# using confluent community edition
All are the same only paths are different

# using docker
* run the docker exec -it command into the container
* find the kafka installation path
* execute the above commands