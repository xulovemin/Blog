---
title: Python实用工具
date: 2019-12-26 08:38:50
categories: 工具
tags:
    - 日常
    - 辅助
---

### 工具汇总

* 扫描第三方包
```cmd
pip install pipreqs
pipreqs ./ --encoding=utf8
```
* 查询安装模块
```cmd
pip freeze >requirements.txt # 查询安装模块
pip download -d D:\pypackage -r requirements.txt  # 推荐使用
pip install --no-index --find-links=D:\pypackage -r requirements.txt # 拷贝过来的文件
```
* 共享电脑文件
```cmd
python -m http.server 8888 
```

### 网站视频下载神器

* 下载依赖包
```cmd
pip install you-get
```
* 查看视频的详细信息
```cmd
you-get -i url
```
* 参数详解
```cmd
-o 文件绝对路径
-O 文件重命名
--format=flv 需要下载的版本号
```
* 获取RUL的json信息
```cmd
you-get --json url
```

* py注册到nacos给springcloud调用
```python
from flask import Flask
import nacos
from apscheduler.schedulers.background import BackgroundScheduler
from flask_restful import Resource, fields, marshal_with, Api

app = Flask(__name__)
api = Api(app)

# 后台定时任务
scheduler = BackgroundScheduler()

SERVER_ADDRESSES = "localhost:8848"

na_client = nacos.NacosClient(SERVER_ADDRESSES)
#注册到nacos
na_client.add_naming_instance('py', 'localhost', 5000)

@scheduler.scheduled_job('cron', second='*/5')
def send_heartbeat():
    na_client.send_heartbeat('py', 'localhost', 5000)

#定时5秒向nacos发送心跳
scheduler.start()

class Index(Resource):
    @marshal_with(i_filed)
    def get(self):
        return {'data': ''}


api.add_resource(Index, '/')

app.run()
```

* springcloud项目配置fegin调用
```java
package com.example.service;

import com.example.vo.Result;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

@FeignClient(value = "py", fallback = PyInterFaceImpl.class)
public interface PyInterFace {

    @GetMapping("/")
    Result top();

}

```