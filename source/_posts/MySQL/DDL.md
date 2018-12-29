---
title: MySQL学习记录之----DDL语句
date: 2017-10-16 10:25:23
categories: 
    - 数据库
    - MySQL
tags: 
    - 数据库
    - MySQL 
---
# 库的管理
查看系统变量`show variables`
## 创建库
```sql
create database if not exists 库名
```
## 修改库的字符集
```sql
alter database 库名 character set 字符集名
```
## 删除库
```sql
drop database if exists 库名
```
# 表的管理
## 创建表
```sql
create table if not exists 表名(
    列名 列的类型(长度) 约束，
    列名 列的类型(长度) 约束，
    ...
)
```
## 修改表
### 修改列名和类型
```sql
//其中column可以省略
alter table 表名 change column 列名 新列名 新列名的类型
```
### 修改列的类型和约束
`alter table 表名 modify column 列名 新列名的类型 约束`

`alter table 表名 add foreign key(列名) references 关联的表(关联外键的列名)`
### 添加新列
`alter table 表名 add column 列名 列的类型`
### 删除列
`alter table 表名 drop column 列名`
### 修改表名
`alter table 表名 rename to 新表名`
## 删除表
`drop table if exists 表名`
## 复制表
### 仅复制表结构
`create table 要创建的表名 copy like 被复制的表名`
### 复制表的结构+数据
`create table 要创建的表名 select * from 被复制的表名`
### 只复制部分数据
```sql
create table 要创建的表名 
select * from 被复制的表名
where 筛选条件
```
# 常见约束
```sql
create table student(
    id int primary key auto_increment,     #主键自增长
    stu_name varchar(20) not null,      #非空
    gender char(1) check(gender='0' or gender='1'),     #检查（mysql不支持）
    seat int unique,     #唯一
    age int default 18,     #默认
    major_id int,
    constraint fk_student_major_id foreign key(major_id) references major(id)   #指定student表的major_id字段是major表的id字段的外键，起别名为fk_student_major_id
)
```

- 自增长列不一定需要和主键搭配
- 一个表至多只能有一个自增长列
- 自增长列类型只能为数值型,包含浮点型
- 自增长列可以通过`set auto_increment_increment=3`设置增长的步长

## 非空约束not null
## 默认约束default
保证该字段有默认值
## 主键约束primary key
字段值唯一且非空
## 唯一约束unique
字段值唯一但可以为空
## 检查约束check(mysql不支持)
## 外键约束foreign key















