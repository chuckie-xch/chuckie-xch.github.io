---
title: Redis主从复制
date: 2018-03-28 00:00:31
tags: redis
---

Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持字符串、哈希表、列表、集合、有序集合，位图，hyperloglogs等数据类型。内置复制、Lua脚本、LRU收回、事务以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动分区。

<!--more-->

### 持久化

持久化就是把内存中的数据写到磁盘中，防止服务宕机。

* RDB   (默认)
* AOF

#### RDB

RDB功能的核心函数`rdbSave`方法生成RDB文件，`rdbLoad`方法加载文件到内存。

**rdbSave方法**：将内存中的数据以RDB格式保存到磁盘中，如果文件已经存在，那么新的RDB文件将会覆盖已有的RDB文件。保存期间，主进程会被阻塞，知道保存完成。

**save**直接调用rdbSave，阻塞主线程。

**bgSave**则fork出一个子进程，调用rdbSave，并在保存完成后向主进程发送信号，通知保存完成。

**存储结构：**



**保存策略：**

* save  900   1
* save   300   10     // 300秒内容如果超过10个key被修改，则发起快照保存
* save  60  10000 



#### AOF

AOF是Append-only-file的缩写。

服务器定时任务调用`flushAppendOnlyFile`函数，该函数完成下面两个工作：

* aof写入保存：

  * write：根据条件，将`aof_buf`中的缓存写入到AOF文件
  * save：根据条件，调用`fsync`或者`fdatasync`函数，将AOF文件保存到磁盘中。

* 保存策略：

  | 模式               | write是否阻塞 | save是否阻塞 | 停机时丢失数据量                |
  | ------------------ | ------------- | ------------ | ------------------------------- |
  | AOF_FSYNC_NO       | 阻塞          | 阻塞         | 最后一次对AOF进行save之后的数据 |
  | AOF_FSYNC_EVERYSEC | 阻塞          | 不阻塞       | 不超过2秒钟的数据               |
  | AOF_FSYNC_ALWAYS   | 阻塞          | 阻塞         | 丢失一个命令的数据              |

**存储结构：**

内容是Redis通讯协议（RESP）格式的命令文本存储。



### 复制

通常为被复制方（master）主动向复制方（slave）发送数据，主要目的是为了保证双方数据一致性，做备份容灾，并且分摊master的压力。



#### 主从复制实战

```shell
cp redis.conf redis-slave1.conf
vim redis-slave1.conf
/slaveof         # 搜索到slaveof
slaveof 127.0.0.1 6379     # master ip及端口
/port 6379    # 搜索到port端口 并更改为6380
# 保存退出
# 修改master bind
vim redis.conf
/bind    # 搜索bind改为如下
bind 0.0.0.0
# 保存退出
# 分别启动两个redis服务
./src/redis-server redis.conf $
./src/redis-server redis-slave1.conf $
```

