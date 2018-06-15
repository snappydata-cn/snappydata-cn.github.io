---
title: SnappyData中SQL新功能:update join与put into
date: 2018-06-15 15:48:22
categories: 业务应用
---


## update点更新

**Spark**中的**RDD具有不可变性**，但是**SnappyData**中数据是**可变的**，即使是列表(column table)，也是支持更新的。

对于实时的OLAP来讲，这是非常重要的功能，比如订单的状态、剩余的库存等，都是需要实时update操作的。

有些业务也许是纯update只保留最新记录，而有些业务update旧数据的状态，也要insert新数据，以便分析历史变化趋势。

这种复杂的业务数据如果放到druid中，让分析变得非常困难。而SnappyData支持各种类型的update和insert操作，让分析复杂的业务数据变得更简单。

<!-- more -->

SnappyData中标准update语法如下：

```
UPDATE TRADE.CUSTOMERS SET ADDR=NULL, SINCE=NULL  WHERE CID > 10;

UPDATE TRADE.CUSTOMERS SET ADDR = 'Snappydata' WHERE ADDR IS NULL;

UPDATE TRADE.SELLORDERS SET QTY = QTY+10;

UPDATE TRADE.SELLORDERS SET STATUS = DEFAULT  WHERE CID = 10;
```

我们实际生产中，在开始阶段是按照这种点更新操作的，但是由于并发很多，数量庞大，导致lead的压力较大，所以将部分业务逐渐的由点更新变为了批量更新。即降低部分数据的实时性(10秒级)，换取更大的吞吐量(数万级)。


## update join -- 批量更新

SnappyData也支持**批量更新**，总体来说有2种形式，第一种是更新为固定的值，第二种是更新为另一个表中的值。

### 批量更新为固定的值

SnappyData中2个列表join，你可以将一个表中某些列的值，批量更新为固定的值，SQL如下：

```
UPDATE a
SET a.col1 = 60,
    a.col2 = 'xxx'
WHERE EXISTS
    (SELECT 1 FROM b WHERE a.id = b.id) 
AND a.name = 'xx';
```

在实际生产中，我们找了其中某个批量update语句，数量分别为9.2亿条和4.5万条记录的2个列表的join，匹配量3.6万，其执行时间为0.6秒，执行计划如下：

![](batch_update_1.png)


### 批量更新为另一个表中的值

第二种是更新为另一个表中的值，SnappyData中提供了[update join](https://github.com/SnappyDataInc/snappydata/pull/906)的功能，标准SQL如下：

```
UPDATE TRADE.SELLORDERS
SET A.TID = B.TID
FROM TRADE.SELLORDERS A
JOIN TRADE.CUSTOMERS B ON A.CID = B.CID;
```

我们在生产中也有这类的需求，1个7亿条和300万的列表join，耗时0.2秒，执行计划如下：

![](batch_update_2.png)


可以看到，在本地join的情况下，速度在毫秒级内完成。



## put into -- replace操作

SnappyData中还有一种功能，类似于merge、upsert或MySQL中的replace into功能。即2个表join，如果join key存在，则进行update(delete && insert)，否则insert。

我们只以column table为例说明。列表如果需要put into的功能，则在DDL时必须明确指定**KEY_COLUMNS**属性（可以是组合列），即唯一标识一行数据，否则执行时报错。

DDL中指定KEY_COLUMNS的例子如下：

```
create table XX(
 order_id bigint,
 goods_id bigint,
 order_status integer,
 dateKey integer,
 XX
 )
 using column options(
 partition_by 'goods_id',
 colocate_with 'XXX',
 redundancy '0',
 overflow 'true',
 buckets '128',
 KEY_COLUMNS 'goods_id,order_id,datekey'
 );
```

put into的标准SQL如下：

```
PUT INTO NEWEMPLOYEES 
SELECT * 
from EMPLOYEES 
WHERE C_NAME='User 1'
```

我们实际也用到了put into语法，执行计划如下：

![](put_into.png)


## 总结

SnappyData支持历史数据的更新，这个功能极大的提高了分析的灵活性和实时性。

同时支持了点更新以及各种批量的更新和替换，使得用户可以在低延迟和高吞吐之间做个平衡，按需实现。



## 参考

[update](https://snappydatainc.github.io/snappydata/reference/sql_reference/update/)

[update join](https://github.com/SnappyDataInc/snappydata/pull/906)

[put into](https://snappydatainc.github.io/snappydata/reference/sql_reference/put-into/)

[create table](https://snappydatainc.github.io/snappydata/reference/sql_reference/create-table/)