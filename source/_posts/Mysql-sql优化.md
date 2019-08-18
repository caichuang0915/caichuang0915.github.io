---
title: Mysql - sql优化
tags:
  - Mysql
---

sql优化

#### sql优化

1. 尽量全值匹配,有联合索引的话key_len会保持在合适的大小
2. 最佳左前缀法则
3. 不在索引列上做任何操作
4. 范围条件放最后
5. 覆盖索引尽量用 不要 ```select *```
6. 不等于要慎用 mysql 在使用不等于(!= 或者<>)的时候无法使用索引会导致全表扫描
7. Null/Not Null 有影响 当字段可为空时，is null会使用索引，is not null会导致索引失效。当字段不为空是，两者都不会使用索引。
8. Like查询要当心
9. 字符类型加引号
10. OR改UNION效率高,用union更快

#### sql快速导入导出

```sql
# 导出
select * into OUTFILE 'D:\\product.txt' from product_info
# 导入
load data INFILE 'D:\\product.txt' into table product_info
```













