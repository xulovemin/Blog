---
title: docker哨兵
date: 2019-06-27 16:42:13
categories: 技术贴
tags:
    - redis
    - 集群
---
### 主从复制

* 开启3个Redis服务
``` cmd
docker run --name redis-6379 -p 6379:6379 -d redis
docker run --name redis-6380 -p 6380:6379 -d redis
docker run --name redis-6381 -p 6381:6379 -d redis
```

* 分别查看主机ip
``` cmd
docker inspect containerid
```

* 进入docker容器内部，分配主从
``` cmd
docker exec -it containerid redis-cli
slaveof 主ip 6379
```

* 查看主从角色
``` cmd
info replication
```

### Sentinel哨兵

* 新建sentinel.conf文件
```cmd
port 26379
sentinel deny-scripts-reconfig yes
# Redis监控mymaster，投票达到1则表示master挂掉了
sentinel monitor mymaster 172.17.0.4 6379 1
# 10秒内mymaster还没活过来，则认为master宕机了
sentinel failover-timeout mymaster 10000
```

* 启动哨兵服务
```cmd
docker run --name redis-26379 -p 26379:26379 -v /data/work/redistest/sentinel.conf:/usr/local/etc/redis/sentinel.conf -d redis redis-sentinel /usr/local/etc/redis/sentinel.conf
```

* 查看哨兵节点日志
* 进入哨兵节点，查看是否有主节点
```cmd
docker exec -it containerid bash
redis-cli -p 26379
sentinel master mymaster
```

* 哨兵节点也可以集群，这里只搭建一个哨兵节点

* docker暂停主节点，过一会自动将从节点变为主节点

### python访问哨兵集群

``` python
from redis.sentinel import Sentinel

sentinel = Sentinel([('172.16.11.130', 26379)])
master = sentinel.discover_master('mymaster')
print(master)
slave = sentinel.discover_slaves('mymaster')
print(slave)

master = sentinel.master_for('mymaster')
master.set('foo', 'bar')
slave = sentinel.slave_for('mymaster')
r_ret = slave.get('foo')
print(r_ret)
```