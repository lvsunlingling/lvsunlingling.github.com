---
layout: post
title: luncene-Elasticsearch
category: 学习
tags: 
description:
---

## 简介

Elasticsearch 是一个实时的分布式搜索分析引擎, 用作全文检索、结构化搜索、分析以及这三个功能的组合

## 简单使用

### 概念

Relational DB ⇒ Databases ⇒ Tables ⇒ Rows ⇒ Columns

Elasticsearch ⇒ Indices ⇒ Types ⇒ Documents ⇒ Fields

document的可以理解为一个JSON序列对象。每个document可包含多个field。

- index 索引，即一系列documents的集合
- shard 分片，ES是分布式搜索引擎，每个索引有一个或多个分片，索引的数据被分配到各个分片上，相当于一桶水用了N个杯子装。
- replica 复制，可以理解为备份分片，相应地有primary shard（主分片）。
- 文档 档  它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。
### 启动Elasticsearch

1. 启动
```
cd elasticsearch-<version>
./bin/elasticsearch -d
# -d可以作为守护进程使用
```

2. 测试是否运行成功

```
curl 'http://localhost:9200/?pretty'
```

```
{
  "name" : "yNNWst_",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "Q3T6MObuQRinowU6NiBLlg",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

这意味着已经运行了一个节点,一个集群是cluster_name相同的一组节点的实例
可以在 **elasticsearch.yml**里面修改

3. 可视化
先下载kibana

安装sense

启动kibana并访问

```
http://localhost:5601/app/sense
```

### 交互
命令格式:

```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

eg:计算集群中文档的数量

```
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

```
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

5.x后对排序，聚合这些操作用单独的数据结构(fielddata)缓存到内存里了，需要单独开启分析功能要打开

```
curl -X PUT "localhost:9200/megacorp/_mapping/employee/" -H 'Content-Type: application/json' -d'
{
  "properties": {
    "interests": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
'
```

## 分布式特性
Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在后台自动执行的操作
- 分配文档到不同的容器 或 分片 中，文档可以储存在一个或多个节点中
- 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
- 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
- 将集群中任一节点的请求路由到存有相关数据的节点
- 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

## 集群

### 集群介绍
一个单独的节点，里面不包含任何的数据和索引,叫做空集群

一个运行中的 Elasticsearch 实例称为一个 节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为 主 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到**文档级别的变更**和**搜索**等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到 集群中的任何节点 ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。


### 集群健康
```
curl -X GET "localhost:9200/_cluster/health"
```

```
{
   "cluster_name":          "elasticsearch",
   "status":                "green", 
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```

status
- green 所有的主分片和副本分片都正常运行。
- yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行。
- red 有主分片没能正常运行。

### 分片
一个 分片 是一个底层的 工作单元 ，它仅保存了 全部数据中的一部分。

一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。

文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。

一个分片可以是 主 分片或者 副本 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

一个副本分片只是一个主分片的拷贝。 副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

在索引建立的时候就已经确定了主分片数，除非重建索引否则不能调整主分片的数目,但是副本分片数可以随时修改。

在 Elasticsearch 中， 每个字段的所有数据 都是 默认被索引的 。 即每个字段都有为了快速检索设置的专用倒排索引。而且，不像其他多数的数据库，它能在 相同的查询中 使用所有这些倒排索引，并以惊人的速度返回结果。

功能
- 容灾：primary分片丢失，replica分片就会被顶上去成为新的主分片，同时根据这个新的主分片创建新的replica，集群数据安然无恙

-  提高查询性能：replica和primary分片的数据是相同的，所以对于一个query既可以查主分片也可以查备分片，在合适的范围内多个replica性能会更优（但要考虑资源占用也会提升[cpu/disk/heap]），另外index request只能发生在主分片上，replica不能执行index request。


设置分片数量
```
curl -X PUT "localhost:9200/blogs" -H 'Content-Type: application/json' -d'
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
'
```

tips:在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。所以会显示yellow


### 扩容
假设现在又三台机器,我们关闭的节点是一个主节点Node1。而集群必须拥有一个主节点来保证正常工作，所以发生的第一件事情就是选举一个新的主节点.集群状态为yellow,当Node1恢复的时候,尝试恢复到一开始的状态

## 数据的输入与输出

Elastcisearch 是分布式的 文档 存储。它能存储和检索复杂的数据结构--序列化成为JSON文档--以 实时 的方式。 换句话说，一旦一个文档被存储在 Elasticsearch 中，它就是可以被集群中的任意节点检索到。

### 文档元数据

- _index 文档在哪存放
- _type 文档表示的对象类别
- _id 文档唯一标识
