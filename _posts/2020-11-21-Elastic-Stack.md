---
layout: post
title: Elastic Stack简介与安装
categories: 日志
description: elk
keywords: elk, elastic
---

## Elastic Stack(elk)简介

### ES概念

ELK领域：

* 搜索引擎
* 日志分析
* 指标分析

大纲：

* Elasticsearch的介绍
* 中文分词
* 全文搜索
* Elasticsearch集群



ELK = Elasticsearch + Logstash + Kibana

Elatsic Stack = ELK + Beats

![截屏2020-11-21_下午10-27-46.png](\images\posts\elk\截屏2020-11-21_下午10-27-46.png)

![eef585b90da241b96fd520912eef1f25.png](\images\posts\elk\eef585b90da241b96fd520912eef1f25.png)

Beats

* Beats是elastic公司开源的一款采集系统监控数据的代理agent，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给Elasticsearch或者通过Logstash发送给Elasticsearch，然后进行后续的数据分析活动。

Beats由如下组成:

* Packetbeat：是一个网络数据包分析器，用于监控、收集网络流量信息，Packetbeat嗅探服务器之间的流量，
* Filebeat：用于监控、收集服务器日志文件，其已取代 logstash forwarder
* Metricbeat：可定期获取外部系统的监控指标信息，其可以监控、收集 Apache、HAProxy、MongoDB
* Winlogbeat：用于监控、收集Windows系统的日志信息；

#### Elasticsearch

* 阅读：Elastic(伊莱斯提克-公司名)search(色赤)
* 全文检索搜索引擎
* github使用的就是这个
* 基于Lucene封装了一层外壳，使操作简单化，天生分布式集群高可用

全文解锁：

* 搜索字符查询出整个文章

Lucene:

* 强大的搜索引擎库
* 倒排索引
  * 将数据库里的词拆成一个个的词，根据词建立索引，根据次数打分排序
  * 搜索的有模糊的词，会拆词，评分最高的有限展示

* 是一个jar包，基于其API进行搜索引擎的开发

Elasticsearch的功能

* 分布式搜索和分析引擎
* 全文检索，结构化检索，数据分析
* 对海量数据进行近实时的处理

适用场景：

* 全文检索，高亮，搜索推荐
* 日志数据分析，logstash采集日志，ES进行复杂的数据分析
* BI系统，ES执行数据分析和挖掘，Kibana进行数据可视化

#### Elasticsearch的特点

* 可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司，也可以运行在单机上，服务小公司
* Elasticsearch不是什么新技术，主要是将全文检索，数据分析以及分布式技术，合并在一起，才形成了独一无二的ES，lucene(全文检索)
* 对用户而言，是开箱即用的，非常简单，作为中小型的应用，直接3分钟部署一下ES，就可以作为生产环境的系统来使用了，数据量不大，操作不是太复杂
* 数据库的功能面对很多领域是不够的，优势：事务，各种联机事务型的操作，特殊的功能，比如全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理，Elasticsearch作为传统数据库的一个补充，提供了数据库所不能提供的很多功能。

#### Lucene和Elasticsearch关系

* lucene，最先进，功能最强大的搜索库，接基于lucene开发，非常复杂，api复杂
* Elasticsearch,基于lucene，隐藏复杂性，提供简单易用的restful api接口，java api接口，还有其他语言的接口

（1）分布式的文档存储引擎

（2）分布式的搜索引擎和分析引擎

（3）分布式，支持PB级数据

* 开箱即用，优秀的默认参数，不需要任何额外配置，完全开源

#### Elasticsearch与数据库

=========================

Document  行

Type  表

Index  库

filed  字段

el以json格式存储

![b0bd311008b0254055b408433d66acb6.png](\images\posts\elk\b0bd311008b0254055b408433d66acb6.png)

### ES名词

**索引词**

* 在elastiasearch中索引词（term）是一个能够被索引的精确值。foo,Foo,FOO几个单词是不同的索引词。索引词（term）是可以通过term查询进行准确的搜索。(中文词不支持)

**文本(text)**

* 文本是一段普通的非结构化文字。通常，文本会被分拆成一个个的索引词，存储在elasticsearch的索引库中。为了让文本能够进行搜索，文本字段需要事先进行分析了；当对文本中的关键词进行查询的时候，搜索引擎应该根据搜索条件搜索出原文本。

**分析(analysis)**

* 分析是将文本转换为索引词的过程，分析的结果依赖于分词器。比如：FOO BAR，Foo-Bar和foo bar这几个词有可能会被分析成相同的索引词foo和bar，这些索引词存储在Elasticsearch的索引库中。

**集群(cluster)**

* 集群由一个或多个节点组成，对外提供服务，对外提供索引和搜索功能。在所有节点，一个集群有一个唯一的名称默认为“elasticsearch”.此名称是很重要的，因为每个节点只能是集群的一部分，当该节点被设置为相同的集群名称时，就会自动加入集群。当需要有多个集群的时候，要确保每个集群的名称不能重复，，否则节点可能会加入到错误的集群。请注意，一个节点只能加入到一个集群。此外，你还可以拥有多个独立的集群，每个集群都有其不同的集群名称。

**节点(node)**

* 一个节点是一个逻辑上独立的服务,它是集群的一部分,可以存储数据,并参与集群的索引和搜索功能。就像集群一样,节点也有唯一的名字,在启动的时候分配。如果你不想要默认名称,你可以定义任何你想要的节点名.这个名字在理中很重要,在Elasticsearch集群通过节点名称进行管理和通信.一个节点可以被配置加入到一个特定的集群。默认情况下,每个节点会加人名为Elasticsearch 的集祥中,这意味着如果你在网热动多个节点,如果网络畅通,他们能彼此发现井自动加人名为Elasticsearch 的一个集群中,你可以拥有多个你想要的节点。当网络没有集祥运行的时候,只要启动一个节点,这个节点会默认生成一个新的集群,这个集群会有一个节点。

**分片(shard)**

* 分片是单个Lucene 实例,这是Elasticsearch管理的比较底层的功能。索引是指向主分片和副本分片的逻辑空间。 对于使用,只需要指定分片的数量,其他不需要做过多的事情。在开发使用的过程中,我们对应的对象都是索引,Elasticsearch 会自动管理集群中所有的分片,当发生故障的时候,Elasticsearch 会把分片移动到不同的节点或者添加新的节点。
* 一个索引可以存储很大的数据,这些空间可以超过一个节点的物理存储的限制。例如,十亿个文档占用磁盘空间为1TB。仅从单个节点搜索可能会很慢,还有一台物理机器也不一定能存储这么多的数据。为了解决这一问题,Elasticsearch将索引分解成多个分片。当你创建一个索引,你可以简单地定义你想要的分片数量。每个分片本身是一个全功能的、独立的单元,可以托管在集群中的任何节点主分片
* 每个文档都存储在一个分片中,当你存储一个文档的时候,系统会首先存储在主分片中,然后会复制到不同的副本中。默认情况下,一个索引有5个主分片。 你可以事先制定分片的数量,当分片一旦建立,则分片的数量不能修改。副本分片每一个分片有零个或多个副本。副本主要是主分片的复制,其中有两个目的:
  * - 增加高可用性:当主分片失败的时候,可以从副本分片中选择一个作为主分片。
  * - 提高性能:当查询的时候可以到主分片或者副本分片中进行查询。默认情況下,一个主分片配有一个副本,但副本的数量可以在后面动态地配置增加。副本分片必部署在不同的节点上,不能部署在和主分片相同的节点上。

* 分片主要有两个很重要的原因是:
  * - 允许水平分割扩展数据。
  * - 允许分配和井行操作(可能在多个节点上)从而提高性能和吞吐量。

* 这些很强大的功能对用户来说是透明的,你不需要做什么操作,系统会自动处理。

**索引(index)**

* 索引是具有相同结构的文档集合。例如,可以有一个客户信息的索引,包括一个产品目录的索引,一个订单数据的索引。 在系统上索引的名字全部小写,通过这个名字可以用来执行索引、搜索、更新和删除操作等。在单个集群中,可以定义多个你想要的索引。

**类型(type)**

* 在索引中,可以定义一个或多个类型,类型是索引的逻辑分区。在一般情况下,一种类型被定义为具有一组公共字段的文档。例如,让我们假设你运行一个博客平台,并把所有的数据存储在一个索引中。在这个索引中,你可以定义一种类型为用户数据,一种类型为博客数据,另一种类型为评论数据。

**文档(doc)**

* 文档是存储在Elasticsearch中的一个JSON格式的字符串。它就像在关系数据库中表的一行。每个存储在索引中的一个文档都有一个类型和一个ID,每个文档都是一个JSON对象,存储了零个或者多个字段,或者键值对。原始的JSON 文档假存储在一个叫作Sour的字段中。当搜索文档的时候默认返回的就是这个字段。

**映射**

* 映射像关系数据库中的表结构,每一个索引都有一个映射,它定义了索引中的每一个字段类型,以及一个索引范围内的设置。一个映射可以事先被定义,或者在第一次存储文档的时候自动识别。

**字段**

* 文档中包含零个或者多个字段,字段可以是一个简单的值(例如字符串、整数、日期),也可以是一个数组或对象的嵌套结构。字段类似于关系数据库中表的列。每个字段都对应一个字段类型,例如整数、字符串、对象等。字段还可以指定如何分析该字段的值。

**主键**

* ID是一个文件的唯一标识,如果在存库的时候没有提供ID,系统会自动生成一个ID,文档的 index/type/id必须是唯一的。

**复制**

* 复制是一个非常有用的功能,不然会有单点问题。 当网络中的某个节点出现问题的时候,复制可以对故障进行转移,保证系统的高可用。因此,Elasticsearch 允许你创建一个或多个拷贝,你的索引分片就形成了所谓的副本或副本分片。
* 复制是重要的,主要的原因有:
  * - 它提供丁高可用性,当节点失败的时候不受影响。需要注意的是,一个复制的分片不会存储在同一个节点中。

* 它允许你扩展搜索量,提高并发量,因为搜索可以在所有副本上并行执行。
* 每个索引可以拆分成多个分片。索引可以复制零个或者多个分片.一旦复制,每个索引就有了主分片和副本分片.分片的数量和副本的数量可以在创建索引时定义.当创建索引后,你可以随时改变副本的数量,但你不能改变分片的数量.
* 默认情況下,每个索引分配5个分片和一个副本,这意味着你的集群节点至少要有两个节点,你将拥有5个主要的分片和5个副本分片共计10个分片.
* 每个Elasticsearch分片是一个Lucene 的索引。有文档存储数量限制,你可以在一个单一的Lucene索引中存储的最大值为lucene-5843,极限是2147483519(=integer.max_value-128)个文档。你可以使用cat/shards API监控分片的大小。

---

## ES安装

### 配置文件

* ES不支持root用户运行，创建elatsic search用户来运行
* ES在删除索引后不会立即从磁盘移除，他只是被标记为已删除，ES将会在添加更多的索引时候才会在后台进行删除，不然直接删除磁盘压力会很大

#### logstash配置

~~~shell
- 布尔类型 Boolean
  - isFailed => true
- 数值类型 Number
  - port => 33
- 字符串类型 String
  - name => "Hello world"
- 数组 Array/List
  - users => [ {id => 1, name => pbp}, {id => 2, name => jane}]
  - path => ["/var/log/messages", "/var/log/*.log"]
- 哈希类型 Hash
  - match => {
    - "field1" => "value1"
    - "field2" => "value2"
  - }
  注释：
- 井号 #
引用Event的属性字段：

- 直接引用字段值
  - 使用 【】 即可，嵌套字段写多层即可
  - if [request] {}
- 在字符串中以sprintf方式引用
  - 使用j %{} 来实现
  - req => "request is %{request}"
表达式：
- 比较：==, !=, <, >, <=, >=
- 正则是否匹配： =~，!~
- 包含（字符串或数组）：in，not in
- 布尔操作符：and，or， nand，xor，！
- 分组操作符：()
~~~

#### Logstash Input插件

通用配置字段：

* codec 类型为  codec
* type 类型为 string，自定义事件的类型，可用于后续判断
* tags 类型为 array，自定义该事件的tag，可用于后续判断
* add_field 类型为 hash，为该事件添加字段

Filter插件：

![d41a6a4b82bf3dd999cb9725bffd918e.png](\images\posts\elk\d41a6a4b82bf3dd999cb9725bffd918e.png)

### ES安装

* 安装java
  * yum install -y java-1.8.0-openjdk.x86_64

* 下载安装软件
  * mkdir  -p /data/es_soft/
  * cd /data/es_soft/
  * wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.0.rpm
  * rpm -ivh elasticsearch-6.6.0.rpm

* 配置启动
  * systemctl daemon-reload
  * systemctl enable elasticsearch.service
  * systemctl start elasticsearch.service
  * systemctl status elasticsearch.service

* 检查是否启动成功
  * ps -ef|grep elastic
  * lsof -i:9200
  * curl 127.0.0.1:9200有返回

---

查看配置文件：

* rpm -ql elasticsearch  #查看elasticsearch软件安装了哪些目录
* rpm -qc elasticsearch  #查看elasticsearch的所有配置文件

#### 配置文件

* /etc/elasticsearch/elasticsearch.yml    #ES主配置文件
* /etc/elasticsearch/jvm.options.            #jvm虚拟机配置文件
* /etc/init.d/elasticsearch                       #init启动文件
* /etc/sysconfig/elasticsearch                #环境变量配置文件
* /usr/lib/sysctl.d/elasticsearch.conf       #sysctl变量文件，修改最大描述符，所以不需要手动修改了
* /usr/lib/systemd/system/elasticsearch.service  #systemd启动文件

目录文件：

* /var/lib/elasticsearch  # 数据目录
* /var/log/elasticsearch  #日志目录
* /var/run/elasticsearch  #pid目录

---

elasticsearch.yml配置相关：

Elasticsearch 已经有了很好的默认值，特别是涉及到性能相关的配置或者选项,其它数据库可能需要调优，但总得来说，Elasticsearch不需要。如果你遇到了性能问题，解决方法通常是更好的数据布局或者更多的节点。

查看所有修改过的配置

* egrep -v "^#" /etc/elasticsearch/elasticsearch.yml
* 修改完配置文件后我们需要重启一下
  * systemctl restart elasticsearch
  * systemctl status elasticsearch

配置设置：

* cluster.name: dba5   #设置集群名称
* node.name: node-1  #设置节点名称
* path.data: /data/elasticsearch  #数据目录
* path.logs: /var/log/elasticsearch  #日志目录
* bootstrap.memory_lock: true  #锁定内存,结合jvm.options. 一起修改，启动时内存锁死，为了防止内存被其他占用--需要打开
* network.host: localhost  #绑定IP地址-需要修改为内网ip
* http.port: 9200  #端口号
* discovery.zen.ping.unicast.hosts: [“localhost”]  #集群发现的通讯节点--在制作es集群时，用于互相发现对方
* discovery.zen.minimum_master_nodes: 2  #最小主节点数--在制作es集群时，选项相关参数,有公式 master/2 +1

注意：

* 修改了数据目录：/data/elasticsearch
  * 因为开启el的同时会创建elasticsearch用户，所以需要修改el下的所有文件的归属者
  * mkdir /data/elasticsearch
  * chown -R elasticsearch:elasticsearch /data/elasticsearch/

* jvm.options中Xms1g
  * 需要Xms1g与Xmx1g最小与最大一样大，将内存锁死
  * 需要配和bootstrap.memory_lock: true一起使用
  * 官方推荐内存锁定建议：
    * 不要超过32G
    * 最大最小内存设置为一样
    * 配置文件设置锁定内存
    * 至少给服务器本身空余50%的内存
    * 看数据量多大
    * 建议一点一点给，8G 12G 24G 32G

修改完重启一般会失败，官方文档有解释：

* 这个时候可能会启动失败，查看日志可能会发现是锁定内存失败
* 官方解决方案
  * https://www.elastic.co/guide/en/elasticsearch/reference/6.6/setup-configuration-memory.html
  * [https://www.elastic.co/guide/en/elasticsearch/reference/6.6/setting-system-settings.html#sysconfig](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/setting-system-settings.html#sysconfig)

* 解决方法：

![b9ead81f74a5fe6d5fe8b4c9d3404706.png](C:/Users/xindong/Desktop/1/notes/image/b9ead81f74a5fe6d5fe8b4c9d3404706.png)

------

## ES交互与es-head配置

ES的http请求需要包含的部分：

curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>’

* VERB： 适当的 HTTP 方法 或 谓词 : GET`、 `POST`、 `PUT`、 `HEAD 或者 `DELETE`。 PROTOCOL http 或者 https`(如果你在 Elasticsearch 前面有一个 `https 代理)
* HOST : Elasticsearch 集群中任意节点的主机名，或者用 localhost 代表本地机器上的节点。
* PORT : 运行 Elasticsearch HTTP 服务的端口号，默认是 9200 。
* PATH :  API 的终端路径(例如 _count 将返回集群中文档数量)。Path 可能包含多个组件，例如: _cluster/stats 和 _nodes/stats/jvm 。
* QUERY_STRING: 任意可选的查询字符串参数 (例如 ?pretty 将格式化地输出 JSON 返回值，使其更容易阅读)
* BODY : 一个 JSON 格式的请求体 (如果请求需要的话)

---

ES交互方式：

* curl命令
  * 不需要安装任何软件，只需要有curl命令

* es-head插件
  * 查看数据方便

* kibana
  * 查看数据以及报表格式丰富

Head插件在5.0以后安装方式发生了改变，需要nodejs环境支持，或者直接使用别人封装好的docker镜像

```
插件官方地址
https://github.com/mobz/elasticsearch-head
使用docker部署elasticsearch-head
docker pull alivv/elasticsearch-head
docker run --name es-head -p 9100:9100 -dit elivv/elasticsearch-head
使用nodejs编译安装elasticsearch-head
yum install nodejs npm openssl screen -y
node -v
npm  -v
npm install -g cnpm --registry=https://registry.npm.taobao.org
cd /opt/
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head/
cnpm install
screen -S es-head
cnpm run start
Ctrl+A+D
```

最好用的安装方式：

* 安装es-head的chorme插件
* 直接安装插件会损坏，需要加个zip解压安装

这两个配置别开，开了的话es-head就无法连接上了

```
* discovery.zen.ping.unicast.hosts: [“localhost”]  #集群发现的通讯节点
* discovery.zen.minimum_master_nodes: 2  #最小主节点数
```

修改ES配置文件支持跨域

http.cors.enabled: true

http.cors.allow-origin: "*"

---

集群状态颜色：

绿色:所有条件都满足，数据完整，副本满足

黄色:数据完整，副本不满足    

红色:有索引里的数据出现不完整了

紫色:有分片正在同步中

---

安装注意的内容：

1.锁定内存要修改配置

2.JVM虚拟机最大最小内存设置为一样

3.最大内存不要超过30G

4.更改数据目录需要授权用户给elasticsearch

5.es启动比较慢

![bcfad047322db77d548a2f516f1c83d0.png](C:/Users/xindong/Desktop/1/notes/image/bcfad047322db77d548a2f516f1c83d0.png)

* 主分片：有粗框

- 复合查询：

* 可以输入curl的命令

![0e2cd88e1ce30fed81f6d9d6072bb7be.png](C:/Users/xindong/Desktop/1/notes/image/0e2cd88e1ce30fed81f6d9d6072bb7be.png)

curl命令

------

## ES API

### 创建索引

* curl -XPUT '192.168.47.178:9200/vipinfo?pretty’
  * PUT 插入
  * ?pretty 以json的格式返回

插入文档数据

```
curl -XPUT '192.168.47.178:9200/vipinfo/user/1?pretty' -H 'Content-Type: application/json' -d'
{
    "first_name" : "John",
    "last_name": "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing", "interests": [ "sports", "music" ]
}'
```

---

es文档必须包含的元数据：

* _index 文档在哪存放  -- 库
  * 一个 索引 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 products 中，而存储所有销售的交易到索引 sales 中

* _type 文档表示的对象类别  -- 表
  * 数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。 例如，所 有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" 和 "lawn- care"。 这些文档共享一种相同的(或非常相似)的模式:他们有一个标题、描述、产品代码和价格。他们只是正 好属于“产品”下的一些子类。
  * Elasticsearch 公开了一个称为 types (类型)的特性，它允许您在索引中对数据进行逻辑分区。不同 types 的文档可能有不同的字段，但最好能够非常相似。

* _id 文档唯一标识 -- 唯一键ID
  * 不能重复，不手动指定会自动随机设置
  * ID 是一个字符串， 当它和 _index 以及 _type 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创 建一个新的文档，要么提供自己的 _id ，要么让 Elasticsearch 帮你生成
  * 想要自己生成id，最好建立一个其他名称的键，搜索的时候关系上就好

### ES增删改查语法

* q=条件查询

------

## ES集群

```
Elasticsearch 可以横向扩展至数百(甚至数千)的服务器节点，同时可以处理PB级数据 Elasticsearch 天生就是分布式的，并且在设计时屏蔽了分布式的复杂性。
Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。
这里列举了一些在后台自动执行的操作:
分配文档到不同的容器 或 分片中,文档可以储存在一个或多个节点中
按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
将集群中任一节点的请求路由到存有相关数据的节点
集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复
一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成,它们共同承担数据和负载的压力。
当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。
当一个节点被选举成为主节点时,它将负责管理集群范围内的所有变更,例如增加、删除索引,或者增加、删除节点等.而主节点并不需要涉及到文档级别的变更和搜索等操作,所以当集群只拥有一个主节点的情况下,即使流量的增加它也不会成为瓶颈.任何节点都可以成为主节点。我们的示例集群就只有一个节点,所以它同时也成为了主节点。
作为用户,我们可以将请求发送到 集群中的任何节点,包括主节点.每个节点都知道任意文档所处的位置,并且能够将我们的请求直接转发到存储我们所需文档的节点.无论我们将请求发送到哪个节点,它都能负责从各个包含我们所需文档的节点收集回数据,并将最终结果返回給客户端. Elasticsearch 对这一切的管理都是透明的。
```

集群的安装部署和单机没有什么区别，区别在于配置文件里配置上集群的相关参数

* cluster.name: dba6    #集群名称，一个集群内所有节点要一样
* node.name: node-1
* path.data: /data/elasticsearch
* path.logs: /var/log/elasticsearch
* bootstrap.memory_lock: true
* network.host: localhost,localhost  #节点所在机器的IP地址
* http.port: 9200
* discovery.zen.ping.unicast.hosts: [”IP1”,” IP2”]  #集群节点发现IP
* discovery.zen.minimum_master_nodes: 1  #最大master节点数
* http.cors.enabled: true
* http.cors.allow-origin: "*"

---

注意：status 字段是我们最关心的。

* green 所有的主分片和副本分片都正常运行。
* yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行。
* red 有主分片没能正常运行。

---

ES-head解析：

* 默认ES会创建5分片1副本的配置

---

**查询集群状态：**

官网地址：

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-health.html

curl -XGET ‘http://localhost:9200/_cluster/health?pretty’

{

  "cluster_name" : "elasticsearch",

  "status" : "yellow",

  "timed_out" : false,

  "number_of_nodes" : 1,

  "number_of_data_nodes" : 1,

  "active_primary_shards" : 5,

  "active_shards" : 5,

  "relocating_shards" : 0,

  "initializing_shards" : 0,

  "unassigned_shards" : 5,

  "delayed_unassigned_shards" : 0,

  "number_of_pending_tasks" : 0,

  "number_of_in_flight_fetch" : 0,

  "task_max_waiting_in_queue_millis" : 0,

  "active_shards_percent_as_number" : 50.0

}

status 字段是我们最关心的。

green 所有的主分片和副本分片都正常运行。

yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行。

red 有主分片没能正常运行。

查看系统检索信息

Cluster Stats API允许从群集范围的角度检索统计信息。

官网地址：

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html

操作命令：

curl -XGET 'http://localhost:9200/_cluster/stats?human&pretty’

查看集群的设置

官方地址：

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-get-settings.html

操作命令：

curl -XGET 'http://localhost:9200/_cluster/settings?include_defaults=true&human&pretty’

查询节点的状态

官网地址：

https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-nodes-info.html

操作命令：

curl -XGET 'http://localhost:9200/_nodes/procese?human&pretty'

curl -XGET 'http://localhost:9200/_nodes/_all/info/jvm,process?human&pretty'

curl -XGET 'http://localhost:9200/_cat/nodes?human&pretty'                  

---

2个节点,master设置为2的时候,一台出现故障导致集群不可用

解决方案:

把还存活的节点的配置文件集群选举相关的选项注释掉或者改成1

discovery.zen.minimum_master_nodes: 1

重启服务

结论:

两个节点数据不一致会导致查询结果不一致

找出不一致的数据,清空一个节点,以另一个节点的数据为准

* es会主片自动同步到副片
* 不管主还是副节点，只要有不同数据都会合并，有冲突的话谁是master用谁的
  * 所以集群一定要清空一个，以另一个为准

然后手动插入修改后的数据

---

![a9c64e66edee90fe92484f2d088f13df.png](C:/Users/xindong/Desktop/1/notes/image/a9c64e66edee90fe92484f2d088f13df.png)

集群分片：碎了

* 一定能保证一个主分片，一个副分片
* 任何一个节点损坏，数据不会丢失，因为其余两个可以拼成完成的数据

* 默认数据分配：
  * 5分片
  * 1副本

监控状态：

* 监控集群健康状态 不是 green

或

* 监控集群节点数量 不是 3

极限损坏：

3节点

坏2台节点

不需要的数据，可以在es-head索引上

* 关闭(可以先不删除)

副本数量可以在建立索引的时候设置，但是分片的个数无法设置

------

## ES监控功能

### x-pack

为elasticsearch, logstash, kibana提供了监控，报警，用户认证等功能，属于一个集成的插件。

* 6.x以前是以插件形式存在
* 6.x以后默认安装的时候就集成在软件包里了
* 默认安装的时候是基础版
* 不同版本功能对比网址：
  * [https://www.elastic.co/subscriptions](https://www.elastic.co/subscriptions)

---

由于x-pack里关于安全认证是收费功能，而我们又不想去破解

那么替代的方案就是Search Guard

相关网址：

* https://docs.search-guard.com/latest/main-concepts
* https://docs.search-guard.com/latest/demo-installer
* https://docs.search-guard.com/latest/search-guard-versions
* https://docs.search-guard.com/latest/search-guard-installation

------

## ELK数据收集

查看ELK案例网站：

* [https://demo.elastic.co/](https://demo.elastic.co/)

安装部署：

db01：2G-4G

* es, kibana, nginx, filebeat

db02: 1G

* nginx, filebeat

filebeat类似于tail -f ，关闭filebeat会记录上一次读到什么位置

* 查看filebeat支持的文件解析类型 在es官网上查看

filebeat加载配置失败，可以使用命令检查

* filebeat -c /etc/filebeat/filebeat.yml

注意：

* es-head：中的 kibana_1 不能删除，删了的话画图数据都没了，kibana无法登陆，需要重启kibana
* filebeat：每30s扫描传输一次
* 需要筛选哪个字段就add message
* 查询下面的Add a filter，可以保存筛选条件
  * 筛选条件前面的disable可以暂时屏蔽筛选条件

* 想message每个字段变成单独的信息，需要用json的格式
* filebeat只关心数据的追加，不会从头开始读取数据
  * es负责收集

### docker安装

* 安装dockers compose，进行elk设置

---

* 如果修改了filebeat的index，需要修改 
* [https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html](https://www.elastic.co/guide/en/beats/filebeat/current/elasticsearch-output.html)
* 定制索引：可以不显示多余的无用的信息
* 日志的收集最重要的步骤是filebeat
* 需要将日志文件转换为json格式
* 当多个文件，在一台机器想建立不同索引，
  * 输入：可以利用tags为日志文件打标签
  * 输出：利用标签添加不同索引

---

---

### elk收集整理

**nginx日志：**

1.默认配置收集

* 索引名固定
* 所有类型放在一起

2.自定义模板

* 自定义索引名称
* 根据日志类型打tag标签拆分

3.把nginx日志转换成json格式

* 可以按照需要的字段过滤分析

**tomact日志**

* 转换成json格式

**docker日志**

1. 默认配置不能按照容器类型拆分
2. 使用docker-compose添加了一个字段service
3. 根据服务类型和日志类型生成不同的容器类型的索引

**java日志**

* 多行匹配模式

**filebeat自带的模块modules收集日志**

* nginx
* mongo
* redis
* mysql

**kibana画图**

* 柱状图
* 折线图
* 饼图
* 仪表图
* 拼接大屏展示

两种消息队列方式：

**使用redis作为缓存**

**使用kafka作为缓存**

------

## ES收集案例

cat filebeat.yml：filebeat配置文件

```
filebeat.inputs:
- type: log //日志类型
  enabled: true
  paths:
    - /var/log/nginx/access.log //日志路径
  json.keys_under_root: true //key不用es为其添加的
  json.overwrite_keys: true   //以上两行只有输入时json格式的时候才会起作用
  tags: ["access"] //打标签，为了后面输出使用
- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log //一台机器，多个文件
  tags: ["error"]
setup.template.settings:
  index.number_of_shards: 3 //分片数
setup.kibana:
  host: "192.168.47.175:5601"
output.elasticsearch:
  hosts: ["localhost:9200"]
  indices:
    - index: "nginx_access-%{[beat.version]}-%{+yyyy.MM.dd}" //为其添加不同索引
      when.contains:
        tags: "access"
    - index: "nginx_error-%{[beat.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "error"
setup.template.name: "nginx"
setup.template.pattern: "nginx_*"
setup.template.enabled: false
setup.template.overwrite: true
```

* 不同文件打标签
  * 输出的格式判断条件可以不管时tag，也可以 key : value 来判断

* 多行日志合并一条
  * 正则表达式：multiline.pattern: '^\['   以'['开头

* 想要给每个容器打标签，可以用docker-compose
* 两个输出条件协议起，就变成了依赖关系

---

将普通日志收集为json格式方法：

* logstash：写匹配规则
* filebeat自带了解析普通nginx日志的功能
  * 官方文档： 
  * Enable module configs in the modules.d directory
  * Enable modules when you run Filebeat
  * Enable module configs in the filebeat.yml file

* 查看filebeat模块： filebeat modules list

filebeat通激活模块更改日志格式：

* 监察filebeat模块

```
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
setup.kibana:
  host: "192.168.47.175:5601"
```

* 激活nginx模块

```
filebeat modules enable nginx
```

* 修改nginx.module配置文件，添加正确与错误日志的路径
  1. ["/var/log/nginx/access.log"]
  2. egrep -v "#|^$" /etc/filebeat/modules.d/nginx.yml

```
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log"
```

* 启动报错，需要安装两个插件

```
[root@elk-175 ~]# /usr/share/elasticsearch/bin/elasticsearch-plugin install ingest-user-agent //浏览器访问lei
[root@elk-175 ~]# /usr/share/elasticsearch/bin/elasticsearch-plugin install ingest-geoip //记录ip地址
```

* 重启

------

## ELK画图

在/usr/share/filebeat/kibana下有画图模板

* 5 6指的是kibana版本

---

### 导入kibana视图

默认如果使用filbeat模版导入视图会把所有的服务都导入进去,而我们实际上并不需要这么多视图，而且默认的视图模版只能匹配filebeat-*开头的索引，所以这里我们有2个需要需要解决：

* 通过一定处理只导入我们需要的模版
* 导入的视图模版索引名称可以自定义

解决方法：

* 备份一份filebeat的kibana视图，删除不需要的视图模版文件
* 修改视图文件里默认的索引名称为我们需要的索引名称

```
修改nginx画图配置：
cp -a /usr/share/filebeat/kibana /root
find . -type f ! -name "*nginx*"|xargs rm -rf
sed -i 's#filebeat\-\*#nginx\-\*#g' Filebeat-nginx-overview.json
还有index-pattern/filebeat.json中的索引名称
替换索引名称
filebeat setup --dashboards -E setup.dashboards.directory=/root/kibana/
```

---

### 自定义画图

* 画的图可以在Dashboard中add展示

![ef7096ac52d9b286aeb14c717ffaff20.png](C:/Users/xindong/Desktop/1/notes/image/ef7096ac52d9b286aeb14c717ffaff20.png)

![a37685e0f30f9076348102786f64b74b.png](C:/Users/xindong/Desktop/1/notes/image/a37685e0f30f9076348102786f64b74b.png)

可以使用redis对日志进行缓存，然后通过logstash取出redis缓存的日志信息

* redis只能传出单节点
* 将日志收集到redis之后，通过logstash读取redis的日志
* logstash日志存放点：/etc/logstash/conf.d
* logstash启动命令：
  * /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/redis.conf

* logstash每读一条redis的数据，就删除一条redis的数据
* 相当于消费队列

### 启动elk监控

* 在kibana界面，打开
* 点击按钮即可

### 用kafaka做消息中间件

* kafaka需要和zookeeper相互依赖

------

## Logstash使用

使用条件来决定filter和output处理特定的事件

比较操作：

* 相等: ==, !=, <, >, <=, >=
* 正则: =~(匹配正则), !~(不匹配正则)
* 包含: in(包含), not in(不包含)

布尔操作：

* and(与), or(或), nand(非与), xor(非或)

一元运算符：

* !(取反)
* ()(复合表达式), !()(对复合表达式结果取反)

---

logstash:

* log4j可以直接通过tcp的方式发送

filebeat：

* 也可以直接配置log4j模块

logstash插件：

* 输入插件：可以选择输入的方式
* 编码插件：可以选择输入的数据的类型
  * json
  * xml
  * multiline：多行

* filter插件：过滤器插件

---

/var/log/message：记录操作操作日志

/var/log/audit/audit.log：记录ssh登录以及登录后的操作日志

正则匹配网站： [http://grokdebug.herokuapp.com/](http://grokdebug.herokuapp.com/)

---

log4j消息处理：

* 插件安装 ： 
* 两种安装方式：

logstash：

* 插件管理：

filebeat：

* 启动命令：systemctl restart filebeat

log4j并不适合log4cxx：解决方案

* log4cxx采用syslog的方式，发送到远程系统日志中，然后log4cxx收集系统日志
  * 弊端：无法发送远程

* logcxx采用xmlappender：logstash采用tcp接收
  * 弊端：解析xml

* grok正则网站：
  * [http://grokdebug.herokuapp.com/](http://grokdebug.herokuapp.com/)
  * pattern文件路径不能用~，必须为绝对路径
  * 采用patten文件的方式，message里一定，一定不能有空格

pattern写法：

* 数字一定要匹配全\d+；使用.*可能匹配不到

```
LOGGER .*logger=\"(?<LOGGER>[a-z]{0,20})
LEVEL .*level=\"(?<LEVEL>[A-Z]{0,20})
THREAD .*thread=\"(?<THREAD>[0-9a-z]{0,20})
MESSAGE .*CDATA\[(?<MESSAGE>.*)]]
RATE .*帧率]\s(?<rate>\d+)
```

logstash写法：

```
input {
  tcp {
    port => 4567
    codec => multiline {
        pattern => "^<log4j:event"
        negate => true
        what => "previous"
    }
  }
}
filter {
  mutate {
    gsub => [ "message", "\n", "" ]
  }
  grok {
    patterns_dir => ["/root/.logstash/patterns"]
    match => {
      "message" => [
        "%{LOGGER:logger}%{LEVEL:level}%{THREAD:thread}%{MESSAGE:message}%{RATE:rate}",
        "%{LOGGER:logger}%{LEVEL:level}%{THREAD:thread}%{MESSAGE:message}"
      ]
    }
    overwrite => ["message"]
  }
}
output {
  stdout{ }
}
```

* 定时执行

---

发送tcp消息：

* nc ip port

* >/dev/tcp/

logstash时区慢8h解决办法：ruby插件是自带的不用安装

* 参考：

```
  ruby {  
    # 新增索引日期,解决地区时差问题
    code => "event.set('index_date', event.get('@timestamp').time.localtime + 8*60*60)"
  }
  mutate {
    convert => ["index_date", "string"]
  }
```

------

## Beats

 index => "%{[@metadata][_index]}"

* metadata：input收集过来的元数据
* 将元数据里面的一个字段作为输出的一个字段

filebeats：

* 阿里云会挂载对应文件夹，所以可以指定根目录为挂载；

------

