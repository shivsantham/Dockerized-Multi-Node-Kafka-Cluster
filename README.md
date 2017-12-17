**Running Multi Node Kafka Cluster On Docker Containers**
                                  
                                   The Zookeeper Ensemble 

Kafka is dependent on zookeeper, for the following. So in-order to setup a cluster, we need to first configure the zookeeper quorum.

A) Electing a controller. The controller is one of the brokers and is responsible for maintaining the leader/follower relationship for all the partitions. When a node shuts down, it is the controller that tells other replicas to become partition leaders to replace the partition leaders on the node that is going away. Zookeeper is used to elect a controller, make sure there is only one and elect a new one it if it crashes.

B) Cluster membership - which brokers are alive and part of the cluster? this is also managed through ZooKeeper.
Topic configuration - which topics exist, how many partitions each has, where are the replicas, who is the preferred leader, what configuration overrides are set for each topic

C) Quotas - how much data is each client allowed to read and write

D) ACLs - who is allowed to read and write to which topic
(old high level consumer) - Which consumer groups exist, who are their members and what is the latest offset each group got from each partition.

Setting up a 3 node zookeeper Ensemble to form a quorum(A replicated group of servers ) first.
The following will launch 3 docker containers across 3 hosts. 

**Node1 / Host1:**

docker run --name zoo-1 -e zk_id=1 -e zk_server.1=0.0.0.0:2888:3888 -e zk_server.2=10.9.38.56:2888:3888 -e zk_server.3=10.9.38.205:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 confluent/zookeeper

**Node2 / Host2:**

docker run --name zoo-2 -e zk_id=2 -e zk_server.1=10.9.37.231:2888:3888 -e zk_server.2=0.0.0.0:2888:3888 -e zk_server.3=10.9.38.205:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 confluent/zookeeper

**Node3 / Host3:**

docker run --name zoo-3 -e zk_id=3 -e zk_server.1=10.9.37.231:2888:3888 -e zk_server.2=10.9.38.56:2888:3888 -e zk_server.3=0.0.0.0:2888:3888 -p 2181:2181 -p 2888:2888 -p 3888:3888 confluent/zookeeper

**Note:** 

• The IP address mentioned here is the IP of the host where the docker containers are running.

• Replace 0.0.0.0 for the current Zk server that you are configuring, using the Public IP wont help to form the Quorum.

• The two port numbers after each server name: " 2888" and "3888". Peers use the former port to connect to other peers. Such a connection is necessary so that peers can communicate, for example, to agree upon the order of updates. More specifically, a ZooKeeper server uses this port to connect followers to the leader. When a new leader arises, a follower opens a TCP connection to the leader using this port. Because the default leader election also uses TCP, we currently require another port for leader election.

                                         Setting up Kafka Nodes


Starting 3 kafka brokers in each of the host. We will have to use Kafka advertised host and port to connect outside the localhost. In case, if we are running all these containers in a docker machine, then we should replace the advertised host as the IP of the docker machine.

**Node1 / Host1:**

docker run --name kafka1 -e KAFKA_BROKER_ID=1 -e   KAFKA_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -e KAFKA_ADVERTISED_HOST_NAME=10.9.37.231 -e KAFKA_ADVERTISED_PORT=9092 -p 9092:9092 confluent/kafka

**Node2 / Host2:**

docker run --name kafka-2 -e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -e KAFKA_ADVERTISED_HOST_NAME=10.9.38.56 -e KAFKA_ADVERTISED_PORT=9092 -p 9092:9092 confluent/kafka

**Node3 / Host3:**

docker run --name kafka-3 -e KAFKA_BROKER_ID=3 -e KAFKA_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -e KAFKA_ADVERTISED_HOST_NAME=10.9.38.205 -e KAFKA_ADVERTISED_PORT=9092 -p 9092:9092 confluent/kafka

Adding REST-PROXY to inspect the metadata of the cluster. We can either have one proxy to inspect the entire cluster or can be configured just like the kafka brokers here, by setting the RP_ZOOKEEPER_CONNECT attribute accordingly.

We are setting RP_ZOOKEEPER_CONNECT to 10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181, as this has information about the quorum level.

**Can be done in one/more nodes:**

docker run --name rest1  -e REST_PROXY_ID=1 -e RP_ZOOKEEPER_CONNECT=10.9.37.231:2181,10.9.38.56:2181,10.9.38.205:2181 -p 8082:8082 confluent/rest-proxy

NOTE: REST_PROXY has no mechanism like quorum. So need a load balancer in place when there are multiple REST proxies.

If everything is setting up correctly and working fine you will see the docker containers up and running like this. docker ps command would return the following.

**CONTAINER ID IMAGE                COMMAND                  CREATED      STATUS         PORTS                                                                    NAMES**

20cdfsdfer92 confluent/rest-proxy "/usr/local/bin/rest"   3 minutes ago Up 2 minutes  0.0.0.0:8082->8082/tcp                                                  restproxy1

8b8559cdd093 confluent/kafka      "/usr/local/bin/kafka"   3 minutes ago Up 2 minutes  0.0.0.0:9092->9092/tcp                                                   kafka1

acf641770822 confluent/zookeeper  "/usr/local/bin/zk-do"   3 minutes ago Up 2 minutes  0.0.0.0:2181->2181/tcp, 0.0.0.0:2888->2888/tcp, 0.0.0.0:3888->3888/tcp   zookeeper1


**DOCKER-COMPOSE:**

In order to run the docker containers via docker-compose, please copy the docker-compose.yml file listed in node1,node2,node3 in 3 different hosts and run the command docker-compose up, this will bring the containers alive.
