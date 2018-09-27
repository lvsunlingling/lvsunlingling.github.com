---
layout: post
title: luncene-Elasticsearch
category: 学习
tags: 
description:
---

## 简介

Elasticsearch 是一个实时的分布式搜索分析引擎, 用作全文检索、结构化搜索、分析以及这三个功能的组合

## 概念

Relational DB ⇒ Databases ⇒ Tables ⇒ Rows ⇒ Columns

Elasticsearch ⇒ index ⇒ type ⇒ Document ⇒ field

Elasticsearch ⇒ 索引 ⇒ 类型 ⇒ 文档

- index 索引，即一系列documents的集合
- document的可以理解为一个JSON序列对象。它是指最顶层或者根对象。每个document可包含多个field。
- shard 分片，ES是分布式搜索引擎，每个索引有一个或多个分片，索引的数据被分配到各个分片上，相当于一桶水用了N个杯子装。
- replica 复制，可以理解为备份分片，相应地有primary shard（主分片）。

## 启动

1. 启动
```
cd elasticsearch-<version>
./bin/elasticsearch -d
# -d可以作为守护进程使用
```

2. github Head 插件安装

```
wget https://github.com/mobz/elasticsearch-head/archive/master.zip
unzip xxx.zip
npm install phantomjs-prebuilt@2.1.14 --ignore-scripts
npm install
npm run start
```
---
修改 elasticsearch 解决跨域问题
```
vim config/elasticsearch.yml

http.cors.enabled: true
http.cors.allow-origin: "*"
```

3. 分布式运行
主节点
```
vim config/elasticsearch.yml

http.cors.enabled: true
http.cors.allow-origin: "*"

cluster.name: myCluster
node.name: master
node.master: true

network.host: 127.0.0.1
```
--- 
```
vim config/elasticsearch.yml

cluster.name: myCluster
node.name: slave1

network.host: 127.0.0.1
http.port: 9201

discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```
---
```
vim config/elasticsearch.yml

cluster.name: myCluster
node.name: slave2

network.host: 127.0.0.1
http.port: 9202

discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```

2. 测试是否运行成功

```
# pretty是格式化,默认显示节点信息
curl 'http://localhost:9200/?pretty'
# 或者访问 http://localhost:9100/
```

## 基本用法
命令格式:
```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

## 基本概念

### 文档元数据
- _index 文档在哪存放
- _type 文档表示的对象类别
- _id 文档唯一标识
- _version 版本号

## 增删改查
### 索引创建
```
PUT 127.0.0.1:9200/people
{
	"settings":{
		"number_of_shards":3,
		"number_of_replicas":1
	},
	"mappings":{
		"man":{
			"properties":{
				"name":{
					"type":"text"
				},
				"country":{
					"type":"keyword"
				},
				"age":{
					"type":"integer"
				},
				"date":{
					"type":"date",
					"format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
				}
			}
		}
	}
}
```

### 插入
PUT/POST 127.0.0.1:9200/people/man/1
```
{
  "name": "ss newcome",
  "country":  "newcome",
  "age":  "28",
  "data": "2014/01/01"
}
```

--- 
为了解决id冲突问题
```
PUT /website/blog/123?op_type=create
# 或者
PUT /website/blog/123/_create
```

_create如果冲突话返回一个

```
{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```

### 修改
本质上还是替换
```
POST 127.0.0.1:9200/people/man/1/_update
{
	"doc":{
		"name":"谁是newcome"
	}
}
```
---
```
POST 127.0.0.1:9200/people/man/1/_update
{
	"script": {
		"lang": "painless",
		"inline": "ctx._source.age = params.age",
		"params": {
			"age": 100
		}
	}
}
```


### 删除
```
DELETE 27.0.0.1:9200/people/man/1
```


### 查询

#### 简单查询
```
GET 127.0.0.1:9200/people/man/1

# 检查文档是否存在
HEAD "http://localhost:9200/people/man/1"
```
---
- term不会分词
- match会分词
- bool 联合查询 看情况
- match_phrase可以查找分析字段
```
# 没有_search不会显示字段hits
post 127.0.0.1:9200/people/_search
{
    "query": {
        <!-- "match_all": {} -->
        <!-- match_phrase -->
        "match":{
          "name": "newcome"
        }
    },
    <!-- 只要name和age字段  -->
    "_source": [ "name", "age" ],
    "sort": [
    	{"data": {"order": "desc"}}
    ],
    "from":1,
    "size":5
}
```
---
```
# 根据 author title查询
post 127.0.0.1:9200/people/_search
{
    "query": {
      "match":{
          "query": "newcome",
          "fields": ["author", "title"]
        }
    }
}
```
---
and or
```
post 127.0.0.1:9200/people/_search
{
    "query": {
      "query_string":{
          "query": "李白 AND 杜普",
          "fields": ["author", "title"]
        }
    }
}
```
---
bool查询
- must	子句（查询）必须出现在匹配的文档中，并将有助于得分。
- filter	子句（查询）必须出现在匹配的文档中。 然而不像 must 此查询的分数将被忽略。
- should	子句（查询）应出现在匹配文档中。 在布尔查询中不包含 must 或 filter 子句，一个或多个should 子句必须有相匹配的文件。 匹配 should 条件的最小数目可通过设置minimum_should_match 参数。
- must_not	子句（查询）不能出现在匹配的文档中。

```
只关注结果是不是 结果会被缓存
{
  "query": {
    "bool": {
      "must": {
          "match": {
            "gender": "F"
          }
      },
      "filter": {
        "term": {
          "word_count": 1000
        }
      }
    }
  }
}
```

#### 结构化查询
匹配字数为1000的书
```
{
    "query": {
      "term": {
        "word_count": 1000;
      }
    }
}
```
匹配年龄在10到20之间
```
{
    "query": {
      "range": {
        "age": {
          "gte": 10,
          "lte": 20 
        }
      }
    }
}
```

#### 聚合查询
分别根据age和date聚合
```
{
	"aggs":{
		"group_by_word_count": {
			"terms":{
				"field": "age"
			}
		},
		"group_by_date": {
			"terms":{
				"field": "data"
			}
		}
	}
}
```

#### 组数据

```
POSY /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```

```
POST /website/blog/_mget
  "docs" : [
    { "_id" : 2 },
    # 单独请求覆盖值
    { "_type" : "pageviews", "_id" :   1 }
  ]
```
---

#### 深度分页
利用游标的方式
```
GET /old_index/_search?scroll=1m #时间被设置为一分钟
{
    "query": { "match_all": {}},
    "sort" : ["_doc"], 
    "size":  1000
}

GET /_search/scroll
{
    "scroll": "1m", 
    "scroll_id" : "cXVlcnlUaGVuRmV0Y2g7NTsxMDk5NDpkUmpiR2FjOFNhNnlCM1ZDMWpWYnRROzEwOTk1OmRSamJHYWM4U2E2eUIzVkMxalZidFE7MTA5OTM6ZFJqYkdhYzhTYTZ5QjNWQzFqVmJ0UTsxMTE5MDpBVUtwN2lxc1FLZV8yRGVjWlI2QUVBOzEwOTk2OmRSamJHYWM4U2E2eUIzVkMxalZidFE7MDs="
}
```

#### 代价较小的批量操作
```
curl -X POST "localhost:9200/_bulk" -H 'Content-Type: application/json' -d'
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
'
```

失败结果:
```
{
   "took": 3,
   "errors": true, 
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, 
            "error":    "DocumentAlreadyExistsException 
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 
      }}
   ]
}
```

复杂使用
```
curl -X POST "localhost:9200/website/log/_bulk" -H 'Content-Type: application/json' -d'
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
'
```

## 乐观并发控制
### 内部版本号
Elasticsearch 使用这个 _version 号来确保变更以正确顺序得到执行. 如果该版本不是当前版本号，我们的请求将会失败。

只有version = 1的时候才会执行

```
PUT /website/blog/1?version=1 
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}

```
### 外部版本号
外部版本号的处理方式和我们之前讨论的内部版本号的处理方式有些不同， Elasticsearch 不是检查当前 _version 和请求中指定的版本号是否相同， 而是检查当前 _version 是否 小于 指定的版本号。 如果请求成功，外部的版本号作为文档的新 _version 进行存储。


创建一个新的具有外部版本号 5 的博客文章
```
curl -X PUT "localhost:9200/website/blog/2?version=5&version_type=external" -H 'Content-Type: application/json' -d'
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
'
```

指定一个新的 version 号是 10 ：
```
PUT /website/blog/2?version=10&version_type=external
{
  "title": "My first external blog entry",
  "text":  "This is a piece of cake..."
}

```

请求成功并将当前 _version 设为 10 ：
```
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "2",
  "_version": 10,
  "created":  false
}
```

## 集群

### 分布式特性
Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在后台自动执行的操作
- 分配文档到不同的容器 或 分片 中，文档可以储存在一个或多个节点中
- 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
- 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
- 将集群中任一节点的请求路由到存有相关数据的节点
- 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

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

### 主分片和副本分片如何交互

我们可以发送请求到集群中的任一节点。 每个节点都有能力处理任意请求。 每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。 在下面的例子中，将所有的请求发送到 Node 1 ，我们将其称为 **协调节点(coordinating node)** 。

当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点。

### 新建、索引和删除单个文档
![流程]({{site.url}}/assets/image/interview/19.jpg)

- 客户端向 Node 1 发送新建、索引或者删除请求。
- 节点使用文档的 _id 确定文档属于分片 0 。请求会被转发到 Node 3，因为分片 0 的主分片目前被分配在 Node 3 上。
- Node 3 在主分片上面执行请求。如果成功了，它将请求并行转发到 Node 1 和 Node 2 的副本分片上。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。

在默认设置下，即使仅仅是在试图执行一个_写_操作之前，主分片都会要求 必须要有 _规定数量(quorum)_（或者换种说法，也即必须要有大多数）的分片副本处于活跃可用状态,默认为:
```
int( (primary + number_of_replicas) / 2 ) + 1
```

### 取回一个文档

![流程]({{site.url}}/assets/image/interview/20.png)

- 客户端向 Node 1 发送获取请求。

- 节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 Node 2 。

- Node 2 将文档返回给 Node 1 ，然后将文档返回给客户端。


### 更新一个文档

![流程]({{site.url}}/assets/image/interview/21.png)

- 客户端向 Node 1 发送更新请求。
- 它将请求转发到主分片所在的 Node 3 。
- Node 3 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 retry_on_conflict 次后放弃。
- 如果 Node 3 成功地更新文档，它将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， Node 3 向协调节点也返回成功，协调节点向客户端返回成功。

### 多文档模式

#### mget

![流程]({{site.url}}/assets/image/interview/22.png)

- 客户端向 Node 1 发送 mget 请求。
- Node 1 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复， Node 1 构建响应并将其返回给客户端。

#### bulk

![流程]({{site.url}}/assets/image/interview/23.png)

- 客户端向 Node 1 发送 bulk 请求。
- Node 1 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。
- 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

## java API
[github](https://github.com/almostlie/elasticDemo)