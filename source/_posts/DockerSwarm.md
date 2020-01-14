---
title: DockerSwarm
date: 2019-10-11 20:51:21
categories: 技术贴
tags:
    - docker
    - swarm
    - 集群
---

### Swarm常用命令

- docker service -h

- swarm stack 启动脚本
```cmd
#!/bin/sh
service_name='redis'
echo "start service..."
docker stack deploy -c docker-compose.yaml $service_name
echo "done!!!"
```

### NFS挂载共享目录

- 查看是否安装
```cmd
rpm -qa nfs-utils rpcbind
```
- 安装
```cmd
sudo yum install -y nfs-utils rpcbind
```
- 编辑共享目录
```cmd
vi /etc/exports
/data/work/share 172.16.11.0/24(rw,sync,no_root_squash)
or
echo "/nfs4k8s *(rw,async,no_root_squash)" >> /etc/exports
```
- 配置生效
```cmd
exportfs -rv
```
- 启动服务
```cmd
sudo systemctl enable rpcbind
sudo systemctl start rpcbind
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
```
- compose文件配置
```cmd
version: "3"
services:
  nginx:
    image: nginx:alpine
    ports:
      - 80:80
    volumes:
      - myvolume:/usr/share/nginx/html
volumes:
  myvolume:
    driver_opts:
      type: "nfs"
      o: "addr=172.16.11.130,rw"
      device: ":/data/work/share"
```
