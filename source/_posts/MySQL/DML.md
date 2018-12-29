---
title: MySQL学习记录之----DML语句
date: 2017-10-19 15:11:43
categories: 
    - 数据库
    - MySQL
tags: 
    - 数据库
    - MySQL 
---
# 插入
```sql
//语法1
insert into 表名(列名...) values (值...)

//语法2
insert into 表名 set 列名=值，列名=值 ...  
```
- 语法1支持插入多行，如`insert into 表名 values(值...),(值...)... `
- 语法1支持子查询，如`insert into 表名 select ...`
# 删除
## 单表删除
```sql
//语法1
delete from 表名 where 筛选条件

//语法2，全表删除，不能加where条件
truncate table 表名
```
- delete可以加where条件，truncate不能
- 全表删除时使用truncate效率比delete高
- 若删除的表中有自增长列，delete删除后再插入数据自增长列的值从断点开始，使用truncate删除后从0开始
- truncate删除无返回值，delete删除有返回值
- truncate删除不能回滚，delete删除可以回滚
## 多表删除
```sql
//sql92语法,要删除哪个表中的数据，delete后加哪个表的别名
delete 表1的别名，表2的别名 from 表1 别名1，表2 别名2 where 连接条件 and 筛选条件

//sql99语法
delete 表1的别名，表2的别名 from 表1 别名1 连接条件 join 表2 别名2 on 连接条件 where 筛选条件
```
# 修改
## 单表修改
```sql
update 表名 set 列=值，列=值... where 筛选条件 
```
## 多表修改
```sql
//sql92语法
update 表1 别名1，表2 别名2 set 列=值... where 连接条件 and 筛选条件

//sql99语法
update 表1 别名1 连接类型 join 表2 别名2 on 连接条件 set 列=值... where 筛选条件
```