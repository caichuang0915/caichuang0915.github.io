---
title: Mysql - Mysql锁和事务
tags:
  - Mysql
---

Mysql锁和事务

#### Mysql锁

不同的存储引擎支持不同的锁机制,MyISAM和MEMORY存储引擎采用的是表级锁InnoDB存储引擎既支持行级锁，也支持表级锁，但默认情况下是采用行级锁。


- 表级锁:开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。
- 行级锁:开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。
- 页面锁:开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。


表级锁更适合于以查询为主，只有少量按索引条件更新数据的应用，如OLAP系统   

行级锁则更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用    


<!-- more -->

##### MyISAM的表锁

语法：

```sql
-- 读锁
lock table table_name read

-- 写锁
lock table table_name write

--释放锁
unlock tables
```

特点：

读锁：会阻塞其他用户对该表的写请求，当前session访问其他表也会报错。
写锁：阻塞其他表的读和写，当前session curd都可以，访问其他表也会报错。

##### InnoDb行锁

语法：

```sql
begin
-- 读锁
select  *  from table_name where  条件  lock in share mode

-- 写锁
select *  from table_name  where 条件 for update

--释放锁
commit/rollback
```

1. 两个事务不能锁同一个索引。开启一个新事务的时候会解锁表。
2. insert ，delete ， update在事务中都会自动默认加上排它锁。
3. 行锁必须有索引才能实现，否则会自动锁全表，那么就不是行锁了。



问：    
系统运行一段时间，数据量已经很大，这时候系统升级，有张表A需要增加个字段，并发量白天晚上都很大，请问怎么修改表结构   

答：    

1. 首先创建一个和你要执行的alter操作的表一样的空的表结构。
2. 执行我们赋予的表结构的修改，然后copy原表中的数据到新表里面。
3. 在原表上创建一个触发器在数据copy的过程中，将原表的更新数据的操作全部更新到新的表中来。 
4. copy完成之后，用rename table 新表代替原表，默认删除原表。

可使用操作工具完成操作工具(需安装)：pt-online-schema-change

```sh
# 下载安装perl环境  http://www.perl.org/get.html 
# 下载percona-toolkit工具集合  https://www.percona.com/doc/percona-toolkit
# 安装环境
ppm install DBI 

ppm install DBD::mysql 

# 执行
pt-online-schema-change h=127.0.0.1,u=root,D=mysqldemo,t=product_info --alter "modify product_name varchar(150) not null default '' " --execute
```


#### MySql事务

语法：

```sql

--开启事务
begin 

START TRANSACTION（推荐）

begin work 

-- 事务回滚 可配合还原点使用savepoint
rollback

-- 事务提交  
commit 

-- 查看数据库下面是否支持事务（InnoDB支持）？
show engines;

-- 查看mysql当前默认的存储引擎？
show variables like '%storage_engine%';

-- 查看某张表的存储引擎？
show create table 表名 ;

-- 对于表的存储结构的修改？
Create table .... type=InnoDB 
Alter table table_name type=InnoDB;

```

##### 事务的特性ACID

- 原子性 全部成功或者全部失败
- 一致性 数据的完整行没有被破坏
- 隔离性 各个事务间不能相互影响
- 持久性 已提交的数据就会永久保存在数据库

##### 事务的隔离级别



事务并发问题

1. 脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据
2. 不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。
3. 幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。


语法：

```sql
set SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
```

- 未提交读（READ UNCOMMITED）存在脏读

	级别最低，效率最快。事务A在修改数据时，未提交事务B也能看见修改后的数据。存在脏读、不可重复读、幻读。

- 已提交读 （READ COMMITED）存在不可重复读

	事务A在修改数据时，未提交事务B不能看见修改后的数据，事务A提交后事务B能看见。存在不可重复读、幻读。

- 可重复读（REPEATABLE READ）存在幻读

	事务B刚开始查询到的数据不会修改，不管数据有没有被其他数据修改。存在幻读。如果有索引（包括主键索引）的时候，以索引列为条件更新数据，会存在间隙锁间、行锁、页锁的问题，从而锁住一些行；如果没有索引，更新数据时会锁住整张表。


- 可串行化（SERIALIZABLE）

	级别最高，效率最低。会锁表。

隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大，对于多数应用程序，可以优先考虑把数据库系统的隔离级别设为Read Committed，它能够避免脏读取，而且具有较好的并发性能。mysql默认的事务隔离级别为repeatable-read。












