---
title: mongo集群分片
date: 2019-07-04 13:39:09
categories: 技术贴
tags:
    - mongo
    - 集群
    - 分片
---

### 介绍

* 主要分为三部分，配置服务副本集、分片副本集、连接 mongos 到分片集群，增加分片到集群

### 编写dockerFile

* --configsvr的作用是将端口改为27019
* --shardsvr的作用是将端口改为27018

``` cmd
version: '3'
services:
  conf1:
    image: mongo:3.4
    container_name: conf1
    ports:
      - 27017:27019
    command: mongod --configsvr --dbpath /data/db --replSet conf
  conf2:
    image: mongo:3.4
    container_name: conf2
    ports:
      - 27018:27019
    command: mongod --configsvr --dbpath /data/db --replSet conf
  conf3:
    image: mongo:3.4
    container_name: conf3
    ports:
      - 27019:27019
    command: mongod --configsvr --dbpath /data/db --replSet conf
  sh1:
    image: mongo:3.4
    container_name: sh1
    ports:
      - 27027:27018
    command: mongod --shardsvr --dbpath /data/db --replSet sh1
  sh2:
    image: mongo:3.4
    container_name: sh2
    ports:
      - 27028:27018
    command: mongod --shardsvr --dbpath /data/db --replSet sh1
  sh3:
    image: mongo:3.4
    container_name: sh3
    ports:
      - 27029:27018
    command: mongod --shardsvr --dbpath /data/db --replSet sh1
  sh4:
    image: mongo:3.4
    container_name: sh4
    ports:
      - 27037:27018
    command: mongod --shardsvr --dbpath /data/db --replSet sh2
  sh5:
    image: mongo:3.4
    container_name: sh5
    ports:
      - 27038:27018
    command: mongod --shardsvr --dbpath /data/db --replSet sh2
  sh6:
    image: mongo:3.4
    container_name: sh6
    ports:
      - 27039:27018
    command: mongod --shardsvr --dbpath /data/db --replSet sh2
  mongos:
    image: mongo:3.4
    container_name: mongos
    ports:
      - 21017:27017
    command: mongos --configdb conf/172.16.11.130:27017,172.16.11.130:27018,172.16.11.130:27019
```

* 启动docker容器

### 配置服务副本集

``` cmd
docker-compose exec conf1 mongo --port=27019
config = {
      "_id" : "conf",
      "members" : [
          {
              "_id" : 0,
              "host" : "172.16.11.130:27017"
          },
          {
              "_id" : 1,
              "host" : "172.16.11.130:27018"
          },
          {
              "_id" : 2,
              "host" : "172.16.11.130:27019"
          }
      ]
}
rs.initiate(config)
```

### 配置分片1、2副本集

``` cmd
docker-compose exec sh1 mongo --port=27018
config = {
      "_id" : "sh1",
      "members" : [
          {
              "_id" : 0,
              "host" : "172.16.11.130:27027"
          },
          {
              "_id" : 1,
              "host" : "172.16.11.130:27028"
          },
          {
              "_id" : 2,
              "host" : "172.16.11.130:27029"
          }
      ]
}
rs.initiate(config)
docker-compose exec sh4 mongo --port=27018
config = {
      "_id" : "sh2",
      "members" : [
          {
              "_id" : 0,
              "host" : "172.16.11.130:27037"
          },
          {
              "_id" : 1,
              "host" : "172.16.11.130:27038"
          },
          {
              "_id" : 2,
              "host" : "172.16.11.130:27039"
          }
      ]
}
rs.initiate(config)
```

### 配置mongos添加分片

``` cmd
docker-compose exec mongos mongo
sh.addShard("sh1/172.16.11.130:27027,172.16.11.130:27028,172.16.11.130:27029")
sh.addShard("sh2/172.16.11.130:27037,172.16.11.130:27038,172.16.11.130:27039")
```

### 数据库、集合启用分片

``` cmd
sh.enableSharding("test")
sh.shardCollection("test.test", {"_id": "hashed" })
```

### python连接mongos测试

``` python
import pymongo

myclient = pymongo.MongoClient("mongodb://172.16.11.130:21017")
mydb = myclient['test']
mycol = mydb['test']
for i in range(1000):
    mycol.insert_one({'name': "李旭", 'sex': '男'})
```

### 验证故障转移

``` cmd
docker stop sh1
docker stop sh4
```

* 将两台主机关机，然后查询