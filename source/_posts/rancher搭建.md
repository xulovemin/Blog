---
title: rancher搭建
date: 2019-10-25 09:06:23
categories: 技术帖
tags:
    - docker编排
    - 集群搭建
---

### 环境准备
- 多台机器

### 安装Rancher
- Rancer2.X在一台机器上执行
```cmd
sudo docker run -d --restart=unless-stopped -v <主机路径>:/var/lib/rancher/ -p 80:80 -p 443:443 rancher/rancher
```

### 登陆Rancker
- 新建集群
- 在另一台或者几台机器上执行提示命令即可
- 等待集群搭建成功

### 一些Rancher操作
- 工作负载
        部署服务的地方，可以建立多个命名空间，作用是用于隔离网络的
- 负载均衡
        将服务映射到域名的功能
- 服务发现
        不用命名空间可以相互调用服务
- PVC
        卷目录映射配置

### NFS映射主机目录
- 添加应用商店

商店名称 | url
- | -
mirror-stable | http://mirror.azure.cn/kubernetes/charts/
- 启动应用商店
*nfs-server需要启动*
        找到nfs-client-provisioner启动，配置nfs.path对应目录地址 例如：/data/work/share nfs.server 对应nfs服务器 例如：192.168.1.130 
- 然后就可以在pvc里面新建存储类PV了
