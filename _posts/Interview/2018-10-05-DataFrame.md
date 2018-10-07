---
layout: post
title: Spark-DataFrame
category: study
tags: 
description:
---

## Introduction
- A distributed collection of rows organized into named columns(RDD with schema)
- An abstraction for selecting ,filtering,aggregation and plotting structured data

## DataFrame AND RDD
![Spark]({{site.url}}/assets/image/interview/31.png)

## api
```scala
vim people
{"name": "newcome","age":12}
{"name": "xiaoming","age":12}

  def main(args: Array[String]): Unit = {

    val spark = SparkSession.builder().appName("DaTaFramApp").master("local[2]").getOrCreate()

    val peopleDF = spark.read.format("json").load("file:///Users/newcome/development/devtmp/people")

    peopleDF.printSchema()

    //默认显示前二十条数据,默认显示不完全数据
    peopleDF.show(30,false)

    //返回前10条记录
    peopleDF.take(10)
    //第一条记录
    peopleDF.first()

    //查询name数据
    peopleDF.select("name").show()
    //多列查询
    peopleDF.select(peopleDF.col("name"),(peopleDF.col("age") + 10).as("age2"))
    //过滤
    peopleDF.filter(peopleDF.col("age") > 19).show()
    //聚合
    peopleDF.groupBy("age").count().show()

    val peopleDF2 = spark.read.format("json").load("file:///Users/newcome/development/devtmp/people")
    //组合
    peopleDF.join(peopleDF2,peopleDF.col("name") === peopleDF2.col("name"),"inner").show()


    spark.stop()
  }
```

## RDD way

```scala
vim info.txt
1,xiaoming,60
2,lisi,22
3,mayun,13

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("SparkSessionRDDApp").master("local[2]").getOrCreate()

    programe(spark)

    spark.stop()
  }

  //代理方式
  def inferReflection(spark: SparkSession): Unit = {
    val rdd = spark.sparkContext.textFile("file:///Users/newcome/development/devtmp/info.txt")

    import  spark.implicits._
    val infoDF = rdd.map(_.split(",")).map(line => Info(line(0).toInt, line(1), line(2).toInt)).toDF()


    infoDF.show()

    infoDF.createOrReplaceTempView("infos")
    spark.sql("select * from infos where age > 30")
  }

  //编程方式
  def programe(spark: SparkSession): Unit = {
    val rdd = spark.sparkContext.textFile("file:///Users/newcome/development/devtmp/info.txt")

    val infoRDD = rdd.map(_.split(",")).map(line => Row(line(0).toInt, line(1), line(2).toInt))

    val structType = StructType(Array(StructField("id",IntegerType, true),
      StructField("name", StringType, true),
      StructField("id", IntegerType, true)))

    val infoDF = spark.createDataFrame(infoRDD,structType)

    infoDF.show()
  }

  case class Info(id: Int, name: String, age: Int)
```

## Dataset
![Spark]({{site.url}}/assets/image/interview/32.png)
```scala
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("SparkSessionRDDApp").master("local[2]").getOrCreate()

    import spark.implicits._
    val path = ""

    val df = spark.read.option("header","true").option("inferSchema","true").csv(path)

    df.show()

    val ds = df.as[Sales]

    ds.map(line => line.itemId).show()

    spark.stop()

  }

  case class Sales(transactionId: Int, customerId:Int, itemId:Int, amountPaid:Double)
```

