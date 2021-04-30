---
title: flink_spark_standalone
date: 2021-04-30 17:43:22
categories: 技术贴
tags:
    - docker
---

### 启动

* docker-compose.yml
```cmd
version: "3"
services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    restart: always
    ports:
      - 9870:9870
      - 9000:9000
    volumes:
      - hadoop_namenode:/hadoop/dfs/name
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode
    restart: always
    ports:
      - 9864:9864
    volumes:
      - hadoop_datanode:/hadoop/dfs/data
    environment:
      SERVICE_PRECONDITION: "namenode:9870"
    env_file:
      - ./hadoop.env

  spark-master:
    image: bde2020/spark-master:2.2.0-hadoop2.8-hive-java8
    container_name: spark-master
    restart: always
    ports:
      - 8080:8080
      - 7077:7077
      - 4040:4040
      - 10000:10000
      - 18080:18080
    env_file:
      - ./hadoop.env
    volumes:
      - ./hive-site.xml:/spark/conf/hive-site.xml
      - ./spark-defaults.conf:/spark/conf/spark-defaults.conf

  spark-worker:
    image: bde2020/spark-worker:2.2.0-hadoop2.8-hive-java8
    container_name: spark-worker
    restart: always
    ports:
      - 8081:8081
    env_file:
      - ./hadoop.env
    environment:
      - "SPARK_MASTER=spark://spark-master:7077"

  jobmanager:
    image: myflink
    container_name: jobmanager
    ports:
      - "8082:8081"
    command: jobmanager
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        classloader.resolve-order: parent-first
    volumes:
      - ./hive-site.xml:/opt/hive_conf/hive-site.xml
      - ./sql-client-defaults.yaml:/opt/flink/conf/sql-client-defaults.yaml

  taskmanager:
    image: myflink
    container_name: taskmanager
    depends_on:
      - jobmanager
    command: taskmanager
    environment:
      - |
        FLINK_PROPERTIES=
        jobmanager.rpc.address: jobmanager
        taskmanager.numberOfTaskSlots: 12

  hive-metastore:
    container_name: hive-metastore
    image: bde2020/hive:2.3.2
    restart: always
    env_file:
      - ./hadoop.env
    command: /opt/hive/bin/hive --service metastore
    environment:
      SERVICE_PRECONDITION: "namenode:9870 datanode:9864 postgresql:5432"
 
  postgresql:
    container_name: postgresql
    image: bde2020/hive-metastore-postgresql:2.3.0
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  hadoop_namenode:
  hadoop_datanode:
  pgdata:

```

* Dockerfile
```cmd
FROM flink:1.12-scala_2.11-java8
COPY ./sql-lib /opt/flink/sql-lib
```

* hadoop.env
```cmd
HIVE_SITE_CONF_javax_jdo_option_ConnectionURL=jdbc:postgresql://postgresql/metastore
HIVE_SITE_CONF_javax_jdo_option_ConnectionDriverName=org.postgresql.Driver
HIVE_SITE_CONF_javax_jdo_option_ConnectionUserName=hive
HIVE_SITE_CONF_javax_jdo_option_ConnectionPassword=hive
HIVE_SITE_CONF_datanucleus_autoCreateSchema=false
HIVE_SITE_CONF_hive_metastore_uris=thrift://hive-metastore:9083

CORE_CONF_fs_defaultFS=hdfs://namenode:9000
CORE_CONF_hadoop_http_staticuser_user=root
CORE_CONF_hadoop_proxyuser_hue_hosts=*
CORE_CONF_hadoop_proxyuser_hue_groups=*
CORE_CONF_io_compression_codecs=org.apache.hadoop.io.compress.SnappyCodec

HDFS_CONF_dfs_webhdfs_enabled=true
HDFS_CONF_dfs_permissions_enabled=false
HDFS_CONF_dfs_namenode_datanode_registration_ip___hostname___check=false
```

* hive-site.xml
```cmd
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://hive-metastore:9083</value>
        <description>Thrift URI for remote metastore.Used by metastore client to connect to remote metastore</description>
    </property>
</configuration>
```

* spark-defaults.conf
```cmd
spark.master                     spark://spark-master:7077
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://namenode:9000/history
```

* sql-client-defaults.yaml
```cmd
tables: []

functions: []

catalogs: 
 - name: myhive
   type: hive
   hive-conf-dir: /opt/hive_conf/
   default-database: default

execution:
  planner: blink
  type: streaming
  time-characteristic: event-time
  periodic-watermarks-interval: 200
  result-mode: table
  max-table-result-rows: 1000000
  max-parallelism: 128
  min-idle-state-retention: 0
  max-idle-state-retention: 0
  current-catalog: myhive
  current-database: default
deployment:
  response-timeout: 5000
  gateway-address: ""
  gateway-port: 0
```

### 说明

* hadoop只用hdfs作为存储不启用yarn
* flink需要新打一个镜像，将需要的jar包拷贝进去flink-connector-jdbc_2.11-1.12.0.jar、flink-shaded-hadoop-2-uber-2.8.3-10.0.jar、flink-sql-connector-hive-2.3.6_2.11-1.12.0.jar、mysql-connector-java-8.0.23.jar、ojdbc6-12.1.0.1-atlassian-hosted.jar，flink-sql链接库所需要的
* spark需要下载spark-hive版本，支持spark-sql
* hive还可以用老版本作为元数据管理
* spark提交任务

```cmd
/spark/bin/spark-submit --master spark://spark-master:7077 --class org.apache.spark.examples.SparkPi /spark/examples/jars/spark-examples_2.11-2.2.0.jar
```
* spark可以启动beeline服务来供给外部使用hivejdbc链接

```cmd
/spark/sbin/start-thriftserver.sh --master spark://spark-master:7077
```
* flink配置完spark-default和sql-client配置就可以使用flink-sql了

```cmd
/opt/flink/bin/sql-client.sh embedded -l ./sql-lib/
```
* 可以直接从hive中查询数据，提交到flink集群上面跑了
