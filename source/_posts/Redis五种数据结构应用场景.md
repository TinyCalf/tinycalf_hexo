---
title: Redis五种数据结构应用场景
date: 2019-08-19 19:27:17
tags:
    - Redis
categories: Redis
author: TinyCalf
---
> Redis有五种数据结构，分别是: String/Hash/List/Set/Zset；使用这五种数据结构就能够完成很多应用，所以有必要简单总结一下，作为解决问题的一个索引。

<!-- more -->
## 字符串 String
String是最常用的数据类型，常用命令有以下这些：
set get del mset mget incr decr incrbyfloat append strlen setrange getrange；
在使用redis的各种命令时，除了关注用法以外还需要注意的就是时间复杂度，这些再redis官网都可以找到，我就不再赘述了；
redis命令文档：https://redis.io/commands；
set是设置键，是最常用的，有几个选项：
* ex设置秒级过期时间
* px毫秒级过期时间
* nx键必须不存在，用于添加
* xx键必须存在，用于更新

另有setnx和setex代替nx和ex两个参数。
set可以设置过期时间是redis可以作为缓存使用的最重要的特性；
mset和mget用于批量设置值和获取值；网络是Redis的瓶颈，需要尽量减少网络的访问次数；
字符串有以下几个典型应用场景：
**缓存功能** 最常用的功能，将热数据的查询结果缓存到redis；
**计数** 也是减少数据库压力的方法之一，比如文章的点击量再一分钟内在redis中自增，每分钟写一次Mysql，清空redis；
**共享session** 这个在分布式或者负载均衡的web服务中非常常用，用户被负载均衡到不同服务器上的时候并不希望session丢失而重新登录；
可以用redis做一个session管理的服务；
同时还可以规定用户需要重新登录的时间间隔；
**限速** 比如短信接口，我们不希望用户能无限访问，可以用string可以设置过期时间的特性；类似其他ip访问限制也同理
## 哈希 Hash
Hash我觉得是最不常用的，简单了解一下，常用命令有这些：
hset hget hdel hlen hmget hmset hexists hkeys hvals hgetall hincrby hstrlen；
Hash的典型使用场景：
比如我们想缓存用户数据，可以将数据库中的用户数据的每一列在哈希结构里表示出来，更新较为灵活；
但是，好像没太大用，因为用string也可以，只不过节约了一些内存。哈希的应用不算丰富。
## 列表 List
List由于特性比较丰富，用法也非常多，先来看看常用命令：
添加 rpush lpush linsert；
查 lrange lindex llen；
删除 lpop rpop lrem ltrim；
修改 lset；
阻塞操作 blpop brpop；
列表是有序的，并且在列表的两侧都可以push和pop，因此lpush + lpop就形成了一个栈；
lpush + rpop就形成了一个队列；
还有一种特殊的pop形式，就是阻塞弹出brpop，这个特殊的弹出其实就是为消息队列而准备的，可以防止一个数据被多次消费；
因此lpush+brpop可以作为消息队列；
ltrim可以指定索引范围并保留，删除其他元素，因此配合lpush可以作为一个有限集合，比如需求是只想保留100条数据，在第101条插入时删除第一条。
## 集合 Set
集合的特性也比较丰富：
集合中不会存在重复元素；
集合元素是无序的；
集合内可以增删改查；
可以取多个集合的交并差集；
集合有以下常用命令：
sadd srem scarf sismember；
srandmember 返回指定个数的元素；
spop 随机弹出多个元素；
smembers 获取所有元素；
sinter 交集；
sunion 并集；
sdiff 差集；
在使用交并差集是，在元素比较多的情况下非常耗时，所以sinter sunion sidff有destination参数，可以将结果保存到新键中；
集合的能实现的最贴合的功能可能就是标签了，比如每个用户有不用的兴趣标签，每个标签有不用的用户；
使用sinter就可以计算出两个用户兴趣的交集，这在社交软件中非常常用；
spop可以从集合中随机弹出元素，srandmember可以随机取出一个元素，但是不删除这个元素，这两个命令都可以实现不用需求的抽奖系统；
部分应用中有靓号系统，也可以通过集合实现，可以srandmember多个靓号以供选择，spop删除那个被选中的靓号，sismember确定系统随机生成的数是不是靓号，以免被下发；
## 有序集合  Zset
Zset区别于Set最大的特点是可以给每个元素设置分数，作为排序的依据；
这里就有必要整理一下列表、集合、和有序集合的异同点了：

| 数据结构 | 是否允许重复元素 | 是否有序 | 有序实现方式 | 应用场景 |
| ----------|-------------------|-----------|---------------|-----------|
| 列表 | 是 | 是 | 索引下标 | 时间轴 消息队列等|
| 集合 | 否 | 否 | 无 | 标签 社交等 |
| 有序集合 | 否 | 是 | 分值 | 排行榜 社交 |

有序集合常用命令如下：
zadd nx必须存在用于添加 xx必须不存在 用于更新 ch返回受影响的个数  incr 增加分数
zscore 获取分数
zcard 获取成员个数
zrank  获取排名
zrevrank 反向排名 withscore参数可以同时返回分数
zrem zincrby zrangebyscore 返回指定范围内的成员
zrevrangebyscore
zinterscore 这个命令用于取交集，但是参数非常多：
destination 交集计算结果保留到这个键；
numkeys 需要做交集计算键的个数；
key 需要做交集甲酸的键； 
wieghts 每个键的权重； 
agreegate sum|min|max 汇总形式；
可以参考一下redis的文档更详细了解这个命令；
当然还有zunionstore取并集；
使用场景就是各种排行榜系统啦：
zadd在增加条目的同时可以给条目分配一个分数；
zincrby可以增加某个条目的分数；
zrank\zrevrank等可以查看排名前几位或者后几位的条目。
## 总结
在使用基本数据结构时，有两个点需要注意：
**命令的时间复杂度**：因为Redis是单线程，时间过长的查询会阻塞其他任务；
**数据结构的编码**：虽然Redis会自动根据情况切换数据编码，但是要优化Redis还是要了解数据编码的形式和设置合理的参数；
下面我整理一个表，大致表示一下基本数据结构能实现的功能：

| 数据结构| 命令 | 功能 |
| -----| -----| ----|
| String| 几乎所有命令 | 大部分的缓存任务 热数据访问缓存 高频写入缓存 | 
| Hash | -| - |
| List | lpush + lpop | 栈 |
| List | lpush + rpop | 队列 | 
| List | lpush + brpop | 消息队列 |
| List | lpush + ltrim | 有序集合 |
| Set |  sadd | 标签 |
| Set | spop/srandmember | 抽奖系统 |
| Set | sadd + sinter | 社交需求 |
| Zset | 许多常用命令 | 排行榜系统 |
