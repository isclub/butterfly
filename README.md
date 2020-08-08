# butterfly

<div align=center><img src="https://github.com/meetbill/butterfly/blob/master/images/butterfly.png" width="350"/></div>

蝴蝶（轻量化 Web 框架）如同蝴蝶一样，此框架小而美，简单可依赖

```
    __          __  __            ______
   / /_  __  __/ /_/ /____  _____/ __/ /_  __
  / __ \/ / / / __/ __/ _ \/ ___/ /_/ / / / /
 / /_/ / /_/ / /_/ /_/  __/ /  / __/ / /_/ /
/_.___/\__,_/\__/\__/\___/_/  /_/ /_/\__, /
                                    /____/
```
<!-- vim-markdown-toc GFM -->

* [1 简介](#1-简介)
    * [1.1 环境](#11-环境)
    * [1.2 特性](#12-特性)
* [2 五分钟 Butterfly 体验指南](#2-五分钟-butterfly-体验指南)
    * [2.1 五分钟体验之部署 Butterfly（预计 1 分钟）](#21-五分钟体验之部署-butterfly预计-1-分钟)
    * [2.2 五分钟体验之编写 handler （预计 3 分钟）](#22-五分钟体验之编写-handler-预计-3-分钟)
    * [2.3 五分钟体验之调试 handler （预计 1 分钟）](#23-五分钟体验之调试-handler-预计-1-分钟)
* [3 设计概述](#3-设计概述)
    * [3.1 请求流程](#31-请求流程)
        * [3.1.1 butterfly 请求流程（后端请求流程）](#311-butterfly-请求流程后端请求流程)
        * [3.1.2 butterfly-fe 请求流程（前端请求流程）](#312-butterfly-fe-请求流程前端请求流程)
    * [3.2 WSGI server 服务器模型](#32-wsgi-server-服务器模型)
    * [3.3 WSGI App MVC 模型](#33-wsgi-app-mvc-模型)
        * [3.3.1 Model 是核心](#331-model-是核心)
        * [3.3.2 本框架中的 MVC](#332-本框架中的-mvc)
    * [3.4 路由自动生成](#34-路由自动生成)
* [4 使用手册](#4-使用手册)
    * [4.1 手册传送门](#41-手册传送门)
    * [4.2 举个栗子](#42-举个栗子)
* [5 版本信息](#5-版本信息)
* [6 参加步骤](#6-参加步骤)

<!-- vim-markdown-toc -->

# 1 简介

## 1.1 环境

```
env:Python 2.7
```
## 1.2 特性

```
# GET 不带参数
GET http://IP:PORT/{app}/{handler}

# GET 带参数
GET http://IP:PORT/{app}/{handler}?args1=value1

# POST 参数时，数据类型需要是 application/json

如(带 -v 参数可以查看完整请求信息)：
curl "http://127.0.0.1:8585/x/ping"                         ===>handlers/x/__init__.py:ping()
curl "http://127.0.0.1:8585/x/hello?str_info=world"         ===>handlers/x/__init__.py:hello(str_info=world)
curl -d '{"str_info":"world"}' http://127.0.0.1:8585/x/hello===>handlers/x/__init__.py:hello(str_info=world)
```

> * 方便开发
>   * (1) 无需配置路由：根据 handlers package 下 package 目录结构自动加载路由（目前不支持动态路由）
>   * (2) 参数保持一致：Handler 的参数列表与 HTTP 请求参数保持一致，HTTP 接口所需参数一目了然
>   * (3) 自动参数检查：自动对 HTTP 请求参数进行参数检查
>   * (4) 使用调试模式：简易方便的 DEBUG
> * 方便运维
>   * (1) 请求完整追溯：请求的响应 Header 中包含请求的 reqid（会记录在日志中）, 便于进行 trace
>   * (2) 自定义响应头：Header 函数中很方便自定义 HTTP header, 如增加固定的接口版本号
>   * (4) 代码耗时打点：通过代码打点可以准确获取每个方法的执行时间，耗时比较长的可能是某个 SQL，可能是某个接口

# 2 五分钟 Butterfly 体验指南

## 2.1 五分钟体验之部署 Butterfly（预计 1 分钟）
> 部署
```
$ wget https://github.com/meetbill/butterfly/archive/master.zip -O butterfly.zip
$ unzip butterfly.zip
$ cd butterfly-master/butterfly
```
> 配置端口 --- 默认 8585 端口，若无需修改可进入下一项启动
```
conf/config.py
```
> 启动
```
$ bash run.sh start
```
> 访问
```
$ curl "http://127.0.0.1:8585/x/ping"
{"stat": "OK", "randstr": "..."}

$ curl "http://127.0.0.1:8585/x/hello?str_info=meetbill"
{"stat": "OK", "str_info": "meetbill"}
```
##  2.2 五分钟体验之编写 handler （预计 3 分钟）

> 创建 app （handlers 目录下的子目录均为 app）, 直接拷贝个现有的 app 即可
```
$ cp -rf handlers/x handlers/test_app
```
> 新增 handler (app 目录下的`__init__.py` 中编写 handler 函数）
```
$ vim handlers/test_app/__init__.py

新增如下内容：

# ------------------------------ handler
@funcattr.api
def test_handler(req, info):
    return retstat.OK, {"data": info}
```
> 重启服务
```
$ bash run.sh restart
```

> 访问
```
$ curl "http://127.0.0.1:8585/test_app/test_handler?info=helloworld"
{"stat": "OK", "data": "helloworld"}
```

##  2.3 五分钟体验之调试 handler （预计 1 分钟）

假如编写的 handler 不符合预期的时候，可以通过 test_handler.py 进行调试

> 调试刚才编写的 test_handler
```
$ python test_handler.py /test_app/test_handler helloworld
... 此处会输出彩色的  DEBUG 信息
Source path:... test_handler.py
>>>>>|19:03:53.293076 4694912448-MainThread call        66             def main():
------19:03:53.294108 4694912448-MainThread line        67                 return func(*args)
    Source path:... /Users/meetbill/butterfly-master/butterfly/handlers/test_app/__init__.py
    Starting var:.. req = <xlib.httpgateway.Request object at 0x10b4148d0>
    Starting var:.. info = 'helloworld'
    >>>>>|19:03:53.294402 4694912448-MainThread call        49 def test_handler(req, info):
    ------19:03:53.294864 4694912448-MainThread line        50     return retstat.OK, {"data": info}
    |<<<<<19:03:53.294989 4694912448-MainThread return      50     return retstat.OK, {"data": info}
    Return value:.. ('OK', {'data': 'helloworld'})
Source path:... test_handler.py
|<<<<<19:03:53.295114 4694912448-MainThread return      67                 return func(*args)
Return value:.. ('OK', {'data': 'helloworld'})
Elapsed time: 00:00:00.002140
=============================================================
('OK', {'data': 'helloworld'})
=============================================================
```

# 3 设计概述

## 3.1 请求流程

### 3.1.1 butterfly 请求流程（后端请求流程）

```
       +---------------------------------------------------------+
       |                       WEB brower                        |
       +---------------------------------------------------------+
           |                        ^      ^          ^
           |                        |      |          |
           |HTTP request            |      |          |
     /     |                        |      |          |
    |  +---V-----------------------------------------------------+
    |  |                       HTTPServer                        |
    |  |     +-------------------+ put +-------------------+     |
    |  |     |ThreadPool(Queue) <+-----+ HTTPConnection    |     |
    |  |     |+---------------+  |     | +---------------+ |     |   WSGI server
    |  |     ||WorkerThread   |  |     | |HTTPRequest    | |     |
    |  |     |+---------------+  |     | +---------------+ |     |
    |  |     +-------------------+     +-------------------+     |
    |  +---------------------------------------------------------+
    |     /------------\            ^      ^          ^
    |    |   environ    |           |      |          |
    |     \------------/            |      |          |
    |  .........|...................|......|..........|...........   WSGI
    |           |                   |      |          |
    |  +--------V--------+          |      |          |
    |  |       req       |          |      |          |        (1) 封装 Request
    |  +-----------------+          |      |          |            生成 reqid
Butterfly       |                   |      |          |
    |           |                   |      |          |      \
    |  +--------V--------+          |      |          |       |(2) 路由
    |  |  apiname_getter |          |      |          |       |    在路由字典中匹配 environ['PATH_INFO']
    |  +-----------------+          |      |          |       |    {
    |           |                   |      |          |       |      '/apidemo/ping':
    |           |                   |      |          |       |        <xlib.protocol_json.Protocol object>,
    |  +--------V--------+ False +--+--+   |          |       |      '/{app}/{handler}':
    |  |is_protocol_exist|------>| 400 |   |          |       |        <xlib.protocol_json.Protocol object>
    |  +-----------------+       +-----+   |          |       |    }
    |           |                          |          |      /
    |           | (protocol_process)       |          |      \
    |           V                          |          |       |(3) 返回 Response
    |  +-----------------+                 |          |       |    参数：第一个参数为 Request 实例化对象 req
    |  | protocol        | Exception    +-----+       |       |          其他参数为 GET 请求参数名
    |  | +-------------+ |------------->| 500 |       |       |    例子：environ['QUERY_STRING']: 'age=16'
    |  | |/app1/handler| |              +-----+       |       |    +-------------handler demo---------------
    |  | |/app2/handler| |Normal+----------------------------+|    |@funcattr.api      # 标识 handler 类型
    |  | +-------------+ |----->|httpstatus, headers, content||    |def demo(req, age):# req + HTTP 请求参数
    |  +-----------------+      +----------------------------+|    |   # (状态字段，数据字段，响应头列表)
    |                                                         |    |   return "OK", {"data": age}, []
     \                                                       /     +----------------------------------------
```
### 3.1.2 butterfly-fe 请求流程（前端请求流程）

<img src="https://github.com/meetbill/meetbill_static/blob/master/butterfly/butterfly-project.png?raw=true" width = "600" alt="" align=center />

## 3.2 WSGI server 服务器模型

> 模型
```
ThreadPool 线程池模型
```
> 核心流程
```
主线程不断执行 tick() 函数，tick() 用于接受新连接（使用 HTTPConnection 封装连接）并将其放入 ThreadPool 的队列

Threadpool 管理的 WorkerThread 进行处理请求
```
> 主要的类
```
* HTTPServer      : HTTP 服务程序，基于 socket，监听在某个端口上，比如：localhost:8585
* 线程池
  * ThreadPool    : 线程池，有一个消息队列，所有的线程都等在这个消息队列上
  * WorkerThread  : HTTP 请求的处理线程，它由 ThreadPool 统一管理，它等着 ThreadPool 消息队列中的消息
* 主线程
  * HTTPConnection: HTTP 连接，WorkerThread 的实际处理体
  * HTTPRequest   : HTTP 请求，HTTP 请求和响应都由该类处理
```
## 3.3 WSGI App MVC 模型

### 3.3.1 Model 是核心

Model，也就是模型，是对现实的反映和简化。**对问题的本质的描述就是 Model。解决问题就是给问题建立 Model。**

当我们关注业务问题时，只有描述 “用户所关心的问题” 的代码才是 Model。当你的关注转移到其他问题时，Model 也会相应发生变化。

**失去了解决特定问题这一语境，单谈 Model 没有意义。**

可以说，View 和 Controller 是 Model 的一部分。

> 为什么人们要单独把 View 和 Controller 跟 Model 分开呢？
```
View 和 Controller 是外在的东西，只有 Model 是本质的东西。
外在的东西天天变化，但很少影响本质。把他们分开比较容易维护。
```

### 3.3.2 本框架中的 MVC

简单来说，Controller 和 View 分别是 Model 的 输入 和 输出。

> * Model       : 也就是模型，是对现实的反映和简化
> * View        : 也就是视图 / 视野，是你真正看到的，而非想象中的 Model
> * Controller  : 也就是控制器，是你用来改变 Model 方式

View 建议在 [butterfly-fe](https://github.com/meetbill/butterfly-fe) 实现，即前后端分离

使用 butterfly 框架时主要是编写 handler
```
                                               handler
                              +-----------------------------------------+
    +-----------+ user action | +------------+  update  +-------------+ |
    |   view    |-------------+>| controller |--------->|    model    | |
    |           |<------------+-|            |<---------|             | |
    +-----------+  update     | +------------+   notify +-------------+ |
                              +-----------------------------------------+
```
## 3.4 路由自动生成

无需人为进行路由配置操作

> 约定优于配置，配置优于实现
```
在 handlers 文件夹下的 package 中的 __init__.py 编写 handler controller 函数来完成 "功能的抽象"
controller 函数是第一个参数为 "req" 的非私有函数
```
重新启动服务时，框架会自动加载 handlers 各 package  `__init__.py` 中符合条件的函数并生成路由

> 条件（第一个参数为 "req" 的非私有函数）
```
(1) 私有函数不会被加载，即函数名是 "_" 开头
(2) 类不会被加载，即 handler 中 controller 使用函数来完成 "功能的抽象"
(3) 函数的第一个形参名需要是 "req"
    调用此函数时的实参 req 是对 HTTP request environ 的封装
```

# 4 使用手册

## 4.1 手册传送门

* [Butterfly 手册](https://github.com/meetbill/butterfly/wiki)
* [Butterfly 示例](https://github.com/meetbill/butterfly-examples)
* [Butterfly 前端](https://github.com/meetbill/butterfly-fe)
* [Butterfly nginx 配置](https://github.com/meetbill/butterfly-nginx)

## 4.2 举个栗子

> 厂内线上项目 ([接口认证方案](https://github.com/meetbill/butterfly/wiki/butterfly_cas)), 使用前后端分离 + 单点登录
>  * 整体方案的后端接口认证使用 nginx auth_request 进行验证

如下是 butterfly-fe（前端） + butterfly-auth（基于 butterfly 框架的后端接口认证） + app-backend(butterfly app) + butterfly-nginx(web 服务） 请求流程
```
               +--------------------------------------------------------------------------+
               |                          butterfly-nginx                                 |
               +--------------------------------------------------------------------------+
                       |                     |                                      |
                       V                     V                                      V
               +-------------+    +---------------------+                      +----------+
               |~* /static/  |    |= /auth/verification |                      |/         |
               |= /index.html|    |= /butterfly_401     |                      |          |
               |= /          |    |= /auth/ssologin     |                      |          |
               +-------------+    +---------------------+                      +----------+
                       |                     |                                       |
                       V                     V                                       V
+----------+       +------------+     +--------------+       +----------+     +-----------+
|web browse|       |butterfly-fe|     |butterfly-auth|       |cas-server|     |app-backend|
+----------+       +------------+     +--------------+       +----------+     +-----------+
     |                    |                  |                     |                   |
     +-------route------->|/                 |                     |                   |
     |<-------page--------+/index.html       |                     |                   |
     |                    |                  |                     |                   |
     ==================================================================== not have token
     |                    |                  |                     |                   |
     +--V----------------request api-------------------------------------------------->|
     |  +-sub request-header not have token->|(/auth/verification) |                   |
     |<-code=401,targetURL=../auth/ssologin--+                     |                   |
     |                    |                  |                     |                   |
     +--window.location.herf=directurl------>|(/auth/ssologin)     |                   |
     |<----code=302,Location=cas-server------+                     |                   |
     |                    |                  |                     |                   |
     +-----302 http://cas-server/login  login page --------------->|(/login)           |
     |<-------------code=302,set Cookie TGT=xxx -------------------+                   |
     |                    |                  |                     |                   |
     +-----302 /auth/ssologin?ticket=xxx --->|                     |                   |
     |                    |                  +-------check st----->|(/session/validate)|
     |                    |                  |<-------st vaild-----+                   |
     |<--code=302 set Cookie butterfly_token-+                     |                   |
     |                    |                  |                     |                   |
     +--302 / ----------->|                  |                     |                   |
     |<-------page--------+/index.html       |                     |                   |
     |                    |                  |                     |                   |
     ======================================================================== have token
     |                    |                  |                     |                   |
     +---V----------------request api------------------------------------------------->|
     |   +-sub request-header have token---->|/auth/verification   |                   |
     |<-------------------response-----------------------------------------------------+
```

# 5 版本信息

本项目的各版本信息和变更历史可以在[这里][changelog] 查看。

# 6 参加步骤

* 在 GitHub 上 `fork` 到自己的仓库，然后 `clone` 到本地，并设置用户信息。
```
$ git clone https://github.com/meetbill/butterfly.git
$ cd butterfly
$ git config user.name "yourname"
$ git config user.email "your email"
```
* 修改代码后提交，并推送到自己的仓库。
```
$ #do some change on the content
$ git commit -am "Fix issue #1: change helo to hello"
$ git push
```
* 在 GitHub 网站上提交 pull request。
* 定期使用项目仓库内容更新自己仓库内容。
```
$ git remote add upstream https://github.com/meetbill/butterfly.git
$ git fetch upstream
$ git checkout master
$ git rebase upstream/master
$ git push -f origin master
```

[changelog]: CHANGELOG.md
