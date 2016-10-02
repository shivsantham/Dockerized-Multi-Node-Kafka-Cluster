# Dockerized-Multi-Node-Kafka-Cluster
**Running Multi Node Kafka Cluster On Docker Containers**

Setting up a 3 node zookeeper Ensemble to form a quorum first.


**Node1 / Host1:**

docker run --name zoo-1 -e zk_id=1 -e zk_server.1=0.0.0.0:2888:3888 -e zk_server.2=10.9.38.56:2888:3888 -e zk_server.3=10.9.38.205:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 confluent/zookeeper

**Node2 / Host2:**

docker run --name zoo-2 -e zk_id=2 -e zk_server.1=10.9.37.231:2888:3888 -e zk_server.2=0.0.0.0:2888:3888 -e zk_server.3=10.9.38.205:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 confluent/zookeeper

**Node3 / Host3:**

docker run --name zoo-3 -e zk_id=3 -e zk_server.1=10.9.37.231:2888:3888 -e zk_server.2=10.9.38.56:2888:3888 -e zk_server.3=0.0.0.0:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 confluent/zookeeper

Note: Replace 0.0.0.0 for the current Zk server that you are configuring, using the Public IP wont help to form the Quorum

Starting 3 kafka brokers in each of the host. We will have to use Kafka advertised host and port to connect outside the localhost. In case, if we are running all these containers in a docker machine, then we should replace the advertised host as the IP of the docker machine.

**Node1 / Host1:**

docker run --name kafka1 -e KAFKA_BROKER_ID=1 -e   KAFKA_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -e
KAFKA_ADVERTISED_HOST_NAME=10.9.37.231 -e KAFKA_ADVERTISED_PORT=9092 -p 9092:9092 confluent/kafka

**Node2 / Host2:**

docker run --name kafka-2 -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -e KAFKA_ADVERTISED_HOST_NAME=10.9.38.56 -e KAFKA_ADVERTISED_PORT=9092 -p 9092:9092 confluent/kafka

**Node3 / Host3:**

docker run --name kafka-3 -e KAFKA_BROKER_ID=3 -e KAFKA_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -e KAFKA_ADVERTISED_HOST_NAME=10.9.38.205 -e KAFKA_ADVERTISED_PORT=9092 -p 9092:9092 confluent/kafka

Adding REST-PROXY to inspect the metadata of the cluster. We can either have one proxy to inspect the entire cluster or can be configured just like the kafka brokers here, by setting the RP_ZOOKEEPER_CONNECT attribute accordingly. We are setting RP_ZOOKEEPER_CONNECT to 10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181, as this has information about the quorum level.

**Can be done in one/more nodes:**

docker run --name rest1  -e REST_PROXY_ID=1 -e RP_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -p 8082:8082 registry.mo.sap.corp:5000/confluent/rest-proxy

If everything is setting up correctly and working fine you will see the docker containers up and running like this. docker ps command would return the following.

**CONTAINER ID IMAGE                COMMAND                  CREATED      STATUS         PORTS                                                                    NAMES**

20cdfsdfer92 confluent/rest-proxy "/usr/local/bin/rest"   3 minutes ago Up 2 minutes  0.0.0.0:8082->8082/tcp                                                  restproxy1

8b8559cdd093 confluent/kafka      "/usr/local/bin/kafka"   3 minutes ago Up 2 minutes  0.0.0.0:9092->9092/tcp                                                   kafka1

acf641770822 confluent/zookeeper  "/usr/local/bin/zk-do"   3 minutes ago Up 2 minutes  0.0.0.0:2181->2181/tcp, 0.0.0.0:2888->2888/tcp, 0.0.0.0:3888->3888/tcp   zookeeper1


**DOCKER-COMPOSE:**

In order to run the docker containers via docker-compose, please copy the docker-compose.yml file listed in node1,node2,node3 in 3 different hosts and run the command docker-compose up, this will bring the containers alive.
