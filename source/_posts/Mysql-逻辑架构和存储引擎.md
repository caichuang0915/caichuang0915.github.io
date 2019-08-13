---
title: Mysql - Mysql逻辑架构和存储引擎
tags:
  - Mysql
---

Mysql逻辑架构和存储引擎

#### Mysql逻辑架构


##### 衡量指标

- TPS：Transactions Per Second（每秒传输的事物处理个数），这是指服务器每秒处理的事务数，支持事务的存储引擎如InnoDB等特有的一个性能指标。 

- QPS：Queries Per Second（每秒查询处理量）同时适用与InnoDB和MyISAM 引擎

- 等待时间：执行Sql等待返回结果之间的等待时间

<!-- more -->

##### MySqlSlap

官方提供的压力测试工具，部分参数如下：

参数 | 作用  
-|-
--create-schema=name | 指定测试的数据库名，默认是mysqlslap |
--engine=name |创建测试表所使用的存储引擎，可指定多个|
--concurrency=N|模拟N个客户端并发执行。可指定多个值，以逗号或者|
--number-of-queries=N | 总的测试查询次数(并发客户数×每客户查询次数)，比如并发是10，总次数是100，那么10个客户端各执行10个 |
--iterations=N  | 迭代执行的次数，即重复的次数（相同的测试进行N次，求一个平均值），指的是整个步骤的重复次数，包括准备数据、测试load、清理 |
--commit=N | 执行N条DML后提交一次 |
--auto-generate-sql, -a |自动生成测试表和数据，表示用mysqlslap工具自己生成的SQL脚本来测试并发压力。|
--auto-generate-sql-load-type=name|# 测试语句的类型。代表要测试的环境是读操作还是写操作还是两者混合的。# 取值包括：read (scan tables), write (insert into tables), key (read primary keys), update (update primary keys), or mixed (half inserts, half scanning selects). 默认值是：mixed.|
--auto-generate-sql-add-auto-increment|对生成的表自动添加auto_increment列|
--number-char-cols=name |自动生成的测试表中包含N个字符类型的列，默认1|
--number-int-cols=name|自动生成的测试表中包含N个数字类型的列，默认1|
--debug-info |打印内存和CPU的信息|


例：    

```sh
./mysqlslap -uroot -proot1234% --concurrency=1000  --iterations 10 -a  --auto-generate-sql-add-autoincrement --engine=innodb --number-of-queries=1000

./mysqlslap -uroot -proot1234%  --concurrency=1,50,100,200 --iterations=3 --number-char-cols=5 --number-int-cols=5 --auto-generate-sql --auto-generate-sql-add-autoincrement  --engine=myisam,innodb  --create-schema='enjoytest1' --debug-info 

```

#### Mysql逻辑架构

![逻辑架构](http://image.tupelo.top/mysql%E9%80%BB%E8%BE%91.png)

- 连接层

  每一个客户端连接请求，服务器都会新建一个线程处理（如果是线程池的话，则是分配一个空的线程），每个线程独立，拥有各自的内存处理空间，会引发数据同步问题。

- 服务层

  - SQL处理层

    SQL语句的解析、优化，缓存的查询，MySQL内置函数的实现

  - 缓存

    默认不开启。若手动开启还要手动设置缓存大小

    ```sql
    show variables like  '%query_cache_type%'

    #开启缓存需要在配置文件中开启
    show variables like  '%basedir%'

    SET GLOBAL query_cache_size = 4000;
    ```

- 引擎层
- 存储层

#### Mysql存储引擎

```sql
#看你的mysql现在已提供什么存储引擎:
show engines;
  
#看你的mysql当前默认的存储引擎:
show variables like '%storage_engine%';
```

##### MyISAM
  
文件存储为：frm、MYD、MYI。特点：

- 不支持事务
- 表级锁     
- 支持全文检索    
- 支持数据压缩 压缩MYI文件，会生成一个.old的文件，删除后会导致历史数据只能查询，不能删除和修改

```sql
myisampack -b -f testmysam.MYI
```

适用场景：

- 非事务型应用（数据仓库，报表，日志数据）
- 只读类应用 查询速度快
- 空间类应用（空间函数，坐标）
           

##### Innodb

- 系统表空间
  
  数据全部存在一个文件 ibdataX


- 独立表空间

  每个表数据单独存储为：tablename.ibd

二者差别：

- 系统表空间无法简单的收缩文件大小，删除数据后ibdataX文件大小不变
- 独立表空间可以通过optimize table 收缩系统文件，删除数据后可刷新tablename.ibd文件大小
- 系统表空间会产生IO瓶颈，多个表写一个文件
- 独立表空间可以同时向多个文件刷新数据，多个表写多个文件


特点：

- Innodb是一种事务性存储引擎
- 完全支持事务得ACID特性
- Redo Log 和 Undo Log
- Innodb支持行级锁（并发程度更高）

适用场景：大多数OLTP应用

对比项 | MyISAM | InnoDB  
-|-|-
主外键|不支持|支持|
事务|不支持|支持|
行表锁|表锁，即使操作一条记录也会锁住整个表
不适合高并发的操作|行锁,操作时只锁某一行，不对其它行有影响
适合高并发的操作|
缓存|只缓存索引，不缓存真实数据|不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响|
表空间|小|大|
关注点|性能|事务|
默认安装|Y|YY|

##### CSV

组成:

- csv文件存储内容
- csm文件存储表得元数据如表状态和数据量
- frm 表结构

特点:

- 以csv格式进行数据存储
- 所有列都不能为null的
- 不支持索引（不适合大表，不适合在线处理）
- 可以对数据文件直接编辑（保存文本文件内容）

适用场景：财务

##### Archive

组成:
  
- frm 表结构
- arz 以zlib对表数据进行压缩，磁盘I/O更少 数据存储在ARZ为后缀的文件中

特点:

- 只支持insert和select操作
- 只允许在自增ID列上加索引

适用场景：日志收集

##### Memory

文件系统存储特点：

- 也称HEAP存储引擎，所以数据保存在内存中
- 支持HASH索引和BTree索引
- 所有字段都是固定长度 varchar(10) = char(10)
- 不支持Blog和Text等大字段
- Memory存储引擎使用表级锁
- 最大大小由max_heap_table_size参数决定

临时表：也存在内存中，但是只限于当前会话

memory数据易丢失，所以要求数据可再生

适用场景:

- hash索引用于查找或者是映射表（邮编和地区的对应表）
- 用于保存数据分析中产生的中间表
- 用于缓存周期性聚合数据的结果表


##### Federated

特点:

- 提供了访问远程MySQL服务器上表的方法
- 本地不存储数据，数据全部放到远程服务器上
- 本地需要保存表结构和远程服务器的连接信息

默认禁止，启用需要再启动时增加federated参数

创建表时添加connect=mysql://user_name[:password]@hostname[:port_num]/db_name/table_name










