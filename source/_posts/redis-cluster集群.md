---
title: redis-cluster集群
date: 2021-06-02 17:34:54
categories: 运维
tags:
    - redis集群
---

### 编写 Redis 配置文件

* 执行以下操作
```cmd
vim redis-cluster.tmpl
```
* 编写内容
```cmd
port ${PORT}
requirepass 1234
masterauth 1234
protected-mode no
daemonize no
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-announce-ip 192.168.105.228
cluster-announce-port ${PORT}
cluster-announce-bus-port 1${PORT}
```
* 说明
        port：节点端口
        requirepass：添加访问认证
        masterauth：如果主节点开启了访问认证，从节点访问主节点需要认证
        protected-mode：保护模式，默认值 yes，即开启。开启保护模式以后，需配置 bind ip 或者设置访问密码，关闭保护模式，外部网络可以直接访问
        daemonize：是否以守护线程的方式启动（后台启动），默认 no
        appendonly：是否开启 AOF 持久化模式，默认 no
        cluster-enabled：是否开启集群模式，默认 no
        cluster-config-file：集群节点信息文件
        cluster-node-timeout：集群节点连接超时时间
        cluster-announce-ip：集群节点 IP，填写宿主机的 IP
        cluster-announce-port：集群节点映射端口
        cluster-announce-bus-port：集群节点总线端口

        每个 Redis 集群节点都需要打开两个 TCP 连接。一个用于为客户端提供服务的正常 Redis TCP 端口，例如 6379。还有一个基于 6379 端口加 10000 的端口，比如 16379。
        第二个端口用于集群总线，这是一个使用二进制协议的节点到节点通信通道。节点使用集群总线进行故障检测、配置更新、故障转移授权等等。客户端永远不要尝试与集群总线端口通信，与正常的 Redis 命令端口通信即可，但是请确保防火墙中的这两个端口都已经打开，否则 Redis 集群节点将无法通信。
* 在机器上执行shell脚本
```cmd
for port in `seq 6371 6376`; do \
  mkdir -p ${port}/conf \
  && PORT=${port} envsubst < redis-cluster.tmpl > ${port}/conf/redis.conf \
  && mkdir -p ${port}/data;\
done
```
* 编写compose文件
```cmd
version: "3.8"
services:
  redis-6371:
    image: redis 
    container_name: redis-6371 
    ports:
      - 6371:6371
      - 16371:16371
    volumes: 
      - ./6371/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6371/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf 
  redis-6372:
    image: redis
    container_name: redis-6372
    ports:
      - 6372:6372
      - 16372:16372
    volumes:
      - ./6372/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6372/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
  redis-6373:
    image: redis
    container_name: redis-6373
    ports:
      - 6373:6373
      - 16373:16373
    volumes:
      - ./6373/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6373/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
  redis-6374:
    image: redis
    container_name: redis-6374
    ports:
      - 6374:6374
      - 16374:16374
    volumes:
      - ./6374/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6374/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
  redis-6375:
    image: redis
    container_name: redis-6375
    ports:
      - 6375:6375
      - 16375:16375
    volumes:
      - ./6375/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6375/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
  redis-6376:
    image: redis
    container_name: redis-6376
    ports:
      - 6376:6376
      - 16376:16376
    volumes:
      - ./6376/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - ./6376/data:/data
    command: redis-server /usr/local/etc/redis/redis.conf
```
* 进入容器创建集群
```cmd
redis-cli -a 1234 --cluster create 192.168.105.228:6371 192.168.105.228:6372 192.168.105.228:6373 192.168.105.228:6374 192.168.105.228:6375 192.168.105.228:6376 --cluster-replicas 1
```
* 常用命令
```cmd
# 连接至集群某个节点
redis-cli -c -a 1234 -h 192.168.105.228 -p 6375
# 查看集群信息
cluster info
# 查看集群结点信息
cluster nodes
```

### 集群说明
* redis cluster的hash slot算法
redis cluster有固定的16384个hash slot，对每个key计算CRC16值，然后对16384取模，可以获取key对应的hash slot
redis cluster中每个master都会持有部分slot，比如有3个master，那么可能每个master持有5000多个hash slot
hash slot让node的增加和移除很简单，增加一个master，就将其他master的hash slot移动部分过去，减少一个master，就将它的hash slot移动到其他master上去移动hash slot的成本是非常低的
客户端的api，可以对指定的数据，让他们走同一个hash slot，通过hash tag来实现
127.0.0.1:7000>CLUSTER ADDSLOTS 0 1 2 3 4 ... 5000  可以将槽0-5000指派给节点7000负责。
每个节点都会记录哪些槽指派给了自己，哪些槽指派给了其他节点。
客户端向节点发送键命令，节点要计算这个键属于哪个槽。
如果是自己负责这个槽，那么直接执行命令，如果不是，向客户端返回一个MOVED错误，指引客户端转向正确的节点。
* redis cluster的多master的写入
在redis cluster写入数据的时候，其实是你可以将请求发送到任意一个master上去执行但是，每个master都会计算这个key对应的CRC16值，然后对16384个hashslot取模，找到key对应的hashslot，找到hashslot对应的master如果对应的master就在自己本地的话，set mykey1 v1，mykey1这个key对应的hashslot就在自己本地，那么自己就处理掉了但是如果计算出来的hashslot在其他master上，那么就会给客户端返回一个moved error，告诉你，你得到哪个master上去执行这条写入的命令
什么叫做多master的写入，就是每条数据只能存在于一个master上，不同的master负责存储不同的数据，分布式的数据存储100w条数据，5个master，每个master就负责存储20w条数据，分布式数据存储
默认情况下，redis cluster的核心的理念，主要是用slave做高可用的，每个master挂一两个slave，主要是做数据的热备，还有master故障时的主备切换，实现高可用的redis cluster默认是不支持slave节点读或者写的，跟我们手动基于replication搭建的主从架构不一样的
jedis客户端，对redis cluster的读写分离支持不太好的默认的话就是读和写都到master上去执行的如果你要让最流行的jedis做redis cluster的读写分离的访问，那可能还得自己修改一点jedis的源码，成本比较高
读写分离，是为了什么，主要是因为要建立一主多从的架构，才能横向任意扩展slave node去支撑更大的读吞吐量redis cluster的架构下，实际上本身master就是可以任意扩展的，你如果要支撑更大的读吞吐量，或者写吞吐量，或者数据量，都可以直接对master进行横向扩展就可以了