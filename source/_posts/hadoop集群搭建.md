---
title: hadoop集群搭建
date: 2021-04-30 16:53:25
categories: 技术贴
tags:
    - docker
    - hadoop集群
---

### 启动

* docker-compose.yml
```cmd
version: "3"
services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop2.7.4-java8
    container_name: namenode
    restart: always
    ports:
      - 50070:50070
    volumes:
      - hadoop_namenode:/hadoop/dfs/name
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop2.7.4-java8
    container_name: datanode
    restart: always
    ports:
      - 50075:50075
    volumes:
      - hadoop_datanode:/hadoop/dfs/data
    environment:
      SERVICE_PRECONDITION: "namenode:50070"
    env_file:
      - ./hadoop.env

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop2.7.4-java8
    container_name: resourcemanager
    restart: always
    ports:
      - 8088:8088
    environment:
      SERVICE_PRECONDITION: "namenode:50070 datanode:50075"
    env_file:
      - ./hadoop.env

  nodemanager:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop2.7.4-java8
    container_name: nodemanager
    restart: always
    ports:
      - 8042:8042
    environment:
      SERVICE_PRECONDITION: "namenode:50070 datanode:50075 resourcemanager:8088"
    env_file:
      - ./hadoop.env

  historyserver:
    image: bde2020/hadoop-historyserver:2.0.0-hadoop2.7.4-java8
    container_name: historyserver
    restart: always
    ports:
      - 8188:8188
    environment:
      SERVICE_PRECONDITION: "namenode:50070 datanode:50075 resourcemanager:8088"
    env_file:
      - ./hadoop.env

  hive-server:
    container_name: hive-server
    image: bde2020/hive:2.3.2
    restart: always
    env_file:
      - ./hadoop.env
    environment:
      SERVICE_PRECONDITION: "hive-metastore:9083"
    ports:
      - 10000:10000

  spark-master:
    image: bde2020/spark-master:2.2.0-hadoop2.8-hive-java8
    container_name: spark-master
    restart: always
    ports:
      - 8080:8080
      - 7077:7077
      - 4040:4040
    env_file:
      - ./hadoop.env
    volumes:
      - ./hive-site.xml:/spark/conf/hive-site.xml

  hive-metastore:
    container_name: hive-metastore
    image: bde2020/hive:2.3.2
    restart: always
    env_file:
      - ./hadoop.env
    command: /opt/hive/bin/hive --service metastore
    environment:
      SERVICE_PRECONDITION: "namenode:50070 datanode:50075 postgresql:5432"
    ports:
      - 9083:9083
 
  postgresql:
    container_name: postgresql
    image: bde2020/hive-metastore-postgresql:2.3.0
    ports:
      - 5432:5432
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  hadoop_namenode:
  hadoop_datanode:
  pgdata:
```

* hadoop.env
```cmd
HIVE_SITE_CONF_javax_jdo_option_ConnectionURL=jdbc:postgresql://postgresql/metastore
HIVE_SITE_CONF_javax_jdo_option_ConnectionDriverName=org.postgresql.Driver
HIVE_SITE_CONF_javax_jdo_option_ConnectionUserName=hive
HIVE_SITE_CONF_javax_jdo_option_ConnectionPassword=hive
HIVE_SITE_CONF_datanucleus_autoCreateSchema=false
HIVE_SITE_CONF_hive_metastore_uris=thrift://hive-metastore:9083

CORE_CONF_fs_defaultFS=hdfs://namenode:8020
CORE_CONF_hadoop_http_staticuser_user=root
CORE_CONF_hadoop_proxyuser_hue_hosts=*
CORE_CONF_hadoop_proxyuser_hue_groups=*
CORE_CONF_io_compression_codecs=org.apache.hadoop.io.compress.SnappyCodec

HDFS_CONF_dfs_webhdfs_enabled=true
HDFS_CONF_dfs_permissions_enabled=false
HDFS_CONF_dfs_namenode_datanode_registration_ip___hostname___check=false

YARN_CONF_yarn_log___aggregation___enable=true
YARN_CONF_yarn_log_server_url=http://historyserver:8188/applicationhistory/logs/
YARN_CONF_yarn_resourcemanager_recovery_enabled=true
YARN_CONF_yarn_resourcemanager_store_class=org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore
YARN_CONF_yarn_resourcemanager_scheduler_class=org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler
YARN_CONF_yarn_scheduler_capacity_root_default_maximum___allocation___mb=8192
YARN_CONF_yarn_scheduler_capacity_root_default_maximum___allocation___vcores=4
YARN_CONF_yarn_resourcemanager_fs_state___store_uri=/rmstate
YARN_CONF_yarn_resourcemanager_system___metrics___publisher_enabled=true
YARN_CONF_yarn_resourcemanager_hostname=resourcemanager
YARN_CONF_yarn_resourcemanager_address=resourcemanager:8032
YARN_CONF_yarn_resourcemanager_scheduler_address=resourcemanager:8030
YARN_CONF_yarn_resourcemanager_resource__tracker_address=resourcemanager:8031
YARN_CONF_yarn_timeline___service_enabled=true
YARN_CONF_yarn_timeline___service_generic___application___history_enabled=true
YARN_CONF_yarn_timeline___service_hostname=historyserver
YARN_CONF_mapreduce_map_output_compress=true
YARN_CONF_mapred_map_output_compress_codec=org.apache.hadoop.io.compress.SnappyCodec
YARN_CONF_yarn_nodemanager_resource_memory___mb=16384
YARN_CONF_yarn_nodemanager_resource_cpu___vcores=8
YARN_CONF_yarn_nodemanager_disk___health___checker_max___disk___utilization___per___disk___percentage=98.5
YARN_CONF_yarn_nodemanager_remote___app___log___dir=/app-logs
YARN_CONF_yarn_nodemanager_aux___services=mapreduce_shuffle
YARN_CONF_yarn_nodemanager_pmem___check___enabled=false
YARN_CONF_yarn_nodemanager_vmem___check___enabled=false
#yarn.nodemanager.pmem-check-enabled：
#是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默#认是true
#yarn.nodemanager.vmem-check-enabled：
#是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默#认是true

MAPRED_CONF_mapreduce_framework_name=yarn
MAPRED_CONF_mapred_child_java_opts=-Xmx4096m
MAPRED_CONF_mapreduce_map_memory_mb=4096
MAPRED_CONF_mapreduce_reduce_memory_mb=8192
MAPRED_CONF_mapreduce_map_java_opts=-Xmx3072m
MAPRED_CONF_mapreduce_reduce_java_opts=-Xmx6144m
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

### 说明

* hadoop集群搭建版本兼容很重要，这里的hadoop、spark、hive的版本是一样的
* hive-metastore作为元数据服务，对外共给客户端使用hiveserver、sparkclient、flinkclient等
* postgresql是存储元数据的表、真正的数据在hdfs里面存着
* hive-server是hive客户端，暴露10000端口，可以通过hivejdbc共给java调用，使用分区、简单查询等
```cmd
docker-compose exec hive-server bash
/opt/hive/bin/beeline -u jdbc:hive2://localhost:10000
> CREATE TABLE pokes (foo INT, bar STRING);
> LOAD DATA LOCAL INPATH '/opt/hive/examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
```
* 由于配置了yarn环境，hivesql会直接提交到yarn上面进行map->reduce操作,mr已经被弃用了，使用spark等进行内存运算会更快
* resourcemanager、nodemanager是yarn的资源管理服务，实际任务是跑在nodemanager上面的，所以它于datanode放在一台机器最好，本地读取文件速度快
```cmd
$HADOOP_PREFIX/bin/hdfs --config $HADOOP_CONF_DIR datanode
$HADOOP_PREFIX/bin/yarn --config $HADOOP_CONF_DIR nodemanager
```
* spark-master在这里只作为客户端，spark on yarn不需要启动spark集群，只需要配置yarn路径即可，然后任务提交到yarn，yarn会自己创建集群，分为客户端与集群两种方式提交
```cmd
/spark/bin/spark-submit --master yarn --deploy-mode cluster --class org.apache.spark.examples.SparkPi /spark/examples/jars/spark-examples_2.11-2.2.0.jar
/spark/bin/spark-submit --master yarn --deploy-mode client --class org.apache.spark.examples.SparkPi /spark/examples/jars/spark-examples_2.11-2.2.0.jar
```
* 可以通过yarn命令查看日志等
```cmd
yarn application -list 
yarn logs -applicationId xxx
```
