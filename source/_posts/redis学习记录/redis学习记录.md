---
title: redis学习记录
date: 2019-02-18 19:16:33
description: 文章访问需要密码
password: llz721097
categories: 
    - 数据库
    - redis
tags: 
    - 数据库
    - redis 
---

# 指令
- 设置有效时间：`expire [属性名] [有效时间]`,单位：s
- 查看某属性剩余有效时间：`ttl [属性名]`，返回剩余秒数
- 取消过期时间：`persist [属性名]`，取消在该属性名上设置的有效时间，之后用`ttl`查看时返回-1
 
<br>

- 选择数据库：`select n`，其中n取值0~15，redis中16个数据库(逻辑上的划分)，登录进来默认是第0个数据库
- `move [key] [数据库下标]`：将当前数据库中的key转移到其他数据库中
- `randomkey`：随机返回数据库中的一个key
- `dbsize`：查看当前数据库的key的数量
- `info`：获取数据库信息
- `config get *`：查看配置信息，展示的即`redis.conf`文件中的信息
- `flushdb`：清空当前数据库
- `flushall`：清空所有数据库


- `rename [原key名] [新key名]`：重命名key


# 主从复制
## 概念

1. `master`可以拥有多个`slave`
2. 多个`slave`可以连接同一个`master`，还可以连接到其他的`slave`
3. 主从复制不会阻塞`master`，在同步数据时，`master`可以继续处理client请求
4. 提供系统的伸缩性

## 过程
1. slave与master建立连接，发送sync同步命令
2. master会开启一个后台进程，将数据库快照保存到文件中，同时master主进程会开始收集新的写命令并缓存
3. 后台完成保存后，将文件发送给slave
4. slave将此文件保存到硬盘上

## 配置
clone服务器后修改`slave`的ip地址，修改从服务器的`redis.conf`配置文件

```
slaveof [master的ip] [master的端口号]
```

若master设置了密码，则加一行配置：
```
masterauth [master的密码]
```

# 持久化
## 1.rdb
- `snapshotting`(快照)默认方式，将内存中以快照方式写入到二进制文件中，默认为`dump.rdb`
- 可以通过配置设置自动做快照持久化的方式，可以配置redis在n秒内如果超过m个key则修改被自动做快照，配置如下：

```
save 900 10 //900秒内如果超过10个key被修改，则发起快照保存
```

## 2.aof
- `append-only file`方式(类似于oracle日志)，由于快照方式是在一定时间间隔做一次，所以可能发生redis意外down的情况丢失最后一次快照后的所有修改数据
- aof比快照方式有更好的持久化性，是由于在使用aof时，redis会将每一个收到的写命令都通过`write`函数追加到命令中
- 当redis重启时会重新执行文件中保存的写命令来在内存中重建这个数据库的内容
- 该文件在`bin`目录下
- aof不是立即写到硬盘上，可以通过配置文件修改强制写到硬盘上，配置如下：

```
appendonly yes  //启动aof持久化方式，有3种修改方式
# appendfsync always    //收到写命令就立即写入到磁盘，效率最慢，但是保证完全的持久化
# appendfsync everysec  //每秒钟写入磁盘一次，在性能和持久化方面做了很好的折中
# appendfsync no    //完全依赖os，性能最好，持久化没保证
```

# 发布订阅消息
- `subscribe [频道]`：订阅监听
- `publish [频道] [发布内容]`：发布消息广播

# redis集群搭建
- 在redis3.0前，提供了`sentinel`工具来监控各`Master`的状态，若`master`异常，则会做主从切换，将`slave`作为`master`，`master`作为`slave`
- redis3.0后，支持集群的容错功能

## 步骤
1. 集群至少需要3个master
2. 创建一个文件夹`redis-cluster`，然后在其下面分别创建6个子文件夹 (8001,8002,8003,8004,8005,8006) 当作redis节点 (3主3从) ，将`redis.conf`文件copy到6个子文件夹中，并分别修改配置文件：

```
daemonize yes   //设置redis为后台启动
port [端口号]   //分别为6个节点指定端口号
bind [ip]   //绑定当前机器的ip
dir [目录]  //分别指定数据文件存储位置
cluster-enabled yes //开启集群模式
cluster-config-file nodes-800*.conf     //800*最好和port对应
cluster-node-timeout 5000   //单位：ms
appendonly yes
```

3. 由于redis集群需要使用`ruby`命令，需要安装`ruby`

```
yum install ruby
yum install rubygems
gem install redis   //安装redis和ruby的接口
```

4. 分别启动6个redis实例
5. 进入redis安装目录，执行`redis-trib.rb`命令（src下）

```
./redis-trib.rb create --replicas 1 [ip]:[端口1] [ip]:[端口2] [ip]:[端口3] [ip]:[端口4] [ip]:[端口5] [ip]:[端口6]       

//其中1表示主节点比从节点的比值，该demo表示主节点3个，从节点3个，且前3个为主节点
```


6.以上即完成了集群的搭建，进行验证  
- 连接任意一个客户端：`./redis-cli -c -h [ip地址] -p [端口号]`，其中`-c`表示集群模式，指定ip地址和端口号`
- `cluster nodes`：查看当前集群下所有节点信息



    






