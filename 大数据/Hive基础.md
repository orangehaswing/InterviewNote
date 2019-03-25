# Hive基础

Hive是一个数据仓库基础工具在Hadoop中用来处理结构化数据。它架构在Hadoop之上，总归为大数据，并使得查询和分析方便。 

## 基本框架

### Hive 不是

- 一个关系数据库 
- 一个设计用于联机事务处理（OLTP） 
- 实时查询和行级更新的语言 

### Hiver特点

- 它存储架构在一个数据库中并处理数据到HDFS。
- 它是专为OLAP设计。 
- 它提供SQL类型语言查询叫HiveQL或HQL。
- 它是熟知，快速，可扩展和可扩展的。

### Hive架构 

下面的组件图描绘了Hive的结构：

![Hive Architecture](https://www.yiibai.com/uploads/allimg/141228/1-14122R10152108.jpg)

该组件图包含不同的单元。下表描述每个单元

- 用户接口/界面	

  Hive是一个数据仓库基础工具软件，可以创建用户和HDFS之间互动。用户界面，Hive支持是Hive的Web UI，Hive命令行，HiveHD洞察（在Windows服务器）。 

- 元存储	

  Hive选择各自的数据库服务器，用以储存表，数据库，列模式或元数据表，它们的数据类型和HDFS映射。

- HiveQL处理引擎	

  HiveQL类似于SQL的查询上Metastore模式信息。这是传统的方式进行MapReduce程序的替代品之一。相反，使用Java编写的MapReduce程序，可以编写为MapReduce工作，并处理它的查询。

- 执行引擎	

  HiveQL处理引擎和MapReduce的结合部分是由Hive执行引擎。执行引擎处理查询并产生结果和MapReduce的结果一样。它采用MapReduce方法。

- HDFS 或 HBASE	

  Hadoop的分布式文件系统或者HBASE数据存储技术是用于将数据存储到文件系统。 

### Hive工作原理 

下图描述了Hive 和Hadoop之间的工作流程。

![How Hive Works](https://www.yiibai.com/uploads/allimg/141228/1-14122R10220b9.jpg)



下表定义Hive和Hadoop框架的交互方式：

| Step No. |                    操作                    |
| :------: | :--------------------------------------: |
|    1     | Execute Query Hive接口，如命令行或Web UI发送查询驱动程序（任何数据库驱动程序，如JDBC，ODBC等）来执行 |
|    2     | Get Plan 在驱动程序帮助下查询编译器，分析查询检查语法和查询计划或查询的要求 |
|    3     | Get Metadata 编译器发送元数据请求到Metastore（任何数据库） |
|    4     |   Send Metadata Metastore发送元数据，以编译器的响应   |
|    5     | Send Plan 编译器检查要求，并重新发送计划给驱动程序。到此为止，查询解析和编译完成 |
|    6     |      Execute Plan 驱动程序发送的执行计划到执行引擎       |
|    7     | Execute Job 在内部，执行作业的过程是一个MapReduce工作。执行引擎发送作业给JobTracker，在名称节点并把它分配作业到TaskTracker，这是在数据节点。在这里，查询执行MapReduce工作 |
|   7.1    | Metadata Ops 与此同时，在执行时，执行引擎可以通过Metastore执行元数据操作 |
|    8     |       Fetch Result 执行引擎接收来自数据节点的结果       |
|    9     |      Send Results 执行引擎发送这些结果值给驱动程序       |
|    10    |      Send Results 驱动程序将结果发送给Hive接口       |

## 数据类型

Hive所有数据类型分为四种类型

- 列类型
- 文字
- Null 值
- 复杂类型

### 整型

整型数据可以指定使用整型数据类型，INT。当数据范围超过INT的范围，需要使用BIGINT，如果数据范围比INT小，使用SMALLINT。 TINYINT比SMALLINT小。

下表描述了各种INT数据类型：

|    类型    |  后缀  |  示例  |
| :------: | :--: | :--: |
| TINYINT  |  Y   | 10Y  |
| SMALLINT |  S   | 10S  |
|   INT    |  -   |  10  |
|  BIGINT  |  L   | 10L  |

### 字符串类型

字符串类型的数据类型可以使用单引号('')或双引号(“”)来指定。它包含两个数据类型：VARCHAR和CHAR。

下表描述了各种CHAR数据类型：

|  数据类型   |     长度     |
| :-----: | :--------: |
| VARCHAR | 1 to 65535 |
|  CHAR   |    255     |

### 时间戳

它支持传统的UNIX时间戳可选纳秒的精度。

它支持的java.sql.Timestamp格式“YYYY-MM-DD HH:MM:SS.fffffffff”和格式“YYYY-MM-DD HH:MM:ss.ffffffffff”。

### 日期

DATE值在年/月/日的格式形式描述 {{YYYY-MM-DD}}.

### 小数点

在Hive 小数类型与Java大十进制格式相同。它是用于表示不可改变任意精度。语法和示例如下：

```
DECIMAL(precision, scale)
decimal(10,0)

```

### 联合类型

联合是异类的数据类型的集合。可以使用联合创建的一个实例。语法和示例如下：

```
UNIONTYPE<int, double, array<string>, struct<a:int,b:string>>

{0:1} 
{1:2.0} 
{2:["three","four"]} 
{3:{"a":5,"b":"five"}} 
{2:["six","seven"]} 
{3:{"a":8,"b":"eight"}} 
{0:9} 
{1:10.0}

```

### 文字

#### 浮点类型

浮点类型是只不过有小数点的数字。通常，这种类型的数据组成DOUBLE数据类型。

#### 十进制类型

十进制数据类型是只不过浮点值范围比DOUBLE数据类型更大。

### Null 值

缺少值通过特殊值 - NULL表示。

### 复杂类型

Hive复杂数据类型如下：

### 数组

在Hive 数组与在Java中使用的方法相同。

```
Syntax: ARRAY<data_type>
```

### 映射

映射在Hive类似于Java的映射。

```
Syntax: MAP<primitive_type, data_type>
```

### 结构体

在Hive结构体类似于使用复杂的数据。

```
Syntax: STRUCT<col_name : data_type [COMMENT col_comment], ...>
```

Hive详细操作

[操作指南](https://www.yiibai.com/hive)















