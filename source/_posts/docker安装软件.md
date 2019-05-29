---
title: docker安装软件
date: 2019-05-28 16:22:25
categories: 技术贴
tags:
    - docker
    - 安装
---

## 安装ElasticSearch及中文分词插件

* 配置项
``` cmd
sudo sysctl -w vm.max_map_count=262144
```
* 进入镜像
``` cmd
docker exec -it 容器id /bin/bash
```
* 安装插件
``` cmd
elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/vES版本/elasticsearch-analysis-ik-ES版本.zip
```
* 退出并重启镜像
``` cmd
exit
```