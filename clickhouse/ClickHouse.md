# ClickHouse

## ClickHouse 有多快

1. ClickHouse 的速度？

   >在1亿数据集体量的情况下：
   >
   >	* Vertica 的 2.63倍
   >	* Greenplum的10倍
   >	* InfiniDB 的 17倍
   >	* MonetDB 的27倍
   >	* Hive的126倍
   >	* Mysql的429倍

2. ClickHouse为什么这么快？

   > 1. 列式存储与数据压缩（默认Lz4） -> 减少数据扫描范围，数据传输时的大小
   > 2. 向量化执行引擎（SIMD指令）-> CPU寄存器层面指令级并行
   > 3.  多线程分布式 -> 线程级并行
   > 4. 细致的算法选型和数据结构选型
   > 5. 持续改进，几乎每个月一个版本

## 字段类型

### 1. 基础类型

#### 1） 数值类型

##### I. Int（整数）

| 名称   | 大小（字节） | 范围                                     | 普遍观念          |
| ------ | ------------ | ---------------------------------------- | ----------------- |
| Int8   | 1            | -128~127                                 | Tinyint           |
| Int16  | 2            | -32768~32767                             | Smallint          |
| Int32  | 4            | -2147483648~21473647                     | Int               |
| Int64  | 8            | -9223372036854775808~9223372036854775807 | Bigint            |
| UInt8  | 1            | 0~255                                    | Tinyint Unsigned  |
| UInt16 | 2            | 0~65535                                  | Smallint Unsigned |
| Uint32 | 4            | 0~4294967295                             | Int Unsigned      |
| Uint64 | 8            | 0~18446744073709551615                   | Bigint Unsigned   |

##### II. Float（浮点数）

| 名称    | 大小（字节） | 有效精度 | 普遍概念 |
| ------- | ------------ | -------- | -------- |
| Float32 | 4            | 7        | Float    |
| Float64 | 8            | 16       | Double   |

##### III.Decimal（定点数）

| 名称          | 等效声明         | 范围                          |
| ------------- | ---------------- | ----------------------------- |
| Decimal32(S)  | Decimal(1~9,S)   | -1\*10^(9~S) 到 1\*10^(9~S)   |
| Decimal64(S)  | Decimal(10~18,S) | -1\*10^(18~S) 到 1\*10^(18~S) |
| Decimal128(S) | Decimal(19~38,S) | -1\*10^(38~S) 到 1\*10^(38~S) |

#### 2） 字符串类型

##### I. String

与编程语言中的String一样，可变长字符串。完全代替了传统数据库的是varchar,text,clob和blob等概念。

##### II. FixedString

类似于传统的Char，是一种定长字符串。与Char不同的是，FixedString使用null字节填充末尾字符，而Char使用空格填充。

```sql
:) SELECT
    toFixedString('abc', 5),
    Length(toFixedString('abc', 5)) AS Length

┌─toFixedString('abc', 5)─┬─Length─┐
│ abc                     │      5 │
└─────────────────────────┴────────┘
```

##### III. UUID

UUID是一种数据库常见的主键类型，在clickhouse中直接把他作为一种数据类型。UUID有32位，格式为8-4-4-4-12, UUID的默认值用0填充，00000000-0000-0000-0000-000000000000

#### 3） 时间类型

##### I. DateTime

精确到秒，支持使用字符串格式写入。支持 yyyy-MM-dd HH:mm:ss 和yyyy/MM/dd  HH:mm:ss, 从查询结果看，默认格式是 yyyy-MM-dd HH:mm:ss

```sql
CREATE TABLE DateTime_test
(
    `c1` DateTime
)
ENGINE = Memory

:) insert into DateTime_test (c1) values ('2021-09-01 00:00:00')
Ok
:) insert into DateTime_test (c1) values ('2021/09/01 00:00:00')
Ok
:) select * from DateTime_test;
┌──────────────────c1─┐
│ 2021-09-01 00:00:00 │
│ 2021-09-01 00:00:00 │
└─────────────────────┘
```

##### II. DateTime64

可以记录亚秒，在DateTime的基础上增加了精度设置。

```sql
CREATE TABLE DateTime64_test
(
    `c1` DateTime64(2)
)
ENGINE = Memory

:) insert into DateTime_test64(c1) values ('2021-09-01 00:00:00')
Ok

:) select * from DateTime_test;
┌─────────────────────c1─┐
│ 2021-09-01 00:00:00.00 │
└────────────────────────┘
```

##### III. Date

只精确到天，同样支持字符串写入，支持yyyy-MM-dd,yyyy/MM/dd，默认格式是yyyy-MM-dd

### 2. 复合类型

#### 1)  Array

Array有两种写法，Array(T)和[T],不需要指定T的类型，ClickHouse有数组类型推断能力, T可以是不同数据，比如 [1,2.0,3.00]，同一个Array中的类型需要相互兼容，不能是[1,'a']，如果存在null元素，则整个Array会被推断为Nullable.定义表字段为Array的时候需要明确元素类型。

#### 2）Tuple

tuple也有两种写法 tuple(T) 和(T)，与Array一样不用指定T的类型，T的类型也可以不兼容，但是定义表的时候需要明确每个位置元素的类型。

#### 3）Enum

ClickHouse支持枚举Enum8和Enum16两种类型，分别对应（String:Int8）和（String:Int16),枚举类型的Key和Value是非空且不允许重复的。

#### 4） Nested

嵌套类型，类似于Struct，嵌套类型只支持一层嵌套，即嵌套类型内部不允许再次出现嵌套类型。注意，嵌套模式为了兼顾一对多的设计，insert 的时候，嵌套类型的每一个值都应该是数组。

### 3. 特殊类型

#### 1）Nullable

Nullable严格来说不是一种独立的数据类型，更像是一种修饰符，与Java8的Optional对象类似。用来表示某个**基础数据类型**是否可以为空，比如 Nullable(Int8)

#### 2) Domain

域名类型分为IPv4和IPv6两种类型。IPv4基于UInt32类型封装，IPv6基于FixedString(16)封装。只有符合ip格式的字符串才能插入IP格式的数据。虽然看起来长得像字符串，但是不支持隐式类型转换，需要显式调用IPv4NumToString和IPv6NumToString才能得到字符串。

## DDL语句

#### 1. 数据库

##### 1）创建数据库

```sql
CREATE DATABASE [IF NOT EXISTS] db_name [ENGINE=Ordinary]
```

ClickHouse在创建数据库的时候也可以指定数据库引擎。

数据库目前一共支持5种类型的引擎

* Ordinary： 默认引擎，绝大多数都是默认，在此数据库下面可以使用任意的表引擎
* Dictionary：字典引擎，此数据库下会自动为所有数据字典创建它们的数据表
* Memory：内存引擎，用于存放临时数据，所有数据表只会停留在内存中，重启后会被清除
* Lazy： 日志引擎，只能使用Log系列的表引擎
* Mysql： Mysql引擎，会自动拉取远端mysql的数据。

##### 2） 重命名数据库

```sql
RENAME DATABASE old_name TO new_name
```

##### 3）删除数据库

```sql
DROP DATABASE [IF EXISTS] db_name
```

#### 2. 数据表

##### 1） 创建数据表

**一般建表语句**

```sql
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name (
    name1 <type> [DEFAULT|MATERIALIZED|ALIAS expr],
    name2 <type> [DEFAULT|MATERIALIZED|ALIAS expr],
    ....
)EGINE = egine
```

**复制已有表结构**

使用AS，支持不同数据库之间复制。

```sql
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name as [db_name.]table_name
```

**查询结果建表**

```sql
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name AS SELECT ...
```

##### 2）查看数据表

```sql
DESC [db_name.]table_name
```

##### 3） 临时表

```sql
CREATE TEMPORARY TABLE [IF NOT EXISTS] table_name (
    name1 <type> [DEFAULT|MATERIALIZED|ALIAS expr],
    name2 <type> [DEFAULT|MATERIALIZED|ALIAS expr],
    ...
)
```



clickhouse支持临时表，临时表有两个特点：

1. 生命周期与会话绑定，所以只支持Memory引擎。会话结束则临时表被销毁；
2. 不属于任何数据库，所以建表语句不指定库名。当临时表名跟普通表相同时，会优先查询临时表数据。

在日常使用clickhouse的时候，通常不会刻意使用临时表。

##### 4）分区表

```sql
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name(
    name1 <type> [DEFAULT|MATERIALIZED|ALIAS expr],
    name2 <type> [DEFAULT|MATERIALIZED|ALIAS expr],
    ...
    pt Date
)ENGINE=MergeTree()
PARTITION BY pt 
ORDER BY 
```

