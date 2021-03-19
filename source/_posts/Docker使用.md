---
title: Docker使用
# cover_picture: images/banner.jpg
categories: 技术贴
tags:
    - docker
    - 部署
---

### 阿里源安装docker
- yum安装
```cmd
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum -y install docker-ce
```

### 二进制安装
- 安装
```cmd
tar -xvf docker-17.03.0-ce.tgz
cp docker/* /usr/local/bin
vim /etc/systemd/system/docker.service
``` 
- service内容
```cmd
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.io
[Service]
Environment="PATH=/usr/local/bin:/bin:/sbin:/usr/bin:/usr/sbin"
EnvironmentFile=-/run/flannel/docker
ExecStart=/usr/local/bin/dockerd --log-level=error $DOCKER_NETWORK_OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
```
- 执行命令
```cmd
systemctl daemon-reload   //重载systemd下 xxx.service文件
systemctl status docker
systemctl start docker    //启动Docker
systemctl enable docker.service   //设置开机自启
```


### docker 安装目录修改
- 软连接方式
```cmd
systemctl start docker
systemctl stop docker
cd /var/lib
cp –rf docker /data
rm –rf docker
ln –s /data/docker docker
```
- 镜像导入导出
```cmd
docker save nginx > nginx.tar
docker load < nginx.tar
```
- 默认配置文件

- usr/lib/systemd/system/docker.service

- 如果更改存储目录就添加

- -- graph=/opt/docker

- 如果更改 DNS 

- 默认采用宿主机的 dns

- -- dns=xxxx 的方式指定

### dockerfile 文件
* Python版本
``` cmd
FROM python:3.6
WORKDIR /usr/src/modelclassify
ADD . .
RUN pip install --no-cache-dir -i https://mirrors.aliyun.com/pypi/simple/ -r requirements.txt
EXPOSE 5000
CMD uwsgi Named_entity_recognition_uwsgi.ini
```
* Java版本
``` cmd
FROM java:8
WORKDIR /usr/src/wordCount
ADD . .
EXPOSE 8080
CMD java -jar app.jar
```
* docker build 后面的 **.** 是重点
``` cmd
docker build -t jc/ner:v1.0 .
```
* docker run
``` cmd
docker run -d(后台启动) -p 5000(外部访问):80(内部暴露) jc/ner:版本号
```

### docker-compose 单机容器编排
* 安装 
``` cmd
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
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

### docker 常用命令
* 镜像类
``` cmd
docker build --rm=true . 构建镜像
docker pull ${IMAGE} 安装镜像
docker images 显示已经安装的镜像
docker images --no-trunc 显示已经安装镜像的详细内容
docker rmi ${IMAGE_ID} 删除指定镜像
docker rmi -f $(docker images | grep none | awk '{print $3}' | xargs)
docker rmi -f $(docker images | awk "/^<none>/ { print $3 }") 删除所有是none的dockers镜像
docker rmi $(docker images --quiet --filter &quot;dangling=true&quot;) 删除未使用的镜像
```
* 容器类
``` cmd
docker run 运行容器
docker ps 显示正在运行的容器
docker ps -a 显示所有的容器
docker stop ${CID} 停止指定容器
docker ps -a --filter &quot;exited=1&quot; 显示所有退出状态为1的容器
docker rm ${CID} 删除指定容器
docker ps -a | grep wildfly | awk '{print $1}' | xargs docker rm -f 使用正则表达式删除容器
docker stop docker ps -q 停止所有正在运行的容器
docker rm $(docker ps -aq) 删除所有停止的容器
docker rm -f $(docker ps -a | grep Exit | awk '{ print $1 }') 删除所有退出的容器
docker exec -it ${CID} /bin/sh 进入容器打开一个shell
docker ps | grep wildfly | awk "{print $1}" 通过正则表达式查找容器的镜像ID
```
