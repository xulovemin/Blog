---
title: Jenkins使用
date: 2020-11-29 13:33:23
categories: DevOps
tags:
    - 持续集成
    - 持续部署
---

### Jenkins安装

- 启动Jenkins
```cmd
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkinsci/blueocean
    ports:
      - 8090:8080
    volumes:
      - /data/jenkins:/var/jenkins_home
      # 这个很重要，可以让Jenkins容器里面使用docker
      - /var/run/docker.sock:/var/run/docker.sock 
```

### jenkins系统管理配置

- webhook配置（github或者gitlab）

![](./webhook.jpg)

- sshserver配置(需要下载插件Publish Over SSH)

![](./sshserver.jpg)

### jenkins全局工具配置

- jdk与git（系统默认就带、使用默认配置即可）

![](./jdk.jpg)
- docker与maven（确定之后不会立即下载，会在jenkins流程需要的时候下载）
- docker需要下载（CloudBees Docker Build and Publish plugin）这个插件
- maven需要下载（Maven Integration plugin）这个插件

![](./docker.jpg)

### jenkins全局凭据管理

- 保存git、harbor等用户名密码

![](./pingju.jpg)

### 新建jenkins任务

- 老版本构建、通过界面配置方式
- 新建项目（构建一个maven项目）
- 源码管理（配置git或者gitlab）

![](./git.jpg)
- 构建触发器（使用git或者gitlab的webhook自动触发）
- maven编译以及push到harbor

![](./maven.jpg)
- ssh远程主机运行docker容器

![](./sshrun.jpg)

### 以piplines的方式构建任务

- 主要是以groovy代码的方式构建
- 可以有逻辑判断等
- 能调用Blue Ocean查看完整构建状态
- 可以用git存储流程代码，方便
- 前面配置一样，流程在下面的代码里面

```cmd
pipeline {
    # 指定机器运行，可选any、label、docker等
    agent {
        label 'master'
    }
    stages {
        # 拉取git代码，根据分支不同建立不同的任务，到时候各自分支提交会自动出发相应的构建流程
        stage('Git Pullling') {
            steps {
                git branch: 'develop', credentialsId: 'xulovemin', url: 'https://github.com/xulovemin/demo.git'
            }
        }
        stage('Maven Build') {
            # 使用系统设置中安装的工具
            tools {
                maven 'maven'
            }
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build') {
            # 系统运行起来自动带docker环境
            steps {
                sh 'docker build -t localhost:8070/library/demo .'
                sh 'docker push localhost:8070/library/demo'
            }
        }
        stage('Deployment') {
            # 远程ssh服务器，需要下载SSH Pipeline Steps这个插件
            # 不同的构建环境ssh不同的主机
            steps {
                script {
                    def remote = [:]
                    remote.name = 'server'
                    remote.host = 'xulovemin1413.xyz'
                    remote.user = 'root'
                    remote.password = 'Lixu1989'
                    remote.port = 22
                    remote.allowAnyHosts = true
                    sshCommand remote: remote, command: """
                    docker pull xulovemin1413.xyz:8070/library/demo
                    cd /work/demo
                    docker-compose down
                    docker-compose up -d
                    """
                }
            }
        }
    }
}
```

- 查看Blue Ocean流程图

![](./blue.jpg)
