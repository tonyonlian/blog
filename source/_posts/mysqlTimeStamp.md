---
title: 关于mysql的timestamp类型的使用及问题
date: 2019-05-31 10:46:00
tags: mongodb
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在发布版本的时候，脚本服务程序报出一个异常：java.sql.SQLException: Value '0000-00-00 00:00:00' can not be represented as java.sql.Timestamp。看到这个异常后心里想这是什么鬼呢？在测试环境上跑的好好的程序，发到生产环境就开始搞事情了，真是不让人省心的代码呀。
 
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;快速定位到源程序的出错行，我call，看到的却只是一行用了无数变得SQL查询。查询函数肯定没问题，因为这个查询函数已经被用的烂熟于心了。现在我敢打保票，我们团体的每个伙伴们闭上眼都能写对这个sql的查询函数。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;剩下的就只能怀疑这个超长的SQL查询语句了。赶快把SQL语句格式化一下，细细查看SQL的每个部分：where条件，排序，子查询，连接查询等等。查看完一遍后也没看出端倪。现在只能是怀疑数据库存有脏数据了。 错误异常是关于时间戳转换的问题。那么也就能定位脏数据是出现在查询所涉及的表的timestamp字段中。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询出所涉及的数据库表，逐条查看数据库表中的timestamp字段。字段的数据是null或时间数据，感觉没错呀。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;把所有的怀疑都顺了一遍，没发现异常。这下傻了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;唉，只能是问问搜索引擎君了，往搜索输入框中一甩异常，飞速回车。我的天呢，铺天盖地的搜索结果。这下心里大喜，这个问题是个被大家遇到常见问题。肯定有解决方案。


**快速点开搜索结果，开始涨姿势了。。。。。。**
      
请出我们今天的主角——mysql的timetamp类型

#### 1. MySQL的TIMESTAMP类型介绍

- MySQL的 TIMESTAMP是一种保存日期和时间组合的时间数据类型。

- TIMESTAMP列的格式为YYYY-MM-DD HH:MM:SS，固定为19个字符。

- TIMESTAMP值的范围从'1970-01-01 00:00:01' UTC到'2038-01-19 03:14:07' UTC。（UTC是零时区标准时间）
- DATETIM和TIMESTAMP类型所占的存储空间不同，前者8个字节，后者4个字节，这样造成的后果是两者能表示的时间范围不同。前者范围为1000-01-01 00:00:00 ~ 9999-12-31 23:59:59，后者范围为'1970-01-01 00:00:01' UTC到'2038-01-19 03:14:07' UTC。所以可以看到TIMESTAMP支持的范围比DATATIME要小,容易出现超出的情况.

- 当您将TIMESTAMP值插入到表中时，MySQL会将其从连接的时区转换为UTC后进行存储。当您查询TIMESTAMP值时，MySQL会将UTC值转换回连接的时区。请注意，对于其他时间数据类型(如DATETIME)，不会进行此转换。当检索由不同时区中的客户端插入TIMESTAMP值时，将获得存储数据库中不同的值。 只要不更改时区，就可以获得与存储的相同的TIMESTAMP值。

timestamp的时区变换是怎么整的？小哥上个示例给大家看看哈



*MySQL TIMESTAMP时区示例*

1.1. 创建一个名为test_timestamp的新表，该表具有一列：t1，其数据类型为TIMESTAMP;


```sql
USE mytestdb;
CREATE TABLE IF NOT EXISTS test_timestamp (
    t1  TIMESTAMP
);
```

1.2. 使用SET time_zone语句将时区设置为"+00：00"UTC

```sql
SET time_zone='+00:00';

```
1.3. 将TIMESTAMP值插入到test_timestamp表中。

```sql
INSERT INTO test_timestamp 
VALUES('2018-09-06 00:00:01');
```

1.4. 从test_timestamp表中查询选择TIMESTAMP值。

```sql
SELECT 
    t1
FROM
    test_timestamp;

+---------------------+
| t1                  |
+---------------------+
| 2018-09-06 00:00:01 |
+---------------------+
1 row in set
```
1.5. 将会话时区设置为不同的时区，以查看从数据库服务器返回的值：

```sql
SET time_zone ='+03:00';

SELECT t1
FROM test_timestamp;

//执行结果
+---------------------+
| t1                  |
+---------------------+
| 2018-09-06 03:00:01 |
+---------------------+
1 row in set

```

#### 2. 将TIMESTAMP列的自动初始化和更新


2.1. 在创建新记录和修改现有记录的时候都对这个数据列刷新：

```sql

TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

```
2.2. 在创建新记录的时候把这个字段设置为当前时间，但以后修改时，不再刷新它：

```sql

TIMESTAMP DEFAULT CURRENT_TIMESTAMP

```
2.3. 在创建新记录的时候把这个字段设置为0，以后修改时刷新它：

```sql

TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

```
2.4. 在创建新记录的时候把这个字段设置为给定值，以后修改时刷新它：

```sql

TIMESTAMP DEFAULT ‘yyyy-mm-dd hh:mm:ss' ON UPDATE CURRENT_TIMESTAMP

```

2.5. 修改timestamp类型的默认属性

```sql

ALTER TABLE `table` MODIFY collumn_1 TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL;

```

TIMESTAMP列的自动初始化和更新与mysql的explicit_defaults_for_timestamp参数设置有关：

> explicit_defaults_for_timestamp设置为OFF:创建timestamp类型字段是，不设置默认值，则会自带 timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP属性

```sql
//自动默认的例子：

mysql> create table test(id int,hiredate timestamp);
Query OK, 0 rows affected (0.01 sec)

//导出建表sql

mysql> show create table test\G
*************************** 1. row ***************************
       Table: test
Create Table: CREATE TABLE `test` (
  `id` int(11) DEFAULT NULL,
  `hiredate` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)

```

> explicit_defaults_for_timestamp设置为ON:会关闭默认的自动初始与更新属性

如果想在explicit_defaults_for_timestamp设置为OFF时想关闭的自动初始与更新属性，有两种方式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1> 用DEFAULT子句该该列指定一个默认值
   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2> 为该列指定NULL属性。

*注意：在MySQL 5.6.5版本之前，Automatic Initialization and Updating只适用于TIMESTAMP，而且一张表中，最多允许一个TIMESTAMP字段采用该特性。从MySQL 5.6.5开始，Automatic Initialization and Updating同时适用于TIMESTAMP和DATETIME，且不限制数量。*


#### 3. 时间戳timestamp字段插入0000-00-00 00:00:00

在生产数据库可以给timestamp类型字段插入1970-01-01 08:00:00 （utc+08），然后在测试数据timestamp类型字段插入1970-01-01 08:00:00 （utc+08）时报错Incorrect datetime value: '1970-01-01 08:00:00' for column 'pause_time' at row 1。

首先解释一下 1970-01-01 08:00:00（utc+08)日期为东8区的0日期，也就是毫秒的开始时间，对应0时区的日期为 1970-01-01 00:00:00（utc） 在mysq的timestamp称为0000-00-00 00:00:00 也称为0日期。

*注意：从5.6.17这个版本就默认设置了不允许插入 0 日期了 术语是 NO_ZERO_IN_DATE  NO_ZERO_DATE*

如果在5.6.1版本及版本后想插入0日期也是可以的，这需要在mysql的配置文件里修改sql_mode，然后重启服务

```conf
[mysqld]
#set the SQL mode to strict
#sql-mode="modes..." 
sql-mode = "STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

```
查看sql-mode设置值的命令

```sql
SHOW VARIABLES LIKE 'sql_mode';
```

关于mysql的sql_mode可参考文档[sql_mode文档](https://dev.mysql.com/doc/refman/5.6/en/sql-mode.html#sqlmode_no_zero_date)

sql_mode常用值

- ONLY_FULL_GROUP_BY：对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中



- NO_AUTO_VALUE_ON_ZERO：该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。如果用户 希望插入的值为0，而该列又是自增长的，那么这个选项就有用了。



- STRICT_TRANS_TABLES：在该模式下，如果一个值不能插入到一个事务表中，则中断当前的操作，对非事务表不做限制



- NO_ZERO_IN_DATE：在严格模式下，不允许日期和月份为零



- NO_ZERO_DATE：设置该值，mysql数据库不允许插入零日期，插入零日期会抛出错误而不是警告。



- ERROR_FOR_DIVISION_BY_ZERO：在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如 果未给出该模式，那么数据被零除时MySQL返回NULL



- NO_AUTO_CREATE_USER：禁止GRANT创建密码为空的用户



- NO_ENGINE_SUBSTITUTION：如果需要的存储引擎被禁用或未编译，那么抛出错误。不设置此值时，用默认的存储引擎替代，并抛出一个异常



- PIPES_AS_CONCAT：将"||"视为字符串的连接操作符而非或运算符，这和Oracle数据库是一样的，也和字符串的拼接函数Concat相类似



- ANSI_QUOTES：启用ANSI_QUOTES后，不能用双引号来引用字符串，因为它被解释为识别符


#### 4. 对于我们碰到异常解决方法

java.sql.SQLException: Value '0000-00-00 00:00:00' can not be represented as java.sql.Timestamp

**解决方法**：

- 在JDBC连接串中有一项属性：zeroDateTimeBehavior,可以用来配置出现这种情况时的处理策略，该属性有下列三个属性值：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1> exception：默认值，即抛出SQL state [S1009]. Cannot convert value....的异常；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2> convertToNull：将日期转换成NULL值；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3> round：替换成最近的日期即0001-01-01；

- 因此对于这类异常，可以考虑通过修改连接串，附加zeroDateTimeBehavior=convertToNull属性的方式予以规避，例如：
jdbc:mysql://localhost:3306/mydbname?zeroDateTimeBehavior=convertToNull

- 另外，这类异常的触发也与timestamp赋值的操作有关，如果能够在设计阶段和记录写入阶段做好逻辑判断，避免写入 '0000-00-00 00:00:00'这类值，那么也可以避免出现 Cannot convert value '0000-00-00 00:00:00' from column N to TIMESTAMP的错误。



一个异常背后涉及的的东东还是蛮多的，通过一波涨姿势操作，我们还是解决了我们的问题。现梳理成文，以作参考！


