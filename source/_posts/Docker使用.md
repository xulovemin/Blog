---
title: Docker使用
# cover_picture: images/banner.jpg
categories: 技术贴
tags:
    - docker
    - 部署
---

## dockerfile文件

### Python版本

``` cmd
FROM python:3.6
WORKDIR /usr/src/modelclassify
ADD . /usr/src/modelclassify
RUN pip install --no-cache-dir -i https://mirrors.aliyun.com/pypi/simple/ -r /usr/src/modelclassify/requirements.txt
EXPOSE 5000
CMD uwsgi Named_entity_recognition_uwsgi.ini
```

### Java版本

``` cmd
FROM java:8
WORKDIR /usr/src/wordCount
COPY . /usr/src/wordCount
EXPOSE 8080
CMD java -jar app.jar
```

### docker build 后面的 **.** 是重点

``` cmd
docker build -t jc/ner:v1.0 .
```

### docker run

``` cmd
docker run (-d后台启动) -p 5000(外部访问):80(内部暴露) jc/ner:版本号
```

### docker-compose 管理多个容器

* 安装 
``` cmd
pip install docker-compose
```
* 新建docker-compose.yml文件及配置

``` cmd
version: "3"
services:
  ner:
    image: "jc/ner:v1.1" #镜像
    ports: #端口
      - "5001:5001"
  region:
    .....
```

* 命令

``` cmd
docker-compose up (-d)
docker-compose down
docker-compose ps
```