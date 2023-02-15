
Spider ProxyPool 爬虫代理IP池
=======
[![Build Status](https://travis-ci.org/jhao104/proxy_pool.svg?branch=master)](https://travis-ci.org/jhao104/proxy_pool)
[![](https://img.shields.io/badge/Powered%20by-@j_hao104-green.svg)](http://www.spiderpy.cn/blog/)
[![Requirements Status](https://requires.io/github/jhao104/proxy_pool/requirements.svg?branch=master)](https://requires.io/github/jhao104/proxy_pool/requirements/?branch=master)
[![Packagist](https://img.shields.io/packagist/l/doctrine/orm.svg)](https://github.com/jhao104/proxy_pool/blob/master/LICENSE)
[![GitHub contributors](https://img.shields.io/github/contributors/jhao104/proxy_pool.svg)](https://github.com/jhao104/proxy_pool/graphs/contributors)
[![](https://img.shields.io/badge/language-Python-green.svg)](https://github.com/jhao104/proxy_pool)

### Spider Proxy Pool

爬虫代理IP池项目,主要功能为定时采集网上发布的免费代理验证入库，定时验证入库的代理保证代理的可用性，提供API和CLI两种使用方式。同时你也可以扩展代理源以增加代理池IP的质量和数量。

* 文档: [document](https://proxy-pool.readthedocs.io/zh/latest/) [![Documentation Status](https://readthedocs.org/projects/proxy-pool/badge/?version=latest)](https://proxy-pool.readthedocs.io/zh/latest/?badge=latest)

* 支持版本: ![](https://img.shields.io/badge/Python-2.x-green.svg) ![](https://img.shields.io/badge/Python-3.x-blue.svg)

* 测试地址: http://demo.spiderpy.cn (勿压谢谢)

* 付费代理推荐: [luminati-china](https://brightdata.grsm.io/proxyPool). 国外的亮数据BrightData（以前叫luminati）被认为是代理市场领导者，覆盖全球的7200万IP，大部分是真人住宅IP，成功率扛扛的。付费套餐多种，需要高质量代理IP的可以注册后联系中文客服，开通后有5美金赠送和教程指引(PS:用不明白的同学可以参考这个[使用教程](https://www.cnblogs.com/jhao/p/15611785.html))。


[//]: # (### 运行项目)

##### 下载代码:

* git clone

```bash
git clone git@github.com:xuefeng365/spider_proxy_pool.git
```

##### 安装依赖:

```bash
# 先安装 包管理工具 poetry
pip install poetry

# 通过poetry安装项目依赖，项目根目录执行以下命令
poetry install
```

##### 更新配置:


```python
# setting.py 为项目配置文件

# 配置API服务

HOST = "0.0.0.0"               # IP
PORT = 5000                    # 监听端口


# # 配置数据库
# DB_CONN = 'redis://:pwd@127.0.0.1:6379/0'
# 无密码
DB_CONN = 'redis://:@127.0.0.1:6379/0'

# proxy table name 表名(自己建的)
TABLE_NAME = 'use_proxy'


# 配置 ProxyFetcher
# 这里是启用的代理抓取方法名，所有fetch方法位于fetcher/proxyFetcher.py
PROXY_FETCHER = [
  "freeProxy01",
  "freeProxy02",
  "freeProxy03",
  "freeProxy04",
  "freeProxy05",
  "freeProxy06",
  "freeProxy07",
  "freeProxy08",
  "freeProxy09",
  "freeProxy10"
]
# 主要修改的几项配置是监听端口（PORT）、 Redis 数据库的配置（DB_CONN）和启用的代理方法名（PROXY_FETCHER）。

```

## 启动项目（以下是windows下使用教程）:
这个项目总体分为两个部分：爬取代理 IP 和 取用代理 IP。
### 爬取代理 IP
如果是windows本地爬取IP,需要先安装了redis,并启动redis服务(如图)

![image-20230209202237371](http://biji.51automate.cn/blogs/img/image-20230209202237371.png)
![](http://biji.51automate.cn/blogs/img/20230209204139.png)
```
# 爬取ip
python proxyPool.py schedule
```
程序每隔一段时间就会定时爬取一下，直到我们的 IP 池里面有一定数量的可用 IP ,IP被保存在Redis中,检测到不可用就会丢弃。
#### 读取数据库中的ip
```python
import redis

r = redis.StrictRedis(host="127.0.0.1", port=6379, db=0)
result = r.hgetall('use_proxy')
result.keys()
```
![](http://biji.51automate.cn/blogs/img/20230209204740.png)

### 取用代理 IP
你需要先启动 webApi 服务
```bash
python proxyPool.py server

```


#### 使用方法

* Api

启动web服务后, 默认配置下会开启 http://127.0.0.1:5000 的api接口服务:

| api | method | Description | params|
| ----| ---- | ---- | ----|
| / | GET | api介绍 | None |
| /get | GET | 随机获取一个代理| 可选参数: `?type=https` 过滤支持https的代理|
| /pop | GET | 获取并删除一个代理| 可选参数: `?type=https` 过滤支持https的代理|
| /all | GET | 获取所有代理 |可选参数: `?type=https` 过滤支持https的代理|
| /count | GET | 查看代理数量 |None|
| /delete | GET | 删除代理  |`?proxy=host:ip`|


* 爬虫使用

　　如果要在爬虫代码中使用的话， 可以将此api封装成函数直接使用，例如：

```python
import requests

def get_proxy():
    return requests.get("http://127.0.0.1:5000/get/").json()

def delete_proxy(proxy):
    requests.get("http://127.0.0.1:5000/delete/?proxy={}".format(proxy))

# your spider code

def getHtml():
    # ....
    retry_count = 5
    proxy = get_proxy().get("proxy")
    while retry_count > 0:
        try:
            html = requests.get('http://www.example.com', proxies={"http": "http://{}".format(proxy)})
            # 使用代理访问
            return html
        except Exception:
            retry_count -= 1
    # 删除代理池中代理
    delete_proxy(proxy)
    return None
```
## 启动项目（以下是Linux系统 docker启动教程）:



### 环境准备

* Linux x64
* docker or 本地
* git 工具



### 项目搭建


#### docker 安装（如系统已经有 请跳过）

1.  一键自动化安装:

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

2. 启动docker

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

3. docker切换镜像源

```bash
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://yytcclg8.mirror.aliyuncs.com"]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
```



#### Redis 安装

1.  利用docker安装redis

```dockerfile
sudo docker pull redis
```

2.  启动redis
```bash
sudo docker run -d --name redis -p 6379:6379 redis --requirepass
```


3. 拉取镜像
```python
docker pull spider_proxy_pool:v1

docker run --env DB_CONN=redis://:@ip:6379/0 -p 5000:5000 spider_proxy_pool:v1
```

4. 查看效果(你的公网ip+5000端口，这里是假的)
   222.222.222.222:5000