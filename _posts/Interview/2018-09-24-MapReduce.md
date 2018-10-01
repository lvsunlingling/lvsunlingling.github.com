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
- Combiner : Combiner是一个“迷你reduce”过程，它只处理单台机器生成的数据。(只能使用求和,次数)不能使用平均数

- Patitioner 对map作hash,交由不同的的job处理
![Combiner AND Patitioner]({{site.url}}/assets/image/interview/24.jpg)

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

### 代码

[javaDemo](https://github.com/almostlie/hadooptrain)

### 打包
```
mvn clean package -DskipTests

scp wordcount.jar root@122.152.194.171:~/lib/

/Users/newcome/development/env/hadoop-2.6.0-cdh5.7.0/bin/hadoop jar ~/Desktop/wordCount.jar com.newcome.hadoop.mapreduce.WordCountApp hdfs://newcome:8020/i.txt hdfs://newcome:8020/xw/

/Users/newcome/development/env/hadoop-2.6.0-cdh5.7.0/bin/hadoop jar ~/Desktop/demo.jar com.newcome.hadoop.mapreduce.PhoneCountApp hdfs://newcome:8020/phone.txt hdfs://newcome:8020/phone/
```

### 配置jobHistory

vim mapred-site.xml
```
<!-- 设置jobhistoryserver 没有配置的话 history入口不可用 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>newcome:10020</value>
</property>

<!-- 配置web端口 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>newcome:19888</value>
</property>

<!-- 配置正在运行中的日志在hdfs上的存放路径 -->
<property>
    <name>mapreduce.jobhistory.intermediate-done-dir</name>
    <value>/history/done_intermediate</value>
</property>

<!-- 配置运行过的日志存放在hdfs上的存放路径 -->
<property>
    <name>mapreduce.jobhistory.done-dir</name>
    <value>/history/done</value>
</property>


```

vim yarn-site.xml
```
<!-- 开启日志聚合 -->
<property>
<name>yarn.log-aggregation-enable</name>
<value>true</value>
</property>
```

```
#启动： 在hadoop/sbin/目录下执行  
./mr-jobhistory-daemon.sh start historyserver
#停止：在hadoop/sbin/目录下执行  
./mr-jobhistory-daemon.sh stop historyserver
```