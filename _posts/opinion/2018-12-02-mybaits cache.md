---
layout: post
title: mybaits cache
category: opinion
tags: 
description:
---

## concept

> 在开发中,我们经常用Hibernate(jpa)和Mybaits等orm框架帮助我们迅速进行增删查改等操作,像mybaits3和Hibernate的一级缓存是默认开启,很容易造成开发上的一些问题,比如引起脏数据等问题

MyBatis的缓存分为一级缓存和二级缓存，两种缓存的缓存粒度是一样的，都是对应一条sql查询语句，但是二者的生命周期是不一样的，一级缓存的生命周期是SqlSession对象的使用期间，随着SqlSession对象的死亡而消失；二级缓存的生命周期是同MyBatis应用一样长

### Sqlsession

> 对应着一次数据库会话,使用门面模式,每一个SqlSession都拥有一个新的Executor对象
- 系统加载时 SqlSessionFactoryBuilder -> DefaultSqlSessionFactory 
- 一次会话时候 DefaultSqlSessionFactory-> DefaultSqlsession

### Executor 

> mybatis内真正起作用对象.
![]({{site.url}}/assets/image/opinion/4.png)

- SimpleExecutor
```
    @Override
    public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
        Statement stmt = null;
        try {
            //获取配置
            Configuration configuration = ms.getConfiguration();
            //新创建一个statementHandler
            StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
            //准备statement
            stmt = prepareStatement(handler, ms.getStatementLog());
            //执行正真的查询方法
            return handler.<E>query(stmt, resultHandler);
        } finally {
            closeStatement(stmt);
        }
    }
```
- ResuseExecutor

> ReuseExecutor的实现其实和SimpleExecutor的类似，只不过内部维护了一个map来缓存statement,不缓存结果

- BatchExecutor

> BatchExecutor 主要是把不同的Statement以及参数值缓存起来

### Cache

Cache： MyBatis中的Cache接口，提供了和缓存相关的最基本的操作.使用装饰器模式
![]({{site.url}}/assets/image/opinion/5.jpg)

PerpetualCache
```java
  private Map<Object, Object> cache = new HashMap<Object, Object>();
```

## 一级缓存

![]({{site.url}}/assets/image/opinion/1.jpg)

![]({{site.url}}/assets/image/opinion/2.jpg)

![]({{site.url}}/assets/image/opinion/3.jpg)

BaseExcuter.java
```java
  protected ConcurrentLinkedQueue<DeferredLoad> deferredLoads;
  protected PerpetualCache localCache;

  @Override
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    BoundSql boundSql = ms.getBoundSql(parameter);
    //根据MappedStatement,parameter,RowBounds(limit)生成CacheKey来盘判断是否同一查询
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
 }

  @SuppressWarnings("unchecked")
  @Override
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    //查询未重叠 或 配置文件要求不使用缓存的时候
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
      clearLocalCache();
    }
    List<E> list;
    try {
      //增加一次查询栈
      queryStack++;
      //从缓存中查找
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        //处理存储过程
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        //从数据库执行查找
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    } finally {
      queryStack--;
    }
    //
    if (queryStack == 0) {
      for (DeferredLoad deferredLoad : deferredLoads) {
        //查询队列中的结果保存到缓存
        deferredLoad.load();
      }
      //关闭并发队列
      deferredLoads.clear();
      //缓存级别为 statement 清空缓存
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        clearLocalCache();
      }
    }
    return list;
  }

```

## 总结

- MyBatis一级缓存的生命周期和SqlSession一致。

- MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。

- MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。

## 二级缓存

> MyBatis二级缓存的工作流程和前文提到的一级缓存类似，只是在一级缓存处理前，用CachingExecutor装饰了BaseExecutor的子类，在委托具体职责给delegate之前，实现了二级缓存的查询和写入功能
![]({{site.url}}/assets/image/opinion/7.png)

### 配置

```
<setting name="cacheEnabled"value="true"/>
```

### Executor 

![]({{site.url}}/assets/image/opinion/6.png)

### 核心代码

```java
  private final Executor delegate;
  //二级缓存
  private final TransactionalCacheManager tcm = new TransactionalCacheManager();

  @Override
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    //查询缓存是否初始化
    Cache cache = ms.getCache();
    if (cache != null) {
      //使用MappedStatement条件刷新缓存
      flushCacheIfRequired(ms);
      //查询是否开启二级缓存
      if (ms.isUseCache() && resultHandler == null) {
          //处理存储过程
        ensureNoOutParams(ms, parameterObject, boundSql);
        //命中直接返回
        List<E> list = (List<E>) tcm.getObject(cache, key);
        //未命中
        if (list == null) {
         //执行一级缓存的代码逻辑
          list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          tcm.putObject(cache, key, list); 
        }
        return list;
      }
    }
    //执行一级缓存的代码逻辑
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }

  private void flushCacheIfRequired(MappedStatement ms) {
    Cache cache = ms.getCache();
      if (cache != null && ms.isFlushCacheRequired()) {      
       tcm.clear(cache);
      }
  }

```


### Hibernate

> Hibernate一级缓存是内置的，不能被卸载（不能被卸载的意思就是这种缓存不具有可选性，必须有的功能，不可以取消Session缓存，也是Hibernate缓存机制默认的缓存。在一级缓存中，持久化类的每个实例都具有唯一的OID。

## mybaits默认SESSION缓存带来的问题
### 列举

- 分表情况下,采用相同的查询语句及其查询条件，所以在到达分表逻辑之前就已经命中缓存，提前返回,造成数据不一致
- 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据

### 解决方案

- 只把mybaits当orm框架使用,关闭一级缓存

```
<!-- 关闭懒加载-->
<setting name="lazyLoadingEnabled" value="false" />
<!-- 关闭延迟加载-->
<setting name="aggressiveLazyLoading" value="false" />
<!-- 缓存作用域为STATEMENT-->
<setting name="localCacheScope" value="STATEMENT" />
```

- 读与写都放在同一个锁里面即可。例如我从读数据开始即上锁，然后更新或者插入的实务提交之后，才释放锁，别的线程才可以做读写操作
- mybatis cache plugin