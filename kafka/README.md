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

This should populate 2 new images in your docker process `core/kafka` and
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
[Kafka documentation website](https://kafka.apache.org/documentation/#quickstart_createtopic)

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

Demo 2
------

Setting up a multi-broker cluster. From [Kafka documentation website](https://kafka.apache.org/documentation/#quickstart_multibroker).

Check if cluster is up and running with `docker ps`, you will see something like
this:

```text
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS               NAMES
4a00108630c5        kafka_helper            "/bin/sh"                24 minutes ago      Up 24 minutes                           kafka_helper
0368196b7d58        core/kafka              "/init.sh --peer z..."   25 minutes ago      Up 24 minutes       9631/tcp            kafka_kafka_2
9ec89e68c40c        core/kafka              "/init.sh --peer z..."   25 minutes ago      Up 25 minutes       9631/tcp            kafka_kafka_3
83e11be5086b        core/kafka              "/init.sh --peer z..."   25 minutes ago      Up 24 minutes       9631/tcp            kafka_kafka_1
62b0939a6864        core/zookeeper          "/init.sh start co..."   25 minutes ago      Up 25 minutes       9631/tcp            kafka_zookeeper_1
```

Attach to the helper container: `docker attach kafka_helper` and run the
following commands:

```bash
# create a replicated topic
kafka-topics.sh --create --zookeeper zookeeper --replication-factor 3 --partitions 1 --topic my-replicated-topic

# check if the creation succeeded and if the replication is actually taking place
kafka-topics.sh --describe --zookeeper zookeeper --topic my-replicated-topic

# send some messages to the stream
kafka-console-producer.sh --broker-list :9092 --topic my-replicated-topic
< Hello World 1!
< Hello World 2!
^C

# check if the messages were correctly sent
kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
> Hello World 1!
> Hello World 2!
^C
```

Now we will kill the leader and check if we can still retrieve messages. When
you ran `kafka-topics.sh --describe --zookeeper zookeeper --topic my-replicated-topic`
there was a `Leader` field which indicates who is the leader of the Kafka cluster.

To isolate the broker id of your docker containers just run this

```bash
docker logs kafka_kafka_1|grep "Registered broker"
docker logs kafka_kafka_2|grep "Registered broker"
docker logs kafka_kafka_3|grep "Registered broker"
```

When you have the name of the leader container kill it with the docker CLI.
For example `docker kill kafka_kafka_2` if the leader is the container
`kafka_kafka_2`.

Go back to the `kafka_helper` container. And run the following commands:

```bash
# this should echo the messages even with the dead leader
kafka-console-consumer.sh --bootstrap-server kafka_3:9092 --from-beginning --topic my-replicated-topic
> Hello World 1!
> Hello World 2!
^C

# check which node is now the new leader
kafka-topics.sh --describe --zookeeper zookeeper --topic my-replicated-topic
```
