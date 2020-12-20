---
title: nginx部署与负载
date: 2019-07-09 13:52:39
categories: 技术贴
tags:
    - nginx
    - 反向代理
    - 负载均衡
---

### nginx容器内的一些目录

* /usr/share/nginx 存放静态html

* /var/log/nginx 存放log的地方

* /etc/nginx 存放nginx.conf配置的地方，它引入的是下面路径里面的conf文件

* /etc/nginx/conf.d 引入的conf文件，默认default.conf，可以自定义

* 运行nginx时需要将default.conf拷贝出来，然后挂载到容器里面
``` cmd
docker cp containerid:/etc/nginx/conf.d/default.conf 宿主机目录
```

### 运行nginx容器

* 将内置文件挂载
``` cmd
docker run -d -p 80:80 -v 宿主机目录:/etc/nginx/conf.d nginx
```

* 然后就可以修改conf或者新增conf让nginx读取配置

### 使用配置

* docker启动一个web项目，不用映射外网端口，保证服务安全

* 配置nginx的conf文件
``` cmd
server {
    listen     80;
    location / {
        # 内网ip端口即可访问
        proxy_pass http://172.17.0.2:5000; 
    }
}
```
* nginx配置stream,新版本功能，可以代理TCP协议，可以负载MySQL、oracle等
```cmd
stream{
	upstream mysql{
		server mysql:3306;
	}
	server{
		listen 3306;
		proxy_pass mysql;
	}
}
```

* nginx支持高并发，放到服务前面缓解压力

### nginx简单的负载均衡

* 在刚才的服务基础上在启动一个服务

* 配置conf文件
``` cmd
upstream lixu_proxy { 
    server 172.17.0.2:5000 weight=1;
    server 172.17.0.4:5000 weight=2;
}
server {
    listen     80;
    access_log /var/log/nginx/nginx_access.log;
    error_log  /var/log/nginx/nginx_error.log;
    location / {
        proxy_pass http://lixu_proxy;
    }
}
```

**设备状态**

* down：表示单前的server暂时不参与负载
* weight：权重，默认为1，weight越大，负载的权重就越大
* max_fails：允许请求失败的次数默认为1。当超过最大次数时，返回proxy_next_upstream 模块定义的错误
* fail_timeout：max_fails次失败后，暂停的时间
* backup：备用服务器, 其它所有的非backup机器down或者忙的时候，请求backup机器，所以这台机器压力会最轻