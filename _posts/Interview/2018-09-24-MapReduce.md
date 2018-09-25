---
layout: post
title: hadoop-MapReduce
category: 学习
tags: 
description:
---

## 简介

“Map（映射）”和“Reduce（归纳）

## MapReduce 架构

### 流程
![MapReduce流程]({{site.url}}/assets/image/interview/16.jpg)

![MapReduce详细流程]({{site.url}}/assets/image/interview/17.png)

```
(input) <k1, v1> -> map -> <k2, v2> -> combine -> <k2, v2> -> reduce -> <k3, v3> (output)

eg:(input)<k1,v1> 可能是偏移量和文本 

(output)<k3, v3> 可能是单词与单词数和

```

### 核心概念

- Split: 交有由MapReduce处理的数据块,是MapReduce最小的计算单元
- blocksize: 是HDFS中的最小数据单元(128m),一般和Split一一对应

- InputFormat: 将我们的数据进行分片(split)

- OutputFormat: 输出

- Combiner

- Patitioner

### 1.x架构
![MapReduce详细流程]({{site.url}}/assets/image/interview/17.png)
#### JobTracker:JT
- 作业的管理者
- 将作业分解成一堆任务:Task(MapTask和ReduceTask)
- 将任务分派给TaskTracker运行
- 作业的监控 容错处理(eg:重启Task)
- 一定时间没收到TT信息,TT挂了,TT上运行的任务会指派到其他TT上运行

#### TaskTracker:TT
- 任务执行者
- 在TT上执行Task(MapTask和ReduceTask)
- 与JT进行交互: 执行/启动/停止作业,发送心跳给JT

#### MapTask
- 自己开发的任务交给Task执行
- 解析每条,交给自己的map方法方法
- 将map输出的结果写到本地磁盘(eg:有些作业只有map没有 reduce ==> HDFS)

#### ReduceTask
- 将Map Task的数据读取
- 数据分组传给自己编写的reduce方法处理
- 输出结果写到HDFS 

### 2.x架构
详见yarn

## WordCount功能之简单实现
```
mvn clean package -DskipTests

scp wordcount.jar root@122.152.194.171:~/lib/

hadoop jar /root/lib/wordcount.jar com.newcome.hadoop.mapreduce.WordCountApp hdfs://hadoop000:9000/hello.txt hdfs://hadoop000:9000/xw/
```