---
layout: post
title: Elastic Stack简介与安装
categories: Elastic Search
description: elk
keywords: elk, elastic
---

### Elastic Stack(elk)简介

ELK领域：

* 搜索引擎
* 日志分析
* 指标分析

大纲：

* Elasticsearch的介绍
* 中文分词
* 全文搜索
* Elasticsearch集群

---

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

---

### ES安装

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