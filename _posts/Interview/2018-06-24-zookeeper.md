---
layout: post
title: hadoop-zookeeper
category: study
tags: 
description:
---

## 单机版安装

### jdk1.8
```
wget url
tar -xzvf filename
cp -r 文件夹名 /usr/jdk8/

vim /etc/profile
export JAVA_HOME=/usr/jdk8
export CLASSPATH=.:%JAVA_HOME%/lib/dt.jar:%JAVA_HOME%/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile 
```

### 下载zk

### zk的文件结构
- bin 运行目录
- conf 存放配置文件，我们需要修改zoo.cfg
- contrib zk附加的一些功能
- dist-maven maven编译后的目录
- docs 文档
- lib zk开发所需要的jar包
- recipes 官方提供案例
- src 源码

### 配置zookkeepr

```
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
```
- tickTime 时间单元，单独无意义，作为iniLimit、syncLimit属性的最小倍数
- iniLimit 用于集群，允许节点连接并同步到matser的最大初始化时间，以时间单元为倍数
- syncLimit 用于集群，master主节点和从节点之间发送消息，请求和应答的最大时间长度（心跳机制）
- dataDir zookeeper传输的文件数据（建议修改默认）
- dataLogDir:日志目录,可以不配，不配的话会与dataDir公用目录
- clientPort:服务器端口，默认2181

### 运行

```
cd /usr/local/zookeeper/bin/
./zkServer.sh 命令({start|start-foreground|stop|restart|status|upgrade|print-cmd}) #启动服务
./zkCli.sh #进入命令行
./zkCli.sh -server 192.168.0.4:2181
```

### 基本命令
- ls 查看目录列表
- get 查看当前节点信息
- ls2 ls + get


## zookeeper基本数据模型
- 树型结构
- 每一个节点称为znode，它可以有子节点，也可以有数据
- 分为永久节点和临时节点，临时节点在客户端断开后消失
- 每一个zk节点都会有自己的版本号
- 删除、修改版本号不匹配会报错（乐观锁）
- zk节点不宜过大，几k即可
- 可以设置acl，限制用户的访问

### 基本数据模型的操作

#### 增删查改
- create [路径] [数据] 创建节点
- create -e [路径] [数据] 创建临时节点
- create -s [路径] [数据] 创建永久节点
- set [路径] [数据] 
- set [路径] [数据] [版本号]
- delete [路径] 
- delete [版本号]

#### watcher监听事件
可以用于在多台文件上改配置
- stat [路径] watch 创建结点
- get  [路径] wacth 改和删
- ls   [路径] watch 子节点的增和删

#### 权限
权限下每一个条件都得满足
- addauth digest liukang:123 添加权限(登录出去后就没了)
- getAcl [路径]
- setAcl [路径] world:anyone:cdrwa（创建：读取：删除：写：管理员）删除、创建权限仅对当前节点的孩子节点有效
- setAcl [路径] auth:liukang:123:cdrwa（创建：读取：删除：写：管理员）删除、创建权限仅对当前节点的孩子节点有效
- setAcl [路径] digest:liukang:3b8aCcKHzHba/IKR8jVTGOllbJI:cdrwa
- setAcl [路径] ip:192.168.0.1:cdrwa

超级管理员权限设置
```
vim ./zkServer.sh
nohup "$JAVA" "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "-Dzookeeper.DigestAuthenticationProvider.superDigest=liukang:3b8aCcKHzHba/IKR8jVTGOllbJI=" \
./zkCli.sh #进入命令行
addauth digest super:admin
```

### 四字命令

```
echo stat | nc 192.168.0.5 2181 #显示当前服务器状态

echo ruok | nc localhost 2181 # are you ok?

echo dump | nc localhost 2181 #显示会话和临时节点

echo conf | nc localhost 2181 #显示服务器有关配置

echo cons | nc localhost 2181 #显示连接到服务器的相关信息

echo envi | nc localhost 2181 #显示相关环境变量

echo mntr | nc localhost 2181 #显示健康信息

echo wchs | nc localhost 2181 #显示watcher

4lw.commands.whitelist=* #添加白名单

echo wchc | nc localhost 2181 #显示session与watch

echo wchp | nc localhost 2181 #显示所有path与watch
```

## 集群配置
```
vim ./conf/zoo.cfg
clientPort=2181
server.1=localhost:2888:3888 #ip：集群端口：竞争端口
server.2=localhost:2889:3889
server.3=localhost:2890:3890


touch ./dataDir/myid
vim myid
1
```
```
vim ./conf/zoo.cfg
clientPort=2182
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890


touch ./dataDir/myid
vim myid
2
```
```
vim ./conf/zoo.cfg
clientPort=2183
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890


touch ./dataDir/myid
vim myid
3
```