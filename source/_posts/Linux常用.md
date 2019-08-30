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