---
title: mongo集群
date: 2019-06-21 15:07:37
categories: 技术贴
tags:
    - mongo
    - 集群
---

### 一主两从

* 可进行读写分离
* 具备故障转移能力

``` cmd
version: '3'
services:
  rs1:
    image: mongo:3.4
    ports:
      - 27017:27017
    command: mongod --dbpath /data/db --replSet myset
  rs2:
    image: mongo:3.4
    ports:
      - 27018:27017
    command: mongod --dbpath /data/db --replSet myset
  rs3:
    image: mongo:3.4
    ports:
      - 27019:27017
    command: mongod --dbpath /data/db --replSet myset
```

### 运行完docker-compose up之后执行一下操作初始化

``` cmd
docker-compose exec rs1 mongo
config = {
      "_id" : "myset",
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

* 查看配置与副本级状态

``` cmd
rs.conf() 
rs.status()
```

* 插入信息到主节点

``` cmd
docker-compose exec rs1 mongo
use test
db.test.insert({msg: 'this is from primary', ts: new Date()})
```

* 在副本集中检测信息是否同步

``` cmd
docker-compose exec rs2 mongo
rs.slaveOk()
use test
db.test.find()
docker-compose exec rs3 mongo
rs.slaveOk()
use test
db.test.find()
```

* 故障测试

``` cmd
docker-compose stop rs1
docker-compose exec rs2 mongo
docker-compose exec rs3 mongo
```

* 使用副本集群

``` python
import pymongo
myclient = pymongo.MongoClient("mongodb://172.16.11.130:27017,172.16.11.130:27018,172.16.11.130:27019")
mydb = myclient['test']
mycol = mydb['test']
mycol.insert_one({'name': '李旭', 'sex': '男'})
```
