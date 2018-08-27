---
layout: post
title: java-并发
category: 学习
tags: 
description:
---

# 线程状态的转换
![线程状态]({{site.url}}/assets/image/interview/9.jpg)

## 新建（New）

新创建了一个线程对象。,尚

## 可运行（Runnable）

线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取cpu 的使用权 

## 运行 (running)

可运行状态(runnable)的线程获得了cpu 时间片（timeslice） ，执行程序代码。

## 阻塞(（block）

### 等待阻塞
运行(running)的线程执行o.wait()方法，JVM会把该线程放入等待队列(waitting queue)中。

### 同步阻塞
运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池(lock pool)中。

### 其他阻塞
运行(running)的线程执行Thread.sleep(long ms)或t.join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入可运行(runnable)状态。

## 死亡
线程run()、main() 方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。
