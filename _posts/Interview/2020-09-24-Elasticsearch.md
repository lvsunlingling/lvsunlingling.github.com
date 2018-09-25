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

###启动Elasticsearch

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
命令:

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

将返回:

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

得到

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

#### status
- green 所有的主分片和副本分片都正常运行。
- yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行。
- red 有主分片没能正常运行。

### 分片