## HTTP


### URL

##### URI主要有两个子集： `URL` 和 `URN`

* URL通过资源位置来标识资源
* URN通过名字来识别资源

*PURL* 永久统一资源定位符，用URL来实现URN功能。


##### URL语法由 9 部分组成

``````
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
``````

* `path` 分为多个路径段(path segment)，每个路径段都有自己的 `params` .
* `frag` 字段不会传送给服务器，由客户端内部使用。




### 报文

HTTP使用术语流入 `inbound` 和流出 `outbound` 来描述事务处理 `transaction` .

报文*发送者*总是位于*接收者*的上游(upstream)，所有报文都会向下游(downstream)流动。


##### 报文的语法

请求报文：

``````xml
<method> <request-URL> <version>
<headers>

<entity-body>
``````

响应报文:

``````xml
<version> <status> <reason-phrase>
<headers>

<entity-body>
``````

起始行和首部总是以 `CRLF` 结束。


状态码分类：

| 整个范围    | 已定义范围   | 分类         |
| ----------- |:------------:| ------------:|
| 100 ~ 199   | 100 ~ 101    | 信息提示     |
| 200 ~ 299   | 200 ~ 206    | 成功         |
| 300 ~ 399   | 300 ~ 305    | 重定向       |
| 400 ~ 499   | 400 ~ 415    | 客户端错误   |
| 500 ~ 599   | 500 ~ 505    | 服务器错误   |


首部分类

* 通用首部：可以出现在请求、响应报文中
* 请求首部
* 响应首部
* 实体首部
* 扩展首部


##### 方法

常用HTTP方法： GET HEAD POST PUT TRACE OPTIONS DELETE.
由于HTTP设计得易于扩展，服务器还可以定义自己的`扩展方法`。

GET和HEAD被认为是 `安全方法` 。

###### GET

HTTP1.1 服务器必须实现此方法。用于请求服务器发送某个资源。

###### HEAD

HTTP1.1 服务器必须实现此方法。
与 `GET` 类似，但服务器在响应中只返回首部。开发者要确保其首部与GET返回的相同。

###### PUT

向服务器写入文档。

###### POST

向服务器输入数据。通常用来支持HTML表单。

###### TRACE

发起“环回”诊断，查看所有中间HTTP应用程序组成的链。主要用于诊断。

###### OPTIONS

请求Web服务器告知其支持的各种功能。

###### DELETE

删除请求URL所指定的资源。



