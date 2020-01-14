---
title: Elasticsearch集群
date: 2019-11-26 18:12:37
categories: 技术贴
tags:
    - elasticsearch
    - 集群
---

### 直接贴compose文件
- 
```
version: '3'
services:
  elasticsearch_n0:
    image: elasticsearch:6.8.4
    container_name: elasticsearch_n0
    environment:
      - cluster.name=elasticsearch-cluster
      - node.name=node0
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch_n0,elasticsearch_n1,elasticsearch_n2"
      - "discovery.zen.minimum_master_nodes=2"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 9200:9200
  elasticsearch_n1:
    image: elasticsearch:6.8.4
    container_name: elasticsearch_n1
    environment:
      - cluster.name=elasticsearch-cluster
      - node.name=node1
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch_n0,elasticsearch_n1,elasticsearch_n2"
      - "discovery.zen.minimum_master_nodes=2"
    ulimits:
      memlock:
        soft: -1
        hard: -1
  elasticsearch_n2:
    image: elasticsearch:6.8.4
    container_name: elasticsearch_n2
    environment:
      - cluster.name=elasticsearch-cluster
      - node.name=node2
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch_n0,elasticsearch_n1,elasticsearch_n2"
      - "discovery.zen.minimum_master_nodes=2"
    ulimits:
      memlock:
        soft: -1
        hard: -1
```
- Linux部署注意，需要修改内存
```cmd
vi /etc/sysctl.conf
最后加上vm.max_map_count=262144
sysctl -p
```

### kafka环境搭建
- 
```
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - 2181:2181
  kafka:
    image: wurstmeister/kafka
    ports:
      - 9092:9092
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 172.17.5.96
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
  manager:
    image: sheepkiller/kafka-manager
    ports:
      - 9001:9000
    environment:
      ZK_HOSTS: zookeeper:2181
```

### zookeeper集群
- 
```
version: '3'
services:
  zoo1:
    image: zookeeper:3.4.11
    hostname: zoo1
    container_name: zookeeper_1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
  zoo2:
    image: zookeeper:3.4.11
    hostname: zoo2
    container_name: zookeeper_2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
  zoo3:
    image: zookeeper:3.4.11
    hostname: zoo3
    container_name: zookeeper_3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
```

### flink集群环境
- 
```
version: "3"
services:
  jobmanager:
    image: flink
    ports:
      - 8081:8081
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
  taskmanager:
    image: flink
    depends_on:
      - jobmanager
    command: taskmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
```