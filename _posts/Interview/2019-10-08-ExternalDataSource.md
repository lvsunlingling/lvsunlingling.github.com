---
layout: post
title: Spark-External Data Source
category: study
tags: 
description:
---

## 写到文件

### 代码
```
    val spark = SparkSession.builder().appName("SparkSessionApp").master("local[2]").getOrCreate()
    val peopleDF = spark.read.json("file:///Users/newcome/development/devtmp/people")
    peopleDF.show()
    peopleDF.select("name").write.format("json").save("file:///Users/newcome/development/devtmp/outputjson")
    spark.stop()
```
### sparkSQL
```
CREATE TEMPORARY VIEW parquetTable
USING org.apache.spark.sql.parquet
OPTIONS (
  path ".Users/newcome/development/devtmp/people"
)
```

## 写到hive
spark.sql("select * from emp").write.saveAsTable("table1")
注意设置spark.sql.shuffle.patitions为200

## mysql
```scala
// Note: JDBC loading and saving can be achieved via either the load/save or jdbc methods
// Loading data from a JDBC source
val jdbcDF = spark.read
  .format("jdbc")
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .load()

val connectionProperties = new Properties()
connectionProperties.put("user", "username")
connectionProperties.put("password", "password")
val jdbcDF2 = spark.read
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties)

// Saving data to a JDBC source
jdbcDF.write
  .format("jdbc")
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .save()

jdbcDF2.write
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties)

// Specifying create table column data types on write
jdbcDF.write
  .option("createTableColumnTypes", "name CHAR(64), comments VARCHAR(1024)")
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties)
```


## 两个数据库下的数据操作
val resultDF = hiveDF.join(mysqlDF,hiveDF.col("name") === mysqlDF.col("name"),"inner").show()
