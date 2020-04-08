# redis


## 扩展

### Bloom Filter 布隆过滤器

- bf.add

- bf.exists

- bf.madd

- Bf.mexists

### 位图

- getbit

- setbit

- bitcount

- bitpos

- bitfield

### Lua 脚本

- 命令

	- eval

	- script load 

	- evalsha

### 发布与订阅

- stream

	- 优点

		- 可以获取历史发送的消息

- PUB/SUB

	- 缺点

		- 消息不能持久化
		  PubSub 的生产者传递过来一个消息，Redis 会直接找到相应的消费者传递过去。如果一个消费者都没有，那么消息直接丢弃。如果开始有三个消费者，一个消费者突然挂掉了，生产者会继续发送消息，另外两个消费者可以持续收到消息。但是挂掉的消费者重新连上的时候，这断连期间生产者发送的消息，对于这个消费者来说就是彻底丢失了。  
		    
		  如果 Redis 停机重启，PubSub 的消息是不会持久化的，毕竟 Redis 宕机就相当于一个消费者都没有，所有的消息直接被丢弃。

	- 命令

		- subscribe

			- 订阅频道

		- unsubscribe

			- 退订频道

		- psubscribe

			- 根据正则订阅频道

		- unsubscribe

			- 根据正则退订频道

		- publish

			- 发布消息

		- pubsub

			- 查看订阅信息

### HyperLogLog

- pfadd

- pfcount

- pfmerge

## 复制

### 命令

- SYNC
  主服务器是 Redis 2.8 之前的版本  
    
  运行原理：  
    
  无论是初次连接还是重新连接， 当建立一个从服务器时， 从服务器都将向主服务器发送一个 SYNC 命令。  
    
  接到 SYNC 命令的主服务器将开始执行 BGSAVE ， 并在保存操作执行期间， 将所有新执行的写入命令都保存到一个缓冲区里面。  
    
  当 BGSAVE 执行完毕后， 主服务器将执行保存操作所得的 .rdb 文件发送给从服务器， 从服务器接收这个 .rdb 文件， 并将文件中的数据载入到内存中。  
    
  之后主服务器会以 Redis 命令协议的格式， 将写命令缓冲区中积累的所有内容都发送给从服务器。

- PSYNC
  主服务器是 Redis 2.8 或以上版本

	- 完整重同步 full resynchronization

		- 执行 BGSAVE 

		- 发送rdb文件

		- 发送缓冲区累积的写命令

	- 部分重同步 partial resynchronization
	  主服务器为被发送的复制流创建一个内存缓冲区（in-memory backlog）， 并且主服务器和所有从服务器之间都记录一个复制偏移量（replication offset）和一个主服务器 ID （master run id）， 当出现网络连接断开时， 从服务器会重新连接， 并且向主服务器请求继续执行原来的复制进程：  
	    
	  如果从服务器记录的主服务器 ID 和当前要连接的主服务器的 ID 相同， 并且从服务器记录的偏移量所指定的数据仍然保存在主服务器的复制流缓冲区里面， 那么主服务器会向从服务器发送断线时缺失的那部分数据， 然后复制工作可以继续执行。  
	  否则的话， 从服务器就要执行完整重同步操作。

		- 主服务器复制偏移量（replication offset）和从服务器复制偏移量 

		- 主服务器复制挤压缓冲区（replication backlog）

		- 服务器运行ID （run ID）

### 支持主从同步和从从同步

### 异步复制

## 过期

存在一个expires的过期字典，保存数据库中所有键过期时间

### 设置过期时间

- expire

	- 设置ttl秒

- pexpire

	- 设置ttl毫秒

- expireat

	- 指定秒级时间戳作为过期时间点

- pexpireat

	- 指定毫秒时间戳作为过期时间点

### 移除过期时间

- persist

### 返回剩余过期时间

- ttl

### 过期策略

- 惰性删除策略

- 定期删除策略

	- 随机取一定数量键， 删除其中的过期键

	- 执行时长

	- 执行频率

- 其他模块影响

	- 生成rdb文件

	- 载入rdb文件

	- AOF文件写入

	- AOF重写

	- 复制
	  从库不会进行过期扫描，从库对过期的处理是被动的

## 集群

### Sentinel

 Redis-Sentinel(哨兵模式)是Redis官方推荐的高可用性(HA)解决方案，当用Redis做Master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换，而Redis-sentinel本身也是一个独立运行的进程，它能监控多个master-slave集群，发现master宕机后能进行自懂切换。

- 监控 Monitoring

- 提醒 Notification

- 自动故障迁移 Automatic failover

### Cluster

Redis Cluster是Redis的分布式解决方案，在Redis 3.0版本正式推出的，有效解决了Redis分布式方面的需求。当遇到单机内存、并发、流量等瓶颈时，可以采用Cluster架构达到负载均衡的目的。

### Codis

国人写的集群中间件

### 分布式锁

- 命令

	- setnx
	  > setnx lock:codehole true  
	  OK  
	  > expire lock:codehole 5  
	  ... do something critical ...  
	  > del lock:codehole  
	  (integer) 1

	- expire

	- del

	- set
	  setnx 和 expire 组合在一起的原子指令  
	  set lock-codehole true ex 5 nx

- 限制程序的并发执行

## 事件

### 单进程单线程处理客户端请求

### redis服务器是一个事件驱动程序

### 文件事件

- 基于Reactor模式

- 组成

	- 套接字

	- I/O多路复用

	- 文件事件分派器

	- 事件处理器

### 时间事件

- serverCron

## 通信协议

### RESP(Redis Serialization Protocol)

- 单元类型

	- 单行字符串

	- 多行字符串

	- 整数值

	- 错误消息

	- 数组

- 单元结束

	- \r\n

## 持久化

### RDB

- 保存和还原所有键值对数据

- save 命令

	- 主进程执行，阻塞服务器

- bgsave 命令

	- 子进程执行，非阻塞

### AOF（Append Only File）

AOF 文件通过保存所有修改数据库的命令来记录数据库的状态。  
  
AOF 文件中的所有命令都以 Redis 通讯协议的格式保存。

- AOF重写

## 事务 transaction

注意：  
事务执行过程某条命令发生了错误，仍会继续执行其他命令

### 命令

- MULTI

	- 事务开始

- EXEC

	- 事务执行

- WATCH
  watch 会在事务开始之前盯住 1 个或多个关键变量，当事务执行时，也就是服务器收到了 exec 指令要顺序执行缓存的事务队列时，Redis 会检查关键变量自 watch 之后，是否被修改了 (包括当前事务所在的客户端)。如果关键变量被人动过了，exec 指令就会返回 null 回复告知客户端事务执行失败，这个时候客户端一般会选择重试。  
    
  注意：  
  Redis 禁止在 multi 和 exec 之间执行 watch 指令，而必须在 multi 之前做好盯住关键变量，否则会出错。

	- 监视数据库键

- DISARD

	- 事务丢弃

## 数据结构与对象

### 字符串

- set/mset

- get/mget

### 链表

可实现队列

- 命令

	- rpush/lpush

	- rpop/lpop

	- rpoplpush

	- brpoplpush

	- brpop/blpop
	  没有元素时，阻塞状态，直到等待超时或发现有元素为止

	- list

- 链表实现的特性

	- 双端，前后指针

	- 无环

	- 带表头表尾指针

	- 带长度计数器

	- 多态，保存各种类型值

### 字典

- 实现

	- 哈希

	- 键冲突

		- 链地址法

	- 哈希的扩展和收缩

- 命令

	- hsetnx
	  Adds a value to the hash stored at key only if this field isn't already in the hash.

	- hlen

	- hdel

	- hgetall

	- hincrby

	- hincrbyfloat

	- hexists

	- hmset

	- hvals

	- hmget

	- hkeys

	- hget

	- hset

### 跳跃表 skip list

http://blog.jobbole.com/111731/

- 命令

	- zadd
	  Time complexity: O(log(N)) for each item added, where N is the number of elements in the sorted set.

	- zrem
	   O(M*log(N)) with N being the number of elements in the sorted set and M the number of elements to be removed.

	- zrange/zrevrange
	  O(log(N)+M) with N being the number of elements in the sorted set and M the number of elements returned.

	- zrangebyscore/zrevrangebyscore
	  O(log(N)+M) with N being the number of elements in the sorted set and M the number of elements being returned. If M is constant (e.g. always asking for the first 10 elements with LIMIT), you can consider it O(log(N)).

	- zrangebylex/zrevrangebylex

	- zcount
	  O(log(N)) with N being the number of elements in the sorted set.

	- zremrangebyscore

	- zremrangebyrank

	- zcard
	  O(1)

	- zscore

	- zrank/zrevrank
	  O(log(N))

	- zincrby
	   O(log(N)) where N is the number of elements in the sorted set.

	- zinterstore

### set 集合

- sadd

- scard

- sdiff
  O(N) where N is the total number of elements in all given sets.  
    
  差集

- srem

- sismemeber

- spop

- srandmember

- smove

- sunion
  O(N) where N is the total number of elements in all given sets.  
    
  并集

- sunionstore
  O(N) where N is the total number of elements in all given sets.

- sinter
  O(N*M) worst case where N is the cardinality of the smallest set and M is the number of sets.  
    
  交集

- sinterstore

### 压缩列表 zip list

### 对象

- 命令

	- object

- 字符串对象

- 列表对象

- 哈希对象

- 集合对象

- 有序集合对象

- 内存回收

	- 引用计数

- 对象空转时长
  空转时长：当前时间减去最后一次被命令程序访问的时间

