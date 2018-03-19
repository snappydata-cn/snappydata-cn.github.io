---
layout: blog
title: SnappyData与TiDB,Spark,Flink的对比-1
date: 2018-03-19 16:55:31
categories: 对比
---

### SnappyData是什么？

**SnappyData**是一个开源的内存分布式存储与计算引擎，提供实时的、HTAP(OLTP+OLAP)场景的解决方案，融合了Apache Spark与GemFire数据库，以多种数据模型提供复杂的、实时的、多维度的OLAP分析，完全支持标准SQL与Spark SQL。

分析人员只需通过SQL便可对实时数据进行低延迟且高准确性的分析工作。


### SnappyData的特性

```
1、分布式存储+计算引擎
2、完全基于内存
3、融合了Gemfire与Apache Spark的特性
4、支持行存，且支持列存(压缩)
5、对行存与列存都支持DML操作
6、区分开源版本与闭源版本(支持off-heap与AQP功能)
7、存储时可指定关联关系，使得数据本地化(colocate)，多表join性能是Spark的20倍+
8、完全兼容Spark，支持标准SQL与Spark SQL
9、对实时数据的处理只需用标准SQL或Spark SQL即可，同时由于其存储明细数据，使得对实时数据的处理既支持乱序又支持Retraction，非常适合ad-hoc类查询
```


###  与TiDB对比

```
TiDB是基于KV的行存；SnappyData支持列存。
TiDB更适合OLTP事务；SnappyData更适合OLAP分析。
TiDB存储于RocksDB；SnappyData完全内存，且支持overflow。
TiDB支持DQL、DML与DDL；SnappyData也支持DQL、DML与DDL。
TiDB支持分布式的多副本；SnappyData也支持多副本。
TiSpark基于Spark处理复杂SQL；SnappyData存储采用与Spark一样的存储格式，省去加载的时间，同时支持colocate，使得join时性能提升2个级别
```

###  SnappyData相比于Spark的优势


```
Spark的RDD不可变；SnappyData可变(Gemfire)，因此SnappyData的数据支持DML
Spark需要从外部数据源加载数据；SnappyData不需要加载，存储格式与Spark的DataFrame一致
Spark计算时基于内存；SnappyData计算转换为Spark Job，也是基于内存(不是所有的SQL都走Spark Job)
Spark对于多表join，涉及跨机器join，性能受限；SnappyData存储支持表间colocate，join发生在机器本地，性能提高
```

###  SnappyData相比于Flink的优势


```
Flink流处理的API是DataStream，学习成本较高；SnappyData API支持标准SQL与Spark SQL，学习成本较低。
Flink的Stream SQL目前支持功能有限，尤其是流表join与双流join
Flink在单流的汇总功能强大，双流join欠缺准确性；SnappyData支持多个表的join，且准确性极高
Flink的处理延时极低(毫秒级)；SnappyData处理延迟同样极低(毫秒级)
Flink处理一个需求就需要单独一个job；SnappyData简单很多，一个SQL对应一个需求，效率极高
Flink支持乱序及Retraction，但是已经发出的结果不能更改；SnappyData由于存储明细数据，因此也支持乱序与Retraction，同时对于已经发出的结果，同样可以更改(客户端只需再次调用一遍SQL即可)
Flink实时处理不能很好的关联大量的历史数据；SnappyData随意关联历史数据，通过SQL不但可以查询实时数据，同样可查询历史数据
```

###  SnappyData的劣势

```
1、尽管列存支持压缩，从而大幅度减少数据量，同时可以横向扩展，但是内存空间的使用仍然使得成本较高
2、存储+计算都在内存进行，且Spark Job的运行基于JVM，要注意防止内存不足的情况出现
3、社区不够活跃
4、对于PB及以上的数据规模，即使横向扩展也不太适合这种规模
```
