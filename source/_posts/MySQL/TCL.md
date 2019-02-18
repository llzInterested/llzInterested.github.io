---
title: MySQL学习记录之----事务
date: 2017-11-12 14:46:32
description: 文章访问需要密码
password: llz721097
categories: 
    - 数据库
    - MySQL
tags: 
    - 数据库
    - MySQL 
---
# 事务控制语言Transaction Control Language
## Mysql存储引擎
`show engines`查看mysql支持的存储引擎
- 在mysql中用的最多的存储引擎有：innodb，myisam，memory等
- innodb支持事务，myisam,memory不支持事务
## 事务概念
 一组要么同时执行成功，要么同时执行失败的SQL语句，是数据库操作的一个执行单元
- 事务开始于：
    - 连接到数据库上，并执行一条DML语句(INSERT，UPDATE，DELETE)
    - 前一个事务结束后又输入另一条DML语句
- 事务结束于：
    - 执行COMMIT或ROLLBACK语句
    - 执行一条DDL语句，如CREATE TABLE语句，这种情况下会自动执行COMMIT语句
    - 执行一条DCL语句，如GRANT语句，这种情况下会自动执行COMMIT语句
    - 断开与数据库的连接
    - 执行了一条DML语句但失败了，这种情况下会为这个无效的DML语句执行ROLLBACK语句
## 事务ACID
- atomicity(原子性)：表示一个事务内所有操作是一个整体，要么全部成功，要么全部失败

- consistency(一致性)：事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态,以转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性

- isolation(隔离性)：当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离,对于任意两个并发的事务T1和T2，在事务T1看来，T2要么在T1开始之前就已经结束，要么在T1结束之后才开始，这样每个事务都感觉不到有其他事务在并发地执行

- durability(持久性)：持久性事务完成后，它对于系统的影响是永久性的，例如我们在使用JDBC操作数据库时，在提交事务方法后，提示用户事务操作完成，当我们程序执行完成直到看到提示后，就可以认定事务以及正确提交，即使这时候数据库出现了问题，也必须要将我们的事务完全执行完成，否则就会造成我们看到提示事务处理完毕，但是数据库因为故障而没有执行事务的重大错误
## 事务创建
### 隐示事务
事务没有明显的开启和结束标记，如`insert`,`update`,`delete`语句
### 显示事务
事务具有明显的开启和结束标记
- 前提：必须先设置自动提交功能为禁用，`set autocommit=0`
显示事务步骤：
1. `set autocommit=0`,禁用自动提交功能
2. `start transaction`，开启事务，该步可省略
3. 编写事务中的sql，如`select`，`insert`，`update`
4. `commit`,提交事务，`rollback`，回滚事务
## 事务并发问题
### 脏读
对于2个事务T1，T2，T1读取了已经被T2更新但还没有被提交的字段,若T2回滚，则T1读取的内容就是临时且无效的
### 不可重复读
对于2个事务T1，T2，T1读取了一个字段，然后T2更新了该字段，T1再次读取同一个字段值就不同了
### 幻读
对于2个事务T1，T2，T1从一个表中读取了一个字段，然后T2在该表中插入了一些新的行，如果T1再次读取同一个表就会多出几行数据
## 事务隔离级别
|隔离级别|描述|
|:-:|:-:|
|read uncommitted(读未提交的数据)|允许事务读取未被其他事务提交的变更，脏读，不可重复读，幻读的问题都会出现|
|read committed(读已提交的数据)|只允许事务读取已被其他事务提交的变更，可避免脏读，但不可重复读，幻读的问题可能出现|
|repeatable read(可重复读)|确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间内，禁止其他事务对这个字段进行更新，可避免脏读和不可重复读，但幻读的问题可能出现|
|serializable(串行化)|确保事务可以从一个表中读取相同的行，在这个事务持续期间内，禁止其他事务对该表执行插入，更新和删除，所有并发问题都可以避免，但性能低下|

- oracle支持2种事务隔离级别：`read committed`,`serializable`。oracle默认的事务隔离级别为`read committed`
- mysql支持4种事务隔离级别，mysql默认的事务隔离级别为`repeatable read`
### 查看默认隔离级别
`select @@tx_isolation`
### 设置隔离级别
设置当前mysql连接的隔离级别

`set session transaction isolation level 事务隔离级别`

设置数据库系统全局的隔离级别

`set global transaction isolation level 事务隔离级别`
## 保存点savepoint
```sql
set autocommit=0;
start transaction;
delete from emps where id=1;
savepoint a;  #设置保存点
delete from emps where id=2;
rollback to a;  #回滚到保存点
```
























