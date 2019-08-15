---
title: Mysql - MySql慢查询和执行计划(重点)
tags:
  - Mysql
---

MySql慢查询和执行计划

#### MySql慢查询

相关配置：

```sql
SHOW VARIABLES LIKE '%slow%'

-- 开启或者关闭
SET GLOBAL slow_query_log = 1

-- 设置时间
SET GLOBAL long_query_time = 10
```

- slow_query_log 启动停止技术慢查询日志
- slow_query_log_file 指定慢查询日志得存储路径及文件（默认和数据文件放一起）
- long_query_time 指定记录慢查询日志SQL执行时间得伐值（单位：秒，默认10秒）
- log_queries_not_using_indexes  是否记录未使用索引的SQL
- log_output 日志存放的地方【TABLE】【FILE】【FILE,TABLE】

	使用TABLE时慢sql会存在mysql.slow_log 不建议使用。

<!-- more -->

##### MySql慢查询分析工具

mysqldumpslow:汇总除查询条件外其他完全相同的SQL，并将分析结果按照参数中所指定的顺序输出。

语法：

```sh
#
#
# -s order (c,t,l,r,at,al,ar) 
#    c:总次数
#    t:总时间
#    l:锁的时间
#    r:总数据行
#    at,al,ar  :t,l,r平均数  【例如：at = 总时间/总次数】
#    -t  top   指定取前面几天作为结果输出

mysqldumpslow -s r -t 10 slow-mysql.log
```

其它工具(需要安装)：pt_query_digest

语法：

```sh
pt-query-digest  --explain h=127.0.0.1, u=root,p=password slow-mysql.log

```

#### MySql执行计划


##### 索引

语法：

```sql
--查看索引
SHOW INDEX FROM table_name

--创建索引
CREATE  [UNIQUE ] INDEX indexName ON mytable(columnname(length));
ALTER TABLE 表名 ADD  [UNIQUE ]  INDEX [indexName] ON (columnname(length)) 

--删除索引
DROP INDEX [indexName] ON mytable;

```

分类：

- 普通索引：即一个索引只包含单个列，一个表可以有多个单列索引
- 唯一索引：索引列的值必须唯一，但允许有空值
- 复合索引：即一个索引包含多个列
- 聚簇索引(聚集索引)：并不是一种单独的索引类型，而是一种数据存储方式。具体细节取决于不同的实现，InnoDB的聚簇索引其实就是在同一个结构中保存了B-Tree索引(技术上来说是B+Tree)和数据行。
- 非聚簇索引：不是聚簇索引，就是非聚簇索引


##### 执行计划

语法：

```sql
Explain + SQL语句
```

<font color=red>执行计划包含的信息(重点)</font>:

- id 查询顺序

	select查询的序列号,包含一组数字，表示查询中执行select子句或操作表的顺序。id相同，执行顺序由上至下。id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行。

- select_type 查询类型

	- SIMPLE：简单的 select 查询,查询中不包含子查询或者UNION
	- PRIMARY：查询中若包含任何复杂的子部分，最外层查询则被标记为
	- SUBQUERY：在SELECT或WHERE列表中包含了子查询
	- DERIVED：在FROM列表中包含的子查询被标记为DERIVED(衍生)，MySQL会递归执行这些子查询, 把结果放在临时表里。
	- UNION：若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中,外层SELECT将被标记为：DERIVED
	- UNION RESULT：从UNION表获取结果的SELECT

- table
- type(重要)

	system>const>eq_ref>ref>range>index>ALL		
	system：表只有一行记录		
	const：表示通过索引一次就找到了		
	eq_ref：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描		
	ref：非唯一性索引扫描，返回匹配某个单独值的所有行.		
	range：范围查询		
	index：查询索引列		
	all：全表扫描		

- possible_keys 可能用到的索引
- key 实际用到的索引



