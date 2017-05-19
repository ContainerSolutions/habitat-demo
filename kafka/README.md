Demo for a multinode Kafka cluster
==================================

In this demo we will start a single node zookeeper, a three nodes kafka cluster
and a helper to help interact with all those nodes.

We will use docker habitat exported images for zookeeper and kafka.

Generate the docker images
--------------------------

```bash
# You need docker and habitat installed.
hab pkg export docker core/kafka
hab pkg export docker core/zookeeper
```

This should populate 2 new images in your docker process `core/kafka`,
`core/zookeeper`.

Start the cluster
-----------------

Docker compose has to be installed here.

```bash
# starts the cluster with 3 kafka nodes
docker-compose up --scale kafka=3

# attach to the helper
docker attach kafka_helper
```

Demo 1
------

Start a producer and read from a consumer. This follows the instruction from
Kafka documentation website [here](https://kafka.apache.org/documentation/#quickstart_createtopic)

Inside your helper run the following commands:
```bash
# create a test topic
kafka-topics.sh --create --zookeeper zookeeper --replication-factor 1 --partitions 1 --topic test

# check if the creation succeeded
kafka-topics.sh --list --zookeeper zookeeper

# now send a message to the test topic, the command will read from stdin and send
# on enter press. Stop the process with ^C
kafka-console-producer.sh --broker-list kafka:9092 --topic test
< Hello World!
^C

# now retrieve all the messages from kafka cluster, the process will also block,
# so break execution with ^C once messages have been read.
kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic test --from-beginning
> Hello World!
```
