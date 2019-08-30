---
title: docker集群
date: 2019-05-29 17:46:07
categories: 技术贴
tags:
    - docker
    - 集群
---
## Swarm安装
* 两台机器
* 初始化集群
``` cmd
docker swarm init --advertise-addr 172.16.4.40
```
* 其他集群加入集群
``` cmd
docker swarm join --token SWMTKN-1-6cxqgpjhpifv6cz0ns5xsqmt1c9k274lbi5pzy26zrf6dp3756-9v65biz83gka55wx0bg2y44em 172.16.4.40:2377
```
*  查看tocken，后续加入节点或管理者
``` cmd
docker swarm join-token (worker|manager)
```
* 返回信息在其他机器执行
``` cmd
docker swarm join --token SWMTKN-1-6cxqgpjhpifv6cz0ns5xsqmt1c9k274lbi5pzy26zrf6dp3756-08v6bjyfptsnubg6ebfsmpr79 172.16.4.40:2377
```
* 查看集群节点
``` cmd
docker node ls
```
* 管理节点不运行副本
``` cmd
docker node update --availability drain NODE-ID
```
* 设置服务数量
``` cmd
docker service scale nginx=3
```

## docker web管理工具
* 下载protainer镜像
``` cmd
docker pull portainer/portainer
```
* 启动镜像
``` cmd
docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /data/portainer:/data portainer/portainer
```
