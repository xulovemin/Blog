---
title: Linux常用
date: 2019-06-12 15:09:29
tags:
    - Linux
    - 爬虫相关
categories: 技术贴
---

### Centos7安装chrome浏览器

* 安装google chrome浏览器
``` cmd
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
yum install google-chrome-stable_current_x86_64.rpm
```

* chromedriver下载
``` cmd
http://npm.taobao.org/mirrors/chromedriver/
```

* python调用
``` python
chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
webdriver.Chrome(chrome_options=chrome_options)
driver.get('www.baidu.com')
```

### 安装python

* 解压
``` cmd
tar xvf Python-3.6.0a1.tar.xz
```

* 编译安装
``` cmd
./configure 
(sudo)make
(sudo)make install
```

* sudo pip命令找不到
``` cmd
https://www.cnblogs.com/white-the-Alan/p/8901118.html
```

* git移除版本控制
``` cmd
git rm -f -r --cached 文件名
```

* 查看centos版本
```cmd
cat /etc/redhat-release
```
* 查看空间
```cmd
df -h 查看磁盘空间
du -h --max-depth=1 查看当前目录占用空间
```
* 查看端口占用
```cmd
# -a 所有
# -n 显示数字
# -t 显示tcp相关
# -u 显示dup相关
# -p 显示程序名
# -l 仅显示listen的服务
# -r 显示路由信息
# -e 显示扩展信息，例如uid等
# -s 按各个协议进行统计
# -c 每隔一个固定时间，执行该netstat命令。
sudo netstat -antp|grep 5000 #查看端口占用
```
