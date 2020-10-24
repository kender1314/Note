# Redis 笔记

## redis是什么?

Redis 通常被称为数据结构服务器，因为值(value)可以是字符串(String), 哈希(Map), 列表(list), 集合(sets) 或有序集合(sorted sets)等类型。

Redis 是互联网技术中使用最为广泛的中间件之一

Redis 是完全开源免费的，遵守BSD协议，是一个灵活的高性能key-value数据结构存储，可以用来作为数据库，缓存和消息队列。

Redis 比其他 key-value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载到内存使用。
- Redis不仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持主从复制，即master-slave模式的数据备份。



## Redis的特点

1. **高性能**： Redis将所有数据集存储在内存中，可以在入门级Linux机器中每秒写（SET）11万次，读（GET）8.1万次。Redis支持Pipelining命令，可一次发送多条命令来提高吞吐率，减少通信延迟。
2. **持久化**：当所有数据都存在于内存中时，可以根据自上次保存以来经过的时间和/或更新次数，使用灵活的策略将更改异步保存在磁盘上。Redis支持仅附加文件（AOF）持久化模式。
3. **数据结构**： Redis支持各种类型的数据结构，例如字符串，散列，集合，列表，带有范围查询的有序集，位图，超级日志和带有半径查询的地理空间索引。
4. **原子操作**：处理不同数据类型的Redis操作是原子操作，因此可以安全地设置或增加键，添加和删除组中的元素，增加计数器等。
5. **支持的语言**： Redis支持许多语言，如ActionScript，C，C++，Erlang，Go，Haskell，Java，JavaScript（Node.js)，Lua，Objective-C，Perl，PHP，Python，R，Ruby，Rust，Scala，Smalltalk和Tcl。
6. **主/从复制**： Redis遵循非常简单快速的主/从复制。配置文件中只需要一行来设置它，而Slave在Amazon EC2实例上完成10 MM key集的初始同步需要21秒。
7. **分片**： Redis支持分片。与其他键值存储一样，跨多个Redis实例分发数据集非常容易。
8. **可移植**： Redis是用ANSI C编写的，适用于大多数POSIX系统，如Linux，BSD，Mac OS X，Solaris等。

## Redis与其他key-value存储有什么不同？

- Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
- Redis虽然运行在内存中，但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是，相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。同时，因RDB和AOF两种磁盘持久化方式是不适合随机访问，所以他们可以是紧凑的以追加的方式生成。





## Redis的数据类型

Redis数据库支持五种数据类型。

- 字符串（string）
- 哈希（hash）
- 列表（list）
- 集合（set）
- 有序集合（sorted set）

### 字符串

String是一组字节。在Redis数据库中，字符串是二进制安全的。这意味着它们具有已知长度，并且不受任何特殊终止字符的影响。可以在一个字符串中存储最多512兆字节的内容。

例

使用SET命令在name键中存储字符串“redis.com.cn”，然后使用GET命令查询name。

```
SET name "redis.com.cn"  
```

OK  

```
GET name   
```

"redis.com.cn" 

![Redis数据类型1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201024152530.png)



### 哈希

哈希是键值对的集合。在Redis中，哈希是字符串字段和字符串值之间的映射。因此，它们适合表示对象。

例

让我们存储一个用户的对象，其中包含用户的基本信息。

```
HMSET user:1 username ajeet password javatpoint alexa 2000  
OK  
HGETALL  user:1  
"username"  
"ajeet"  
"password"  
"javatpoint"  
"alexa"  
"2000" 
```

![Redis数据类型2](../../../../Typora/picture/redis-data-types2-1.png)

这里，HMSET和HGETALL是Redis的命令，而user：1是键。

每个哈希可以存储多达232 – 1亿个字段 – 值对。

### 列表

Redis列表定义为字符串列表，按插入顺序排序。可以将元素添加到Redis列表的头部或尾部。

例

```
lpush javatpoint java  
(integer) 1  
lpush javatpoint java  
(integer) 1  
lpush javatpoint java  
(integer) 1  
lpush javatpoint java  
(integer) 1  
lrange javatpoint 0 10  
"cassandra"  
"mongodb"  
"sql"  
"java"  
```

![Redis数据类型3](../../../../Typora/picture/redis-data-types3-1.png)

列表的最大长度为232 – 1个元素（每个列表超过40亿个元素）。

### 集合

集合（set）是Redis数据库中的无序字符串集合。在Redis中，添加，删除和查找的时间复杂度是O(1)。

例

```
sadd tutoriallist redis  
(integer) 1  
redis 127.0.0.1:6379> sadd tutoriallist sql  
(integer) 1  
redis 127.0.0.1:6379> sadd tutoriallist postgresql  
(integer) 1  
redis 127.0.0.1:6379> sadd tutoriallist postgresql  
(integer) 0  
redis 127.0.0.1:6379> sadd tutoriallist postgresql  
(integer) 0  
redis 127.0.0.1:6379> smembers tutoriallist  
1) "redis"  
2) "postgresql"  
3) "sql" 
```

![Redis数据类型4](../../../../Typora/picture/redis-data-types4-1.png)

在上面的示例中，您可以看到postgresql被添加了三次，但由于该集的唯一属性，它只添加一次。

集合中的最大成员数为232 – 1个元素（每个列表超过40亿个元素）。

### 有序集合

Redis有序集合类似于Redis集合，也是一组非重复的字符串集合。但是，排序集的每个成员都与一个分数相关联，该分数用于获取从最小到最高分数的有序排序集。虽然成员是独特的，但可以重复分数。

例

```
redis 127.0.0.1:6379> zadd tutoriallist 0 redis  
(integer) 1  
redis 127.0.0.1:6379> zadd tutoriallist 0 sql  
(integer) 1  
redis 127.0.0.1:6379> zadd tutoriallist 0 postgresql  
(integer) 1  
redis 127.0.0.1:6379> zadd tutoriallist 0 postgresql  
(integer) 0  
redis 127.0.0.1:6379> zadd tutoriallist 0 postgresql  
(integer) 0  
redis 127.0.0.1:6379> ZRANGEBYSCORE tutoriallist 0 10  
1) "postgresql"  
2) "redis"  
3) "sql"   
```

![Redis数据类型5](../../../../Typora/picture/redis-data-types5-1.png)

















###