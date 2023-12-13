# Redis

# 使用docker安装启动

```shell
docker pull redis
docker run -itd --name myRedis -p 6379:6379 redis
#测试是否启动Redis服务
docker exec -it myRedis redis-cli ping #启动响应 PONG
```

# 客户端

## 命令行客户端

```shell
redis-cli 
	-h ip默认127.0.0.1
	-p port默认6379
	-a 密码
```

## 图形化客户端

**Datagrip**

# 数据结构

## key格式

可用  **:**   连接做到分层和避免重复（**[项目名]:[业务名]:[类型]:[id]**）

## 通用命令

- KEYS

> 获取匹配的键，性能影响大不建议使用

```shell
KEYS * #获取所有键
*表示通配任意个数		？表示通配一个
```

- DEL

> 删除匹配的键，返回删除数量

- EXISTS

> 判断匹配的键是否存在

- EXPIRE

> 给一个key设置有效期，有效期到期时该key会被自动删除，单位秒

- TTL

> 查看一个key的剩余有效期（-1永久有效，-2过期）

## String

> value类型有string，int，float

- SET key value	                             

   设置键值对

- MSET key value [key2 value2...]  

  设置多个键值对

- GET key                                            

  获取key的value

- MGET key [key2...]                             

  获取多个key的value

- INCR key 

  相应的整数value+1

- INCRBY key num

  相应的整数value+num

- INCRBYFLOAT key num

  相应的浮点数value+num

- SETNX key value

  设置键值对，需key不存在

- SETEX  key time value

  设置键值对，有效期为time秒

## Hash类型

> value类型为散列map，对应的结构为key，field，value

- HSET key field value [field2 value2...]  

  设置key的[多个]field的值

- HGET key field                                           

  获取key的field的值

- HMGET key field [field2...]                             

  获取key的多个field的值

- HGETALL key

  获取key的所有field和value

- HKEYS

  获取key的所有filed

- HVALS

  获取key的所有value

- HINCRBY key filed num

​		相应的整数value+num

- SETNX key filed value

  设置键值对，需filed不存在

## List类型

- LPUSH key element... 

  向链表左侧插入一个或多个元素

- RPUSH key element... 

  向链表右侧插入一个或多个元素

- LPOP key n

  移除并返回列表左侧的n个元素，没有则返回nil

- RPOP key n

  移除并返回列表右侧的n个元素，没有则返回nil

- LRANGE key star end

  返回一段下标范围内的所有元素（下标从0开始，闭区间）

- BLPOP/BRPOP key time

  与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil

## Set类型

> 基于散列，value为集合

- SADD key element ...

  向set中添加一个或多个元素

- SREM key element...

  移除set中的指定元素

- SCARD key

  返回set中元素的个数

- SISMEMBER key element

  判断一个元素是否存在子set中

- ﻿SMEMBERS key

  获取set中的所有元素

- SINTER key1 key2...

  求key1与key2的交集

- SDIFF key1 key2...

  求key1与key2的差集（key1中有key2中没有）

- SUNIONkey1key2..

  求key1和key2的并集

## ZSet类型

> 有序集合
>
> 所有顺序为升序，如果要降序则在命令的Z后面添加REV即可

- ZADD key score element...

  添加一个或多个元素到sorted set，如果已经存在则更新其score值

- ﻿﻿ZREM key element

  删除sorted set中的一个指定元素

- ﻿﻿ZSCORE key element 

  荻取sorted set中的指定元素的score値

- ZRANK key element

   荻取sorted set 中的指定元素的排名

- ﻿﻿ZCARD key

  荻取sorted set中的元素个数

- ﻿﻿ZCOUNT key min max

  统计在给score范围内的所有元素的个数

- ﻿﻿ZINCRBY key num element

  让sorted set中的指定元素自增，步长为指定的num值

- ﻿ZRANGE key min max

  获取指定**排名**范围内的元素

- ﻿ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

  获取指定score范围内的元素

- ﻿ZDIFF、ZINTER、ZUNION

  求差集、交集、并集

## Stream类型

> 用于消息队列

- XADD key *|ID filed value [field2 value2...]

  发送消息（*表示自动生成ID，否则ID需要为 `时间戳-递增ID `格式）

- XREAD [COUNT 读取个数] [BLOCK 阻塞时间(ms)] STREAMS key ID [ID2...]  (0表示第一条，$表示最新一条)

  读消息

- XLEN key

  查看消息个数

- XGROUP CREATE key groupName ID 

  创建消费者组

- XREADGROUP GROUP groupName 消费者名 [COUNT 读取个数] [BLOCK 阻塞时间(ms)] STREAMS key ID [ID2...]（>表示下一个未消费的消息，其他是从pending-list读）

  读消息，读了未确认的消息会进入pending-list）

- XACK key groupName ID

  确认消息

- XPENDING key groupName - + count

  查看pending-list中的消息