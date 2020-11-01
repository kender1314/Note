# Redis 笔记

## Redis命令

### docker安装

```
docker run --name redis -d -v /data/redis/data:/data -p 6379:6379 redis redis-server --appendonly yes --requirepass 123456
```

requirepass：设置账号密码为123465

### 连接redis客户端

```
redis-cli

auth 123456  #登陆
```

### 配置信息命令

```
CONFIG GET *          #获取所有配置信息
CONFIG GET loglevel   #获取配置中的loglevel的信息
CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE   #编辑配置信息
```

### 在远程服务上执行命令

如果需要在远程 redis 服务上执行命令，同样我们使用的也是 **redis-cli** 命令。

**语法**

```
$ redis-cli -h host -p port -a password
```

**实例**

以下实例演示了如何连接到主机为 127.0.0.1，端口为 6379 ，密码为 mypass 的 redis 服务上。

```
$redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> PING
PONG
```

### Redis键(Keys)

让我们以Redis DEL命令为例。如果键被删除，它将给出输出1，否则它将为0。

```
SET javatpoint redis #设置键值
GET javatpoint       #根据key获取值
DEL javatpoint  	 #删除键值
```

| 序号 |                          命令及描述                          |
| :--: | :----------------------------------------------------------: |
|  1   | [DEL key](https://www.redis.com.cn/commands/del) 该命令用于在 key 存在时删除 key。 |
|  2   | [DUMP key](https://www.redis.com.cn/commands/dump) 序列化给定 key ，并返回被序列化的值。 |
|  3   | [EXISTS key](https://www.redis.com.cn/commands/exists) 检查给定 key 是否存在。 |
|  4   | [EXPIRE key](https://www.redis.com.cn/commands/expire) seconds 为给定 key 设置过期时间，以秒计。 |
|  5   | [EXPIREAT key timestamp](https://www.redis.com.cn/commands/expireat) EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 |
|  6   | [PEXPIRE key milliseconds](https://www.redis.com.cn/commands/pexpire) 设置 key 的过期时间以毫秒计。 |
|  7   | [PEXPIREAT key milliseconds-timestamp](https://www.redis.com.cn/commands/pexpireat) 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计 |
|  8   | [KEYS pattern](https://www.redis.com.cn/commands/keys) 查找所有符合给定模式( pattern)的 key 。 |
|  9   | [MOVE key db](https://www.redis.com.cn/commands/move) 将当前数据库的 key 移动到给定的数据库 db 当中。 |
|  10  | [PERSIST key](https://www.redis.com.cn/commands/persist) 移除 key 的过期时间，key 将持久保持。 |
|  11  | [PTTL key](https://www.redis.com.cn/commands/pttl) 以毫秒为单位返回 key 的剩余的过期时间。 |
|  12  | [TTL key](https://www.redis.com.cn/commands/ttl) 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
|  13  | [RANDOMKEY](https://www.redis.com.cn/commands/randomkey) 从当前数据库中随机返回一个 key 。 |
|  14  | [RENAME key newkey](https://www.redis.com.cn/commands/rename) 修改 key 的名称 |
|  15  | [RENAMENX key newkey](https://www.redis.com.cn/commands/renamenx) 仅当 newkey 不存在时，将 key 改名为 newkey 。 |
|  16  | [TYPE key](https://www.redis.com.cn/commands/type) 返回 key 所储存的值的类型。 |

### Redis字符串(Strings)

| 指数 |              命令              |                         描述                          |
| :--: | :----------------------------: | :---------------------------------------------------: |
|  1   |         SET key value          |              此命令用于设置指定键的值。               |
|  2   |            GET key             |                此命令用于检索键的值。                 |
|  3   |     GETRANGE key start end     |     此命令用于获取存储在键中的字符串的子字符串。      |
|  4   |        GETSET key value        |       此命令用于设置键的字符串值并返回其旧值。        |
|  5   |       GETBIT key offset        | 此命令用于返回存储在key的字符串值中的偏移量处的位值。 |
|  6   |      MGET key1 [key2 ..]       |             此命令用于获取所有给定键的值              |
|  7   |    SETBIT key offset value     |   此命令用于设置或清除存储在key的字符串值中的偏移位   |
|  8   |    SETEX key secodes value     |              此命令用于设置key到期时的值              |
|  9   |        SETNX key value         |        仅当key不存在时，此命令用于设置key的值         |
|  10  |   SETRANGE key offset value    |   此命令用于覆盖从指定偏移量开始的键处的字符串部分    |
|  11  |           STRLEN key           |          此命令用于检索存储在key中的值的长度          |
|  12  |  MSET key value [key value …]  |            此命令用于将多个键设置为多个值             |
|  13  | MSETNX key value [key value …] | 仅当没有任何键存在时，此命令用于将多个键设置为多个值  |
|  14  | PSETEX key milliseconds value  |    此命令用于设置key的值和到期时间（以毫秒为单位）    |
|  15  |            INCR key            |              此命令用于将键的整数值递增1              |
|  16  |      INCRBY key increment      |           此命令用于按给定量递增键的整数值            |
|  17  |   INCRBYFLOAT key increment    |          此命令用于按给定的量增加键的浮点值           |
|  18  |            DECR key            |              此命令用于将键的整数值递减1              |
|  19  |      DECRBY key decrement      |          此命令用于按给定数量递减键的整数值           |
|  20  |        APPEND key value        |                此命令用于将值附加到键                 |

### Redis哈希(Hashes)



Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。每个哈希键中可以存储多达40亿个字段值对。

```
#设置hash
HMSET javatpoint name "A solution to all Technology" description "India's fastest growing tutorial website" daily 1 million visitors 10 millions page visit.   
#获取hash
HGETALL javatpoint 
```

| 指数 |                  命令                   |                  描述                  |
| :--: | :-------------------------------------: | :------------------------------------: |
|  1   |        HDEL key field2 [field2]         |        删除一个或多个哈希字段。        |
|  2   |            HEXISTS key field            |         确定是否存在哈希字段。         |
|  3   |             HGET key field              |   获取存储在指定键中的哈希字段的值。   |
|  4   |               HGETALL key               | 获取存储在指定键的散列中的所有字段和值 |
|  五  |       HINCRBY key field increment       |     按给定数字递增散列字段的整数值     |
|  6   |    HINCRBYFLOAT key field increment     |      将散列字段的浮点值递增给定量      |
|  7   |                HKEYS key                |          获取哈希中的所有字段          |
|  8   |                HLEN key                 |           获取散列中的字段数           |
|  9   |           HMGET key1 [field2]           |        获取所有给定哈希字段的值        |
|  10  | HMSET key field1 value1 [field2 value2] |       将多个哈希字段设置为多个值       |
|  11  |             HSET key field              |         设置哈希字段的字符串值         |
|  12  |            HSETNX key field             |   仅当字段不存在时，设置哈希字段的值   |
|  13  |                HVALS key                |          获取哈希值中的所有值          |

### Redis列表(Lists)

Redis列表是按插入顺序排序的字符串列表。可以在列表的头部（左边）或尾部（右边）添加元素。

列表可以包含超过40亿个元素。

**例**

```
redis 127.0.0.1:6379> LPUSH javatpoint sql  
(integer) 1  
redis 127.0.0.1:6379> LPUSH javatpoint mysql  
(integer) 2  
redis 127.0.0.1:6379> LPUSH javatpoint cassandra  
(integer) 3  
redis 127.0.0.1:6379> LRANGE javatpoint 0 10  
1) "cassandra"  
2) "mysql"  
3) "sql"  
redis 127.0.0.1:6379>  
```

| 指数 |                 命令                  |                             描述                             |
| :--: | :-----------------------------------: | :----------------------------------------------------------: |
|  1   |       BLPOP key1 [key2] timeout       |    删除和获取列表中的第一个元素，或阻塞直到一个元素可用。    |
|  2   |       BRPOP key1 [key2] timeout       |   删除和获取列表中的最后一个元素，或阻塞直到一个元素可用。   |
|  3   | BRPOPLPUSH source destination timeout | 从列表中弹出一个值，将其推送到另一个列表并返回它; 或阻止，直到有一个可用。 |
|  4   |           LINDEX key index            |                  通过索引从列表中获取元素。                  |
|  5   | LINSERT key before\|after pivot value |           在列表中的另一个元素之前或之后插入元素。           |
|  6   |               LLEN key                |                       获取列表的长度。                       |
|  7   |               LPOP key                |                删除和获取列表中的第一个元素。                |
|  8   |       LPUSH key value1 [value2]       |                 将一个或多个值添加到列表中。                 |
|  9   |           LPUSHX key value            |              仅当列表存在时，将值添加到列表中。              |
|  10  |         LRANGE key start stop         |                   从列表中获取一系列元素。                   |
|  11  |         LREM key count value          |                      从列表中删除元素。                      |
|  12  |         LSET key index value          |                 通过索引设置列表中元素的值。                 |
|  13  |         LTRIM key start stop          |                    将列表修剪到指定范围。                    |
|  14  |               RPOP key                |               删除和获取列表中的最后一个元素。               |
|  15  |     RPOPLPUSH source destination      |   删除列表中的最后一个元素，将其附加到另一个列表并返回它。   |
|  16  |       RPUSH key value1 [value2]       |                  将一个或多个值附加到列表。                  |
|  17  |           RPUSHX key value            |             仅当列表存在时，用于将值附加到列表。             |

### Redis集合(Sets)

Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中没有重复的数据。

在Redis中，添加、删除和查找的时间复杂都是O(1)（不管Set中包含多少元素）。

集合中最大的成员数为 232 – 1 (4294967295, 每个集合可存储40多亿个成员)

```
redis 127.0.0.1:6379> SADD javatpoint db2  
(integer) 1  
redis 127.0.0.1:6379> SADD javatpoint mongodb  
(integer) 1  
redis 127.0.0.1:6379> SADD javatpoint db2  
(integer) 0  
redis 127.0.0.1:6379> SADD javatpoint cassandra  
(integer) 1  
redis 127.0.0.1:6379> SMEMBERS javatpoint  
1) "cassandra"  
2) "db2"  
3) "mongodb"  
```

| 指数 |                      命令                      |                         描述                          |
| :--: | :--------------------------------------------: | :---------------------------------------------------: |
|  1   |           SADD key member1 [member2]           |            将一个或多个成员添加到集合中。             |
|  2   |                   SCARD key                    |                获取集合中的成员数量。                 |
|  3   |               SDIFF key1 [key2]                |               返回给定所有集合的差集。                |
|  4   |       SDIFFstore destination key1 [key2]       |    返回给定所有集合的差集并存储在 destination 中。    |
|  5   |               SINTER key1 [key2]               |                    返回集合交集。                     |
|  6   |      SINTERSTORE destination key1 [key2]       |    返回给定所有集合的交集并存储在 destination 中。    |
|  7   |              SISMEMBER key member              |        判断 member 元素是否是集合 key 的成员。        |
|  8   |        SMOVE source destination member         | 将 member 元素从 source 集合移动到 destination 集合。 |
|  9   |                    SPOP key                    |           移除并返回集合中的一个随机元素。            |
|  10  |            SRANDMEMBER key [count]             |             返回集合中一个或多个随机数。              |
|  11  |           SREM key member1 [member2]           |              移除集合中一个或多个成员。               |
|  12  |               SUNION key1 [key2]               |               返回所有给定集合的并集。                |
|  13  |      SUNIONSTORE destination key1 [key2]       |     所有给定集合的并集存储在 destination 集合中。     |
|  14  | SSCAN key cursor [match pattern] [count count] |                  迭代集合中的元素。                   |

### Redis有序集合(Sorted Sets)

Redis 有序集合和集合一样也是string类型元素的集合，且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 – 1 (4294967295, 每个集合可存储40多亿个成员)。

```
redis 127.0.0.1:6379> ZADD javatpoint 1 redis  
(integer) 0  
redis 127.0.0.1:6379> ZADD javatpoint 2 cassandra  
(integer) 1  
redis 127.0.0.1:6379> ZADD javatpoint 3 cassandra  
(integer) 0  
redis 127.0.0.1:6379> ZADD javatpoint 3 mysql  
(integer) 1  
redis 127.0.0.1:6379> ZADD javatpoint 4 mysql  
(integer) 0  
redis 127.0.0.1:6379> ZRANGE javatpoint 0 10 WITHSCORES  
1) "redis"  
2) "1"  
3) "cassandra"  
4) "3"  
5) "mysql"  
6) "4" 
```

| 序号 |                      命令                      |                             描述                             |
| :--: | :--------------------------------------------: | :----------------------------------------------------------: |
|  1   |    ZADD key score1 member1 [score2 member2]    |    向有序集合添加一个或多个成员，或者更新已存在成员的分数    |
|  2   |                   ZCARD key                    |                     获取有序集合的成员数                     |
|  3   |               ZCOUNT key min max               |             计算在有序集合中指定区间分数的成员数             |
|  4   |          ZINCRBY key increment member          |         有序集合中对指定成员的分数加上增量 increment         |
|  5   |  ZINTERSTORE destination numkeys key [key …]   | 计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中 |
|  6   |             ZLEXCOUNT key min max              |            在有序集合中计算指定字典区间内成员数量            |
|  7   |       ZRANGE key start stop [WITHSCORES]       |          通过索引区间返回有序集合成指定区间内的成员          |
|  8   |  ZRANGEBYLEX key min max [LIMIT offset count]  |                通过字典区间返回有序集合的成员                |
|  9   | ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] |             通过分数返回有序集合指定区间内的成员             |
|  10  |                ZRANK key member                |                 返回有序集合中指定成员的索引                 |
|  11  |           ZREM key member [member …]           |                移除有序集合中的一个或多个成员                |
|  12  |           ZREMRANGEBYLEX key min max           |            移除有序集合中给定的字典区间的所有成员            |
|  13  |         ZREMRANGEBYRANK key start stop         |            移除有序集合中给定的排名区间的所有成员            |
|  14  |          ZREMRANGEBYSCORE key min max          |            移除有序集合中给定的分数区间的所有成员            |
|  15  |     ZREVRANGE key start stop [WITHSCORES]      |     返回有序集中指定区间内的成员，通过索引，分数从高到底     |
|  16  |   ZREVRANGEBYSCORE key max min [WITHSCORES]    |      返回有序集中指定分数区间内的成员，分数从高到低排序      |
|  17  |              ZREVRANK key member               | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
|  18  |               ZSCORE key member                |                  返回有序集中，成员的分数值                  |
|  19  |  ZUNIONSTORE destination numkeys key [key …]   |    计算给定的一个或多个有序集的并集，并存储在新的 key 中     |
|  20  | ZSCAN key cursor [MATCH pattern] [COUNT count] |        迭代有序集合中的元素（包括元素成员和元素分值）        |















## Redis是什么?

Redis 通常被称为数据结构服务器，因为值(value)可以是字符串(String), 哈希(Map), 列表(list), 集合(sets) 或有序集合(sorted sets)等类型。

Redis 是互联网技术中使用最为广泛的中间件之一。

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

![Redis数据类型2](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161421.png)

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

![Redis数据类型3](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161418.png)

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

![Redis数据类型4](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161415.png)

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

![Redis数据类型5](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161412.png)

## Redis 事务

### 事务介绍

事务是指“一个完整的动作，要么全部执行，要么什么也没有做”。Redis 事务不是严格意义上的事务，只是用于帮助用户在一个步骤中执行多个命令。单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

Redis 事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

MULTI、EXEC、DISCARD、WATCH 这四个指令构成了 redis 事务处理的基础。

1.MULTI 用来组装一个事务；
2.EXEC 用来执行一个事务；
3.DISCARD 用来取消一个事务；
4.WATCH 用来监视一些 key，一旦这些 key 在事务执行之前被改变，则取消事务的执行。

在Redis中，通过使用“MULTI”命令启动事务，然后需要传递应在事务中执行的命令列表，之后整个事务由“EXEC”命令执行。

![Redis交易1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161406.png)
![Redis交易2](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161432.png)

### 例子

让我们举一个例子来看看如何启动和执行Redis事务。

```
redis 127.0.0.1:6379> MULTI  
OK  
redis 127.0.0.1:6379> EXEC  
(empty list or set)  
redis 127.0.0.1:6379> MULTI  
OK  
redis 127.0.0.1:6379> SET javatpoint redis  
QUEUED  
redis 127.0.0.1:6379> GET javatpoint  
QUEUED  
redis 127.0.0.1:6379> INCR visitors  
QUEUED  
redis 127.0.0.1:6379> EXEC  
1) OK  
2) "redis"  
3) (integer) 1  
```

![Redis Transactions 3](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101161436.png)

在上面的例子中，我们看到了QUEUED的字样，这表示我们在用MULTI组装事务时，每一个命令都会进入到内存队列中缓存起来，如果出现QUEUED则表示我们这个命令成功插入了缓存队列，在将来执行EXEC时，这些被QUEUED的命令都会被组装成一个事务来执行。

对于事务的执行来说，如果redis开启了AOF持久化的话，那么一旦事务被成功执行，事务中的命令就会通过write命令一次性写到磁盘中去，如果在向磁盘中写的过程中恰好出现断电、硬件故障等问题，那么就可能出现只有部分命令进行了AOF持久化，这时AOF文件就会出现不完整的情况，这时，我们可以使用redis-check-aof工具来修复这一问题，这个工具会将AOF文件中不完整的信息移除，确保AOF文件完整可用。

------

### Redis 事务错误

有关事务，大家经常会遇到的是两类错误：

1.调用EXEC之前的错误
2.调用EXEC之后的错误

“调用EXEC之前的错误”，有可能是由于语法有误导致的，也可能时由于内存不足导致的。只要出现某个命令无法成功写入缓冲队列的情况，redis都会进行记录，在客户端调用EXEC时，redis会拒绝执行这一事务。（这是2.6.5版本之后的策略。在2.6.5之前的版本中，redis会忽略那些入队失败的命令，只执行那些入队成功的命令）。我们来看一个这样的例子：

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> haha //一个明显错误的指令
(error) ERR unknown command 'haha'
127.0.0.1:6379> ping
QUEUED
127.0.0.1:6379> exec
//redis无情的拒绝了事务的执行，原因是“之前出现了错误”
(error) EXECABORT Transaction discarded because of previous errors.
```

而对于“调用EXEC之后的错误”，redis则采取了完全不同的策略，即redis不会理睬这些错误，而是继续向下执行事务中的其他命令。这是因为，对于应用层面的错误，并不是redis自身需要考虑和处理的问题，所以一个事务中如果某一条命令执行失败，并不会影响接下来的其他命令的执行。我们也来看一个例子：

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 23
QUEUED
//age不是集合，所以如下是一条明显错误的指令
127.0.0.1:6379> sadd age 15 
QUEUED
127.0.0.1:6379> set age 29
QUEUED
127.0.0.1:6379> exec //执行事务时，redis不会理睬第2条指令执行错误
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get age
"29" //可以看出第3条指令被成功执行了
```

最后，我们来说说最后一个指令“WATCH”，这是一个很好用的指令，它可以帮我们实现类似于“乐观锁”的效果，即CAS（check and set）。

WATCH本身的作用是“监视key是否被改动过”，而且支持同时监视多个key，只要还没真正触发事务，WATCH都会尽职尽责的监视，一旦发现某个key被修改了，在执行EXEC时就会返回nil，表示事务无法触发。

```
127.0.0.1:6379> set age 23
OK
127.0.0.1:6379> watch age //开始监视age
OK
127.0.0.1:6379> set age 24 //在EXEC之前，age的值被修改了
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 25
QUEUED
127.0.0.1:6379> get age
QUEUED
127.0.0.1:6379> exec //触发EXEC
(nil) //事务无法被执行
```

### Redis事务命令

以下是Redis事务的一些基本命令的列表。

| 序号 | 命令及描述                                                   |
| :--: | :----------------------------------------------------------- |
|  1   | [DISCARD](https://www.redis.com.cn/commands/discard) 取消事务，放弃执行事务块内的所有命令。 |
|  2   | [EXEC](https://www.redis.com.cn/commands/exec) 执行所有事务块内的命令。 |
|  3   | [MULTI](https://www.redis.com.cn/commands/multi) 标记一个事务块的开始。 |
|  4   | [UNWATCH](https://www.redis.com.cn/commands/unwatch) 取消 WATCH 命令对所有 key 的监视。 |
|  5   | [WATCH key [key …\]](https://www.redis.com.cn/commands/watch) 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。 |

## Redis脚本

**Redis脚本**



Redis脚本使用Lua解释器来执行脚本。

自版本2.6.0开始内嵌于Redis中。

用于编写脚本的命令是EVAL。

**句法**

```
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...]  
```

**例**

让我们举一个例子来看看Redis脚本的工作原理：

```
redis 127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1   
key2 first second    
1) "key1"   
2) "key2"   
3) "first"   
4) "second"
```

## Redis连接

Redis连接命令用于控制和管理到Redis Server的客户端连接。

### 例

以下示例说明客户端如何向Redis服务器验证自身并检查服务器是否正在运行。

```
redis 127.0.0.1:6379> AUTH "password"  
(error) ERR Client sent AUTH, but no password is set  
redis 127.0.0.1:6379>  
redis 127.0.0.1:6379> PING  
PONG  
redis 127.0.0.1:6379>  
```

![Redis Connections 1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101162157.png)

#### 注意：在这里您可以看到未设置“密码”，因此您可以直接访问任何命令。

------

## Redis连接命令

以下是Redis数据库中使用的一些基本连接命令的列表：

| 序号 |     命令      |                描述                |
| :--: | :-----------: | :--------------------------------: |
|  1   | AUTH password | 使用给定密码对服务器进行身份验证。 |
|  2   | ECHO message  |         打印给定的字符串。         |
|  3   |     PING      |      检查服务器是否正在运行。      |
|  4   |     QUIT      |           关闭当前连接。           |
|  5   | SELECT index  |      更改当前连接的选定数据库      |



## Redis服务器

Redis Server命令用于管理Redis服务器。

有不同的服务器命令可用于获取服务器信息，统计信息和其他特征。

### 例

我们举一个例子来看看如何获取有关服务器的所有统计信息和信息。

```
redis 127.0.0.1:6379> ping  
PONG  
redis 127.0.0.1:6379> AUTH "password"  
(error) ERR Client sent AUTH, but no password is set  
redis 127.0.0.1:6379> PING  
PONG  
redis 127.0.0.1:6379> ECHO "Welcome to Javatpoint"  
"Welcome to Javatpoint"  
redis 127.0.0.1:6379> INFO  
redis_version:2.4.6  
redis_git_sha1:26cdd13a  
redis_git_dirty:0  
arch_bits:64  
multiplexing_api:winsock2  
gcc_version:4.6.1  
process_id:6360  
uptime_in_seconds:4442  
uptime_in_days:0  
lru_clock:1716856  
used_cpu_sys:1.80  
used_cpu_user:0.42  
used_cpu_sys_children:0.00  
used_cpu_user_children:0.00  
connected_clients:1  
connected_slaves:0  
client_longest_output_list:0  
client_biggest_input_buf:0  
blocked_clients:0  
used_memory:1188152  
used_memory_human:1.13M  
used_memory_rss:1188152  
used_memory_peak:1188112  
used_memory_peak_human:1.13M  
mem_fragmentation_ratio:1.00  
mem_allocator:libc  
loading:0  
aof_enabled:0  
changes_since_last_save:0  
bgsave_in_progress:0  
last_save_time:1506142039  
bgrewriteaof_in_progress:0  
total_connections_received:1  
total_commands_processed:4  
expired_keys:0  
evicted_keys:0  
keyspace_hits:0  
keyspace_misses:0  
pubsub_channels:0  
pubsub_patterns:0  
latest_fork_usec:0  
vm_enabled:0  
role:master  
```

![Redis发布服务器1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101162516.png)

------

### Redis服务器命令

以下是Redis数据库中使用的一些服务器命令的列表：

| 序号 |                    命令                    |                             描述                             |
| :--: | :----------------------------------------: | :----------------------------------------------------------: |
|  1   |                BGREWRITEAOF                |                     异步重写仅附加文件。                     |
|  2   |                   BGSAVE                   |                   将数据集异步保存到磁盘。                   |
|  3   |    CLIENT KILL [ip:port] [ID client-id]    |                      终止客户端的连接。                      |
|  4   |                CLIENT LIST                 |                 获取服务器的客户端连接列表。                 |
|  5   |               CLIENT GETNAME               |                     获取当前连接的名称。                     |
|  6   |            CLIENT PAUSE timeout            |            在指定时间内停止处理来自客户端的命令。            |
|  7   |       CLIENT SETNAME connection-name       |                      设置当前连接名称。                      |
|  8   |               CLUSTER SLOTS                |                    获取集群节点的映射数组                    |
|  9   |                  COMMAND                   |                获取Redis命令详细信息的数组。                 |
|  10  |               COMMAND COUNT                |                    获取Redis命令的总数。                     |
|  11  |              COMMAND GETKEYS               |           给定完整的Redis命令，此命令用于提取key。           |
|  12  |                   BGSAVE                   |                    数据集异步保存到磁盘。                    |
|  13  | COMMAND INFO command-name [command-name …] |              获取特定Redis命令详细信息的数组。               |
|  14  |            CONFIG GET parameter            |                 此命令用于获取配置参数的值。                 |
|  15  |               CONFIG REWRITE               |             此命令用于使用内存配置重写配置文件。             |
|  16  |         CONFIG SET parameter value         |               此命令用于获取给定值的配置参数。               |
|  17  |              CONFIG RESETSTAT              |              此命令用于重置INFO返回的统计信息。              |
|  18  |                   DBSIZE                   |              此命令用于返回所选数据库中的键数。              |
|  19  |              DEBUG OBJECT key              |              此命令用于获取有关密钥的调试信息。              |
|  20  |               DEBUG SEGFAULT               |                   此命令用于使服务器崩溃。                   |
|  21  |                  FLUSHALL                  |            此命令用于从所有数据库中删除所有密钥。            |
|  22  |                  FLUSHDB                   |             此命令用于从当前数据库中删除所有键。             |
|  23  |                  MONITOR                   |          此命令用于获取有关服务器的信息和统计信息。          |
|  24  |                  LASTSAVE                  |        此命令用于检索上次成功保存到磁盘的UNIX时间戳。        |
|  25  |                  MONITOR                   |           此命令用于实时侦听服务器收到的所有请求。           |
|  26  |                    ROLE                    |           此命令用于在复制上下文中返回实例的角色。           |
|  27  |                    SAVE                    |              此命令用于将数据集同步保存到磁盘。              |
|  28  |          SHUTDOWN [NOSAVE] [SAVE]          |      此命令用于将数据集同步保存到磁盘，然后关闭服务器。      |
|  29  |             SLAVEOF host port              | 此命令用于使服务器成为另一个实例的从属服务器，或将其作为主服务器提升。 |
|  30  |       SLOWLOG subcommand [argument]        |               此命令用于管理Redis慢查询日志。                |
|  31  |                    SYNC                    |                       此命令用于复制。                       |
|  32  |                    TIME                    |                此命令用于返回当前服务器时间。                |



## Redis备份和恢复

SAVE命令用于创建当前Redis数据库的备份。此命令将通过执行同步SAVE在Redis目录中创建dump.rdb文件。

**语法**

SAVE

**返回值**

执行成功后，SAVE命令返回OK。

------

### Redis备份示例

使用SAVE命令创建当前数据库的备份。

SAVE

![Redis备份1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101162842.png)

它将在Redis目录中创建dump.rdb文件。

可以看到dump.rdb文件已创建。

![Redis备份2](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101162848.png)

### 还原Redis数据

将Redis备份文件（dump.rdb）移动到Redis目录中并启动服务器以恢复Redis数据。

查找Redis的安装目录，使用Redis的CONFIG命令，如下所示。

![Redis备份3](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101162854.png)

Redis服务器安装在以下目录中。

“/var/lib/redis”

------

### BGSAVE命令

BGSAVE是创建Redis备份的备用命令。

此命令将启动备份过程并在后台运行。

**语法**

BGSAVE

**例**

![Redis备份4](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101162859.png)

## Redis安全



对于数据库来说，安全性是非常必要的，以确保数据的安全性。它提供身份验证，因此如果客户端想要建立连接，则需要在执行命令之前进行身份验证。

您需要在配置文件中设置密码以保护Redis数据库。

### 例

我们来看看如何保护您的Redis实例。

使用“config get command”

config get requirepass

![Redis安全1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163016.png)

您可以看到上面的属性为空，表示我们没有此实例的任何密码。您可以通过执行以下命令来更改此属性并为此实例设置密码。

config set requirepass “javatpoint123”

![Redis安全2](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163023.png)

CONFIG get requirepass

![Redis安全3](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163027.png)

当您设置此密码时，如果客户端在未经身份验证的情况下运行该命令，则会收到错误“NOAUTH Authentication required。”。因此，客户端需要使用AUTH命令来验证自己。

------

### AUTH命令的用法



```
127.0.0.1:6379> AUTH "javatpoint123"  
OK  
127.0.0.1:6379> SET mykey "hindi100" 
OK  
127.0.0.1:6379> GET mykey  
"hindi100"  
127.0.0.1:6379>  
```



![Redis安全4](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163038.png)

## Redis 基准测试

Redis基准测试redis-benchmark是一种实用工具，用于通过同时使用multiple(n)命令来检查Redis的性能。

**句法**

```
redis-benchmark [option] [option value]
```

### 例

调用Redis Benchmark 命令：

redis-benchmark -n 100000

![Redis基准1](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163153.png)
![Redis基准2](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163215.png)

redis 性能测试工具可选参数如下所示：

| 序号 |   选项   |                    描述                    |  默认值   |
| :--: | :------: | :----------------------------------------: | :-------: |
|  1   |  **-h**  |              指定服务器主机名              | 127.0.0.1 |
|  2   |  **-p**  |               指定服务器端口               |   6379    |
|  3   |  **-s**  |             指定服务器 socket              |           |
|  4   |  **-c**  |               指定并发连接数               |    50     |
|  5   |  **-n**  |                 指定请求数                 |   10000   |
|  6   |  **-d**  |   以字节的形式指定 SET/GET 值的数据大小    |     2     |
|  7   |  **-k**  |          1=keep alive 0=reconnect          |     1     |
|  8   |  **-r**  | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
|  9   |  **-P**  |         通过管道传输 <numreq> 请求         |     1     |
|  10  |  **-q**  |    强制退出 redis。仅显示 query/sec 值     |           |
|  11  | **–csv** |              以 CSV 格式输出               |           |
|  12  |  **-l**  |           生成循环，永久执行测试           |           |
|  13  |  **-t**  |      仅运行以逗号分隔的测试命令列表。      |           |
|  14  |  **-I**  |  Idle 模式。仅打开 N 个 idle 连接并等待。  |           |

### 实例

以下实例我们使用了多个参数来测试 redis 性能：主机为 127.0.0.1，端口号为 6379，执行的命令为 set,lpush，请求数为 10000，通过 -q 参数让结果只显示每秒执行的请求数。

```
redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 100000 -q
```

![Redis基准3](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201101163158.png)























###