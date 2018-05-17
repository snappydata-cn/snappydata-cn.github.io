---
title: 客户端连接SnappyData
date: 2018-05-17 10:33:46
categories: 业务应用
---

### Snappy-SQL交互命令行

通过Apache Derby ij tool，SnappyData实现了一个交互式的命令行工具: **snappy**。通过此脚本便可在SnappyData集群中运行SQL命令或SQL脚本，此文件位于bin目录下。


在创建JDBC连接前，你可以通过**snappy.history**指定一个路径，来保留未来执行的所有SQL命令:

```
$ export JAVA_ARGS="-Dsnappy.history=/temp/snappydata-history.sql"
$ ./snappy
```

<!-- more -->

注意：这里选择snappy脚本和snappy-sql脚本都可以，snappy-sql最终也是要调用snappy脚本执行。

接下来就可以创建连接并执行SQL命令，需要注意的是所有命令都以分号结尾，并且不区分大小写。

通常我们指定一个locator hostname进行连接，默认端口1527.

```
$ ./bin/snappy
snappy> connect client '<locatorHostName>:1527';
```

指定locator后，locator会将连接请求根据负载情况发送到某个server节点并返回客户端已经连接的消息，客户端执行的所有sql命令都由这个server进行解析。


![](p1.png)

当然也可以执行show、describe等命令来查看其他信息，具体命令可通过**help**来查看帮助。

```
elapsedtime on;
show connections;
show members;
describe <Table_Name>;
...
```


### JDBC连接

JDBC连接到SnappyData 1.0.0(1.0.1)，需要引入如下依赖：

```
<!-- https://mvnrepository.com/artifact/io.snappydata/snappydata-store-client -->
<dependency>
  <groupId>io.snappydata</groupId>
  <artifactId>snappydata-store-client</artifactId>
  <version>1.6.0</version>
</dependency>

<dependency>
  <groupId>org.apache.thrift</groupId>     
  <artifactId>libthrift</artifactId>
  <version>0.9.2</version>
</dependency>

<dependency>
  <groupId>org.apache.spark</groupId>
  <artifactId>spark-unsafe_2.11</artifactId>
  <version>2.0.2</version>
</dependency>
```

Driver要选择**io.snappydata.jdbc.ClientDriver**，例如：

```
Class.forName("io.snappydata.jdbc.ClientDriver");
// Locator或Server默认的监听端口号为1527
Connection c = DriverManager.getConnection ("jdbc:snappydata://locatorHostName:1527/");

```

### 第三方客户端工具

为了便于开发测试，用户也可以选择一个第三方工具进行JDBC连接。一般情况下所有工具都没有自带SnappyData的Driver，所以用户需要手动将所需的依赖拷贝到lib目录或者通过配置添加，并指定Driver class为**io.snappydata.jdbc.ClientDriver**。

这里我们以“**[SQuirrel SQL](http://squirrel-sql.sourceforge.net/)**”的客户端工具为例，来说明配置的步骤。


#### 需要的依赖

依赖列表如下：

```
snappydata-store-client-1.6.0.jar
snappydata-store-shared-1.6.0.jar
gemfire-shared-1.6.0.jar
commons-io-2.5.jar
gemfire-trove-1.6.0.jar
slf4j-log4j12-1.7.21.jar
slf4j-api.1.7.21.jar
libthrift-0.9.2.jar
spark-unsafe_2.11-2.0.2.jar
spark-tags_2.11-2.0.2.jar
```

其中1-7都封装在snappydata-store-client中，8则属于libthrift，9-10封装在spark-unsafe_2.11中。


#### 界面配置

在**Drivers**界面，添加一个新的Driver，并将需要的依赖分别添加，**Class Name**项选择“io.snappydata.jdbc.ClientDriver”。

```
Driver Name: Apache SnappyData
Example URL: jdbc:snappydata://<locatorHostName>:<locatorClientPort>/
Website URL: https://snappydatainc.github.io/snappydata/howto/connect_using_jdbc_driver/

Class Name: io.snappydata.jdbc.ClientDriver
```

截图如下：

![](p2.png)


Driver配置完成后，开始配置URL连接，在**Aliases**界面，添加一个SnappyData Driver的URL连接。

![](p3.png)


连接成功后，执行查询：

![](p4.png)


### 总结

从使用角度来说，SnappyData完全可以当作MySQL使用，SQL语法是标准SQL+Spark SQL，支持分析函数，比MySQL功能还要强大。

从性能角度看，SnappyData的OLAP功能和MySQL没有可比性，OLAP功能强大。

从灵活度角度看，SnappyData支持自由的探索，AdHoc查询用起来非常灵活、快速、实时，这点是Kylin和Druid等预聚合系统无法达到的。

从易用性角度看，支持SQL，这对于分析人员来讲门槛降低，效率大大提升。

从业务角度看，支持update、delete、支持join，业务分析更灵活。



### 引用

[Snappy-SQL Shell Interactive Commands](https://snappydatainc.github.io/snappydata/reference/interactive_commands/store_command_reference/)

[How to Use Snappy SQL shell (snappy-sql)](https://snappydatainc.github.io/snappydata/howto/use_snappy_shell/)

[How to Connect using JDBC Driver](https://snappydatainc.github.io/snappydata/howto/connect_using_jdbc_driver/)










