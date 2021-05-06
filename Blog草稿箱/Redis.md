# 数据存储发展历程

**1. 单机MySQL时代**

<img align='left' src="img/Redis.img/image-20210331100509079.png" alt="image-20210331100509079" style="zoom:50%;" />

这种模式下的瓶颈：

- 数据量太大，一个机器存放不下
- 数据的索引太大，一个机器的内存放不下
- 访问量（读写混合）太大，一个服务器承受不住  

 **2. 缓存 Memcached+读写分离  **

网站上80%的情况都是在读，每次都去查询数据库，效率很低。这时候可以加入缓存机制，第一次查询去MySQL中读取数据，将数据返回给用户的同时，在缓存中也存储下来。第二次访问，就可以直接从缓存中读取

<img align='left' src="img/Redis.img/image-20210331100742247.png" alt="image-20210331100742247" style="zoom:50%;" />  















**3. 分库分表 + 水平拆分（MySQL集群）**  

<img align='left' src="img/Redis.img/image-20210331100844656.png" alt="image-20210331100844656" style="zoom:50%;" />

但是随着社会的发展，要存储数据的类型（音乐，视频，地理位置，人际交往圈、用户自己产生的数 据，用户日志等）也越来越繁多，数据量也爆发式增长。这样MySQL等关系型数据库就越来越不够用 了！NoSQL数据库就开始进入人们的视野！NoSQL数据库可以很好的解决这些问题。

# NoSQL简介

NoSQL（Not Only SQL） 泛指非关系型数据库

**NoSQL特点：**

- 方便扩展（数据之间没有关系）
- 大数据量高性能（Redis写8w/s, 读11w/s，NoSQL的缓存记录级是一种细粒度的，性能会更高）
- 数据类型是多样型的！ 不需要事先设计数据库，随取随用
- 存储方式多样， 键值对，列存储，文档存储，图形数据库
- 没有固定的查询语言

**NoSQL的四大分类：**

- KV键值对：新浪 Redis  ；美团 Redis+Tair  ；阿里 百度： Redis+Memcached  
- 文档型数据库 bson格式：MongoDB是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档；MongoDB是一个介于关系型数据库和非关系型数据库的中间产品。MongoDB是非关系型数据库
  中功能最丰富，最像关系型数据库的  
- 列存储数据库：HBase  ；分布式文件系统 GFS  
- 图关系数据库：他不是存图片的，存储的是关系，比如：朋友圈社交网络、广告推荐！Neo4j， infoGrid  

**四种分类的比较**

<img align='left' src="img/Redis.img/image-20210331101614810.png" alt="image-20210331101614810" style="zoom:50%;" />

# Redis简介

> Redis是一种开放源代码（BSD许可）的内存中数据结构存储，用作数据库，缓存和消息代理。Redis提供数据结构，例如==字符串，哈希，列表，集合，带范围查询的排序集合，位图，超日志，地理空间索引和流==。Redis具有内置的复制，Lua脚本，LRU驱逐，事务和不同级别的磁盘持久性，并通过Redis Sentinel哨兵和Redis Cluster自动分区提供了高可用性。

redis首先是一个强大的缓存服务器，比memcache强大很多，不仅仅支持多种数据结构（不像memcache只能存储字符串）如字符串、list列表、set集合、map映射表等结构，还可以支持数据的持久化存储（memcache只支持内存存储），经常被应用到高并发的服务器环境设计之中  

**Redis、memcached、MySQL与PostgreSQL比较**

![image-20210331102057409](img/Redis.img/image-20210331102057409.png)

# Redis是单线程的

Redis是很快的，官方表示，Redis是基于内存操作的，CPU不是Redis的性能瓶颈，Redis的瓶颈就是根据机器的内存和网络带宽。既然可以使用单线程来实现，就使用单线程了！

Redis是C语言实现的，官方数据：读：110000/s 写： 80000/s，完全不比同样使用key-value的Memcached差  

**Redis为什么单线程还这么快？**  

核心： <font color='blue'>**Redis是将所有的数据全部放在内存中的，所以说使用单线程去操作效率就是最高的，相比多线程，减少了CPU上下文切换的耗时**</font>。对于内存系统来说，没有上下文切换效率就是最高的，多次读写都是在一个CPU上的  

- 纯内存操作，避免大量访问数据库，减少直接读取磁盘数据，redis将数据储存在内存里面，读写数据的时候都不会受到硬盘 I/O 速度的限制，所以速度快；
- 单线程操作，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；
- 采用了非阻塞I/O多路复用机制

> redis的读写是单线程的，总体的redis-server是多线程的，因为redis它还要做其他事情
>
> ![image-20210317201231824](img/Redis.img/image-20210317201231824.png)

# Redis基本命令

Redis默认有16个数据库，默认使用的是第0个数据库，可以通过select切换数据库（Redis的命令同SQL语句一样对大小写不敏感）

**数据库相关命令**

```shell
#启动客户端
redis-cli

#启动客户端
redis-cli -p 6379 #-p指定端口号

#切换数据库
select 3

#查看当前数据库中键keys的个数（元素个数）
dbsize

#清空当前数据库
flushdb

#清空所有数据库
flushall
```

**Redis键值对，键key相关命令**

```shell
#查看当前数据库所有的key
keys *

#设置key
set stu wang

#查看key的值
get stu

#设置key的过期时间(秒)
expire stu 10

#查看key的过期时间
ttl stu

#判断key是否存在
exists stu

#删除key
del stu

#查看key对应的value的数据类型
type stu
```

**帮助命令**

```shell
help
help 命令  #help set
help @类型 #help @string
```

# Redis的5个基本类型

redis中的数据都是以key/value的形式存储的，五大数据类型主要是指value的数据类型

## 字符串string

```shell
#设置值
set stu wang

#获取值
get sty

#判断key是否存在
exists stu

#追加字符串，如果key不存在，相当于set命令，返回值为追加之后的strlen
append stu jiaxin

#获取字符串的长度
strlen key1

#对指定key的value自增1/自减1，value要为整数值
incr key
decr key

#设置增长/减少的步长
incr key 2
decr key 3

#获取给定范围的字符串，要获取全部传0到-1
getrange key1 0 -1

#从指定位置开始替换字符串的值
setrange key1 2 xxx  #将key1的value从下标2开始替换为xxx

#设置key1的值为hhh并设置过期时间为5s
setex key1 5 hhh

#设置key的值，如果key不存在则设置失败，返回0
setnx key value

#同时设置多个key的值
mset key value [key value...]

#同时获取多个key的值
mset key1 key2 ...

#同时设置多个key的值，一个设置失败则全部失败（原子操作）
msetnx key value [key value...]
```

**用string存储对象时key值的设计**

```shell
#设置一个user:1对象，值为Json字符串
set  user:1  {name:wang,age:20}

#设置user:1的名字为wang，user:1的年龄为20
mset user:1:name wang user:1:age 20

#获取user:1的名字、年龄
mget user:1:name user:1:age

#获取key的value，然后key的value设置为newvalue
getset key newvalue
```

**string类似的使用场景**

- value除了是字符串也可以是数字！  
- 计数器：文章浏览量，点赞数
- 统计多单位的数量  
- 对象缓存存储  

## 列表list

在Redis中，我们可以用list完成栈、队列、阻塞队列  

```shell
# 链表左边插入值
lpush key value

#链表右边插入值
rpush key value

#获取指定范围的值
lrange key start stop

#从列表的左边或者右边移除值
lpol key 
rpop key

#获取key对应的value链表指定下标的值
lindex key index

#获取列表中元素的个数
llen key

#移除列表中的元素
lrem key count value 
lrem stu 1 wang #移除list中的1个wang

#将链表修剪/截取到指定范围， 通过下标截取指定的长度，这个list已经改变了，只剩下截取的元素！
ltrim stu 1 2

#移除列表中最后一个元素，将它添加到另一个列表中
rpoplpush key1 key2

#根据下标替换列表中的值
lset key index value

#在列表中插入值
linsert key before|after value new_value
```

在两边插入或者改动值，效率最高！中间元素，相对来说效率会低一点  

队列（头删尾插）：LPOP RPUSH  

栈    （栈顶出入）：LPOP LPUSH  

## 集合set

redis里面set是无序的，C++里面set是有序的，C++的unordered_set是无序set

set中的值是不能重复的

```shell
#给set中添加值
sadd key value1 value2 ...

#获取set中的所有值
smembers key

#判断某个值是否在set中
sismember key value

#获取set中元素的个数
scard key

#删除set中的值
srem key value1 value2....

#从set中获取随机值
srandmember key

#从set中获取count个随机值
srandmember key count

#随机删除指定个数个元素
spop key count

#将指定的元素从一个set中移动到另一个set中
smove set1 set2 value
```

**数字集合类**

差集、交集、并集

抖音中，A用户将所有关注的人放在一个set集合中，将他的粉丝放在一个集合中，通过这几个运算可以实现共同关注，共同爱好，二度好友（推荐好友）等  

```shell
#差集
sdiff set1 set2 

#交集 共同还有可以通过这个实现
sinter set1 set2 

#并集
SUNION set1 set2 
```

## 哈希hash

可以将哈希看成是一个Map集合，==key-value中的value是一个map集合，key - {field1-value1,field2-value2}==

```shell
#设置一个hash的值
hset key field1 value1 

#获取一个hash的值
hget key field

#设置或者获取多个fhash的值
hmset key field1 value1  field2 value2 ... 
hmget key field1 field2...

#获取hash中的所有值
hgetall key

#删除指定field的hash键值对
hdel key field

#获取hash的键值对的个数
hlen key

#判断hash中的字段是否存在
hexists key field

#获取一个hash中的所有fields
hkeys key

#获取一个hash中国的所有value
hvals value

#给hash中指定字段的值加上一个增量
hincrby key field 2 

#如果不存在，则添加，如果存在，则失败
hsetnx key field value
```

**hash的应用**

hash中存储经常变更的值：比如用户信息： user ： name-value，age-value，sex-value

hash更适合对象的存储，String更加适合字符串存储  

## 有序集合zset

```shell
#添加一个值
zadd key scores value
zadd myset 1 wang #添加一个值
zadd myset 2 yang 3 zhang #添加多个值

#获取zset中一个范围的值
zrange key start stop

#将zset中的值按照score从小到大排序输出
zrangebyscore key min max

#移除zset中指定的元素
zrem key value

#查看zset中的元素个数
zcard key

#根据score的值统计在给定区间的元素个数
zcount key min max
```

# Redis的三种特殊类型  

## geospatial 地理空间  

geo的底层就是一个zset集合，所以zset的命令可以应用于geoadd的key

```shell
#获取zset中一个范围的值
zrange china:city 0 -1

#移除zset中指定的元素
zrem china:city shanghai
```

```shell
#添加地理位置
geoadd key 纬度 经度 名称
GEOADD china:city 116.40 39.90 beijing

#返回给定名称的纬度和经度
geopos key 名称
GEOPOS china:city xian

#返回两个给定位置之间的距离，单位：米m，千米km，英里mi，英尺ft
geodist key city1 city2
GEODIST china:city beijing xian km

#返回指定元素的纬度和经度的字符串
geohash key city
GEOHASH china:city xian

#以给定的纬度经度为中心，找到某一半径内的元素
georadius key 经度 维度 半径 单位m|km|ft|mi

#以一个成员为中心，查找指定半径范围内容的元素
georadiusbymember key city 半径 单位
```

## hyperloglog基数统计

A{1,3,5,7,8,7}	B{1,3,5,7,8}

A和B的基数（不重复的元素个数）= 5， 可以接受一定的误差！  

Redis Hyperloglog 是基数统计的算法。0.81%的错误率！  

优点： 占用的内存是固定的，2^64不同的元素计数，只需要12KB的内容空间。

<img align='left' src="img/Redis.img/image-20210331133417668.png" alt="image-20210331133417668" style="zoom:67%;" />

## bitmaps位图

bitmaps是位图存储的，都是二进制位来进行记录， 所以只要是只有两种状态值的场景，都可以使用bitmaps来存储。比如：登录、未登录；打卡，未打卡；活跃，不活跃等  

```shell
#在bitmaps中添加数据
setbit key offset value
SETBIT sign 0 1
SETBIT sign 1 1
SETBIT sign 2 1
SETBIT sign 3 0

#查看位图中某个位置的值
getbit key offset

#统计位图中value等于1的个数
bitcount key start end
```

# 事务

事务的本质： 一组命令的集合！一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺
序执行！一次性，顺序性，排他性，执行一系列的命令！

MySQL中的事务，要么同时成功，要么同时失败，必须保证原子性！

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- **开始事务：multi**  
- **命令入队**
- **执行事务：exec**

所有的命令在事务中，并没有直接被执行！只有发起执行命令的时候才会执行！  

```shell
multi      #开启事务
set k1 v1  #命令入队
set k2 v2  #命令入队
get k2     #命令入队列
exec       #执行事务
```

**discard：**放弃事务

```shell
multi      #开启事务
set k1 v1  #命令入队
set k2 v2  #命令入队
get k2     #命令入队列
discard    #放弃事务：事务队列中的命令都不会执行
```

**编译型异常（命令有错）：**事务中所有的命令都不会被执行  

```shell
multi
set k1 v1
getset k2  #命令不正确
set k2 v2
exec       #执行事务时，其他命令也不会执行
```

**执行时异常：**如果事务队列中存在语法性错误，执行命令的时候，其他命令可以正常执行，错误
命令抛出异常  

```shell
set k1 "v1"
multi 
incr k1  #执行的时候会失败，因为k1是字符串类型
set k2 y2
set k3 y3
get k2
exec     #执行事务时，incr命令执行出错，其他命令依旧会执行
```

# Redis的乐观锁、Watch  

**悲观锁：**很悲观，认为什么时候都会出问题，无论做什么都会加锁！但是影响效率！  

**乐观锁：**很乐观，认为平常时候不会出问题，所以不会提前加锁！更新数据时去判断一下，在此期间是否有人
修改过这个数据！MySQL的version的使用：先获取version，更新数据时比较version，看version是否被修改  

**watch：**监视给定的键以确定MULTI/EXEC块是否执行，其他线程在watch期间没有修改指定键，则执行事务，否则不执行事务

<img src="img/Redis.img/image-20210404222519503.png" alt="image-20210404222519503" style="zoom:50%;" />

watch监视下的事务执行失败后，可以unwatch解锁，在重新开始监视（不解锁，上个事务失败也会自动解锁）

# hiredis

hiredis是Redis官方推荐的基于C接口的客户端组件，它提供接口，供c语言调用以操作数据库。  

```cpp
//头文件
#include <hiredis/hiredis.h>

//连接redis数据库，参数为ip地址和端口号，默认端口6379.该函数返回一个redisContext对象
redisContext *redisConnect(const char *ip, int port);

//执行redis命令，返回redisReply对象
void *redisCommand(redisContext *c, const char *format, ...);

//释放redisCommand执行后返回的RedisReply对象
void freeReplyObject(void *reply);

//断开redisConnect所产生的连接
void redisFree(redisContext *c);

//redisReply对象结构如下：
typedef struct redisReply
{
	int type; // 返回结果类型
	long long integer; // 返回类型为整型的时候的返回值
	size_t len; // 字符串的长度
	char *str; // 返回错误类型或者字符串类型的字符串
	size_t elements; // 返回数组类型时，元素的数量
	struct redisReply **element; // 元素结果集合
}redisReply;

//返回类型有以下几种：
REDIS_REPLY_STRING   1   //字符串
REDIS_REPLY_ARRAY    2   //数组，多个reply，通过element数组以及elements数组大小访问
REDIS_REPLY_INTEGER  3   //整型
REDIS_REPLY_NIL      4   //空，没有数据
REDIS_REPLY_STATUS   5   //状态，str字符串以及len
REDIS_REPLY_ERROR    6   //错误，同STATUS
```

**C程序操作Redis代码示例：**

```cpp
#include <hiredis/hiredis.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <assert.h>
#include <stdlib.h>


int main()
{
    //连接redis
    redisContext *c  = redisConnect("127.0.0.1", 6379);  
    assert(c != NULL);

    //要执行的redis命令
    char * com = "hmset user:1 name wangjiaxin age 21";  

    //执行命令
    redisReply * r = (redisReply*)redisCommand(c, com);  
    assert(r != NULL);

    //执行命令后获得的结果集的类型
    if(r->type == REDIS_REPLY_STATUS) 
    {
        printf("%s\n", r->str);
        printf("success\n");
    }
    else
    {
        printf("fail\n");
    }

    com = "hgetall user:1";
    r = (redisReply*)redisCommand(c, com);

    if(r->type == REDIS_REPLY_ARRAY)  //数组，多个reply，通过element数组以及elements数组大小访问
    {
        int i = 0;
        for(; i < r->elements; ++i)
        {
            if((i+1) % 2 == 0)
            {
                printf("%s\n", r->element[i]->str);
            } 
            else
            {
                printf("%s ", r->element[i]->str);
            }
        }
    }        
    freeReplyObject(r); // 释放redisReply对象
    redisFree(c); // 断开连接
    return 0;
}
```

![image-20210404232501577](img/Redis.img/image-20210404232501577.png)

# Redis的持久化

Redis是内存数据库，如果不将内存中的数据库状态保存到磁盘中，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以Redis提供了持久化的功能  

## **RDB （Redis DataBase）**  

<img align='left' src="img/Redis.img/image-20210405103029498.png" alt="image-20210405103029498" style="zoom:50%;" />

在指定的时间间隔内将内存中的数据集快照写入磁盘，也就是Snapshot快照，它恢复时是将快照文件直接读到内存中  

Redis会单独创建（fork）一个子进程来进程持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上一次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进程大规模数据的恢复，且对数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。  

rdb保存的文件是**dump.rdb**  

**触发rdb的机制**

- save的规则满足的情况下，会自动触发rdb操作  
- 执行flushall命令，会触发rdb操作  
- 退出redis服务器时，也会触发rdb操作  

**如何恢复rdb文件**

- 只需要将rdb文件放在redis的启动目录下就可以。redis启动时会自动检查dump.rdb文件恢复其中的数
  据  
- 获取路径配置参数：config get dir   #如果该目录下存在dump.rdb文件，启动就会根据这个文件恢复数据  

**优点：**

- 适合大规模的数据恢复  
- 对数据的完整性要求不高  

**缺点：**

- 需要一定的时间间隔进行操作，如果redis意外宕机，最后一次持久化后的操作数据就没有了
- fork进程的时候，会需要占用一定的内存空间

## AOF（Append Only File）

**将所有==写操作==的命令都记录下来，类似history，恢复时将所有的命令都执行一遍！**  

AOF保存的文件是appendonly.aof  

如果这个aof文件被恶意修改，这个时候redis是启动不起来的，我们需要修复这个aof文件！redis提供的一个工具 redis-check-aof --fix 来修改aof文件  

**优点：**

- 每一次修改都同步，文件的完整性比较好  
- 每秒同步一次，可能会丢失1s的数据  

**缺点：**

- 相对于数据文件来说，aof远远大于rdb  
- 修复的速度比rdb慢  
- AOF的运行效率也比rdb慢

# Redis发布订阅

Redis发布订阅（pub/sub）是一种 消息通信模式 ：发布者（pub）发送消息，订阅者(sub)接受消
息。

应用： 微信、抖音等的关注系统！

Redis客户端可以订阅任意数量的频道。  

**发布订阅模型**

<img align='left' src="img/Redis.img/image-20210405104816516.png" alt="image-20210405104816516" style="zoom:50%;" />

下图展示了频道channel1，以及订阅这个频道的三个客户端 -- client2 client5和client1之间的关系：  当有新消息通过PUBLISH命令发送给频道channel1时，这个消息就会被发送给订阅他的三个客户端：  

![image-20210405105046517](img/Redis.img/image-20210405105046517.png)

**相关命令**

```shell
#订阅给定的一个或多个频道的信息
subscribe channel [channel...]

#订阅一个或多个符合给定模式的频道
psubscribe pattern[pattern...]

#查看订阅与发布系统状态
PUBSUB subcommand [argument [argument...]]

#将信息发送到指定的频道
publish channel message

#退订所有给定模式的频道
punsubscribe [pattern [pattern ...]]

#退订给定的频道
ubsubscribe [channel [channel ...]]
```

**示例**

![image-20210405110200541](img/Redis.img/image-20210405110200541.png)

**发布订阅模式底层原理**

Redis是使用C实现的，通过分析Redis源码里的pubsub.c文件，了解发布和订阅机制的底层实现，借此加深对Redis的理解。  

Redis通过PUBLISH、SUBSCRIBE和PSUBSCRIBE等命令实现发布和订阅功能。  

==通过SUBSCRIBE命令订阅某频道后==：redis-server里维护一个字典，字典的键就是一个个channel，而字典的值则是一个链表，链表中保存了所有订阅这个channel的客户端。**SUBSCRIBE命令的关键，就是将客户端添加到给定channel的订阅链表中**  

==通过PUBLISH命令向订阅者发送消息==：redis-server会使用给定的频道作为键，在它所维护的channel字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。  

Pub/Sub从字面上理解就是发布（Publist）与订阅(Subscribe)，在Redis中，可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅他的客户端都会受到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。  

**应用场景**

- 实时消息系统  
- 实时聊天 -- 频道当做聊天室，将信息回显给所有人即可  
- 消息队列中间件  

# Redis主从复制

**主从复制：**指的是将一个Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点（master/leader),后者称为从节点（slave/follower）。 数据的复制是单向的 ，只能从主节点到从节点。==Master以写为主，Slave以读为主==。

默认情况下，每台Redis服务器都是主节点，且一个主节点可以有多个从节点（或者没有从节点），但一个从节点只能有一个主节点。  

**主从复制的作用：**

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式  
2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复，实际上是一种服务冗余
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 （即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载。尤其是在写少读多的场景下，通过从节点分担读负载，可以大大提供Redis服务器的并发量。    
4. 高可用(集群)基石：除了上述作用以外，主从复制还是哨兵和集群能够实现的基础，因此说主从复制是Redis高可用的基础  

**一般来说，将Redis运用于工程项目中，只使用一台Redis是万万不可能的，原因如下：** 

1. 从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大
2. 从容量上，单个Redis服务器内存容量有限，就算一台Redis服务器内存容量为256G，也不能将所有内存用作Redis存储内存，一般来说，单台Redis最大使用内存不应该超过20G

**电商网站上的商品，一般都是一次上传，无数次浏览，也就是“==多读少写==”。对于这种场景，可以使用如下的架构：**

<img align='left' src="img/Redis.img/image-20210405111259240.png" alt="image-20210405111259240" style="zoom: 43%;" />  















**环境配置**

```shell
info replication  #查看当前库的信息
```

<img align='left' src="img/Redis.img/image-20210405111746564.png" alt="image-20210405111746564" style="zoom:50%;" />

复制3个配置文件，修改对应的信息：

1. 端口号

2. pid名字

3. log文件名字

4. dump.rdb名字

修改完之后，启动3个redis服务器，可以通过进程信息查看：  

```shell
redis-server /etc/redis/1.conf
redis-server /etc/redis/2.conf
```

<img align='left' src="img/Redis.img/image-20210405214331869.png" alt="image-20210405214331869" style="zoom:50%;" />

**默认情况下，每台Redis服务器都是主节点 ，主需要配置从机就行！  主要有两种配置模式：**

- 一主二从
- 链路模式：一主后面链从结点
- <img align='left' src="img/Redis.img/image-20210405112209147.png" alt="image-20210405112209147" style="zoom:43%;" />

认老大！一主（6379）二从（6380,6381），启动`redis-cli -p 6380`与`redis-cli -p 6381`输入如下命令：

```shell
slaveof 127.0.0.1 6379 #SLAVEOF host port 找老大

info replication #查看主机信息
```

```shell
slaveof no one #使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器
```

![image-20210405220522546](img/Redis.img/image-20210405220522546.png)

**主机可以写，从机不能写只能读！主机中的所有信息和数据，都会自动保存到从机中！**  

![image-20210405220245691](img/Redis.img/image-20210405220245691.png)

主机断开连接，从机依旧连接到主机的，但是没有写操作，这个时候，主机如果回来了，从机依旧可以
直接获取到主机写的信息  

**复制原理**

Slave从机启动成功连接到Master主机后会向Master发送一个sync同步命令  

Master接受到命令后，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，Master将传送 整个数据文件(全量复制) 到Slave，并完成一次完全同步

- 全量复制 ：Slave服务在接受到数据库文件数据后，将其存盘并加载到内存中  
- 增量复制 ：Master继续将新的所有收集到的修改命令一次传给Slave，完成同步  

只要是重新连接Master，一次完全同步（全量复制）将被自动执行

**从机恢复变成主机：**从机执行 SLAVEOF no one 可以重新恢复为主机    

## 哨兵模式

==自动选举老大的模式！==

主从切换技术的方法是：**当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力**，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。Redis从2.8开始正式提供了Sentinel（哨兵）架构来解决这个问题  

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个单独的进程，作为进程，他会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例**  

<img align='left' src="img/Redis.img/image-20210405112943299.png" alt="image-20210405112943299" style="zoom:43%;" />

**哨兵的作用**  

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器  
- ==当哨兵监测到Master宕机，会自动将Slave切换成Master，然后通过发布/订阅模式通知其他的从服务器，修改配置文件，让它切换主机 === 

**多哨兵模式**

一个哨兵进程对Redis服务器进程监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式  

<img align='left' src="img/Redis.img/image-20210405113129278.png" alt="image-20210405113129278" style="zoom:45%;" />

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover进程，仅仅是哨兵1主观的认为主服务器不可用，这个现象称为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover[故障转移]操作。切换成功后，就会通过发布/订阅模式，让各个哨兵把自己监视的从服务器实现切换主机，这个过程称为**客观下线**  

**配置哨兵模式**

- 配置哨兵配置文件 sentinel.conf

  ```shell
  #sentinel monitor 主机名称 host port 1
  sentinel monitor myredis 127.0.0.1 6379 1
  
  # 最后面的1，代表主机挂了，Slave投票看谁接替成为主机，票数最多的，就会成为主机
  ```

- 启动哨兵

  ```shell
  redis-sentinel sentinel.conf
  # 如果Master节点断开，这个时候就会从从机中随机选择一个转换为主机！
  # 如果之前的主机重新启动后，只能归并到新的主机下，成为新主机的从机  
  ```

**哨兵模式的优点**

- 哨兵集群，基于主从复制模式，所有的主从复制优点，它全有  
- 主从可以切换，故障可以转移，系统的可用性会更好  
- 哨兵模式就是主从模式的升级，手动转自动，更加健壮  

**哨兵模式的缺点**

- Redis不好在线扩展，集群容量一旦到达上限，在线扩容十分麻烦  
- 实现哨兵模式的配置很麻烦，里面有很多选择  

# Redis缓存穿透和雪崩

> Redis缓存的使用，极大的提高了应用程序的性能和效率，特别是数据查询等。但同时，它也带来了一些问题。其中，最主要的问题就是数据一致性，从严格意义上来讲，这个问题是无解的。如果对数据一致性要求很高，那么就不能使用缓存。
>
> 另外一个典型的问题就是：内存穿透，内存击穿和内存雪崩问题。目前，业界也都有比较流行的解决
> 方案。  

## 缓存穿透—查不到

缓存穿透的就是用户想要查询一个数据，发现Redis中没有，也就是缓存没有命中，于是向持久层数据库发起查询，发现也没有这个数据，于是本次查询失败。**当用户很多的情况下，缓存都没有命中，又都去请求持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于缓存穿透。**  

<img align='left' src="img/Redis.img/image-20210405222440892.png" alt="image-20210405222440892" style="zoom:40%;" />

**解决方案**

==方案一：布隆过滤器==：布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合的则丢弃，从而避免对持久层数据库的查询压力。  

<img align='left' src="img/Redis.img/image-20210405222729690.png" alt="image-20210405222729690" style="zoom:40%;" />

==方案二：缓存空对象==：当存储层不命中后，及时返回的空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源。

<img align='left' src="img/Redis.img/image-20210405222918481.png" alt="image-20210405222918481" style="zoom:40%;" />  













但是这种方法会存在两个问题：

- 如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键
- 即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这
  对于需要保持一致性的业务会有影响  

## 缓存击穿—量太大，缓存过期  

这里需要注意和缓存穿透的区别，缓存击穿，是指一个key非 常热点。在不停的扛着大量并发，大并发集中对这一个点进行访问，**当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在也给屏蔽上凿开了一个洞。**  

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并且写会缓存，会导致数据库瞬间压力过大  

**解决方案**

==方案一：热点数据永不过期==：从缓存层面来看，没有设置过期时间，所以不会出现热点key过期后产生的问题  

==方案二：加互斥锁==：使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验比较大  

## 缓存雪崩—缓存集中过期失效

**缓存雪崩指的是在某一时间段，缓存集中过期失效，或者Redis宕机。**

比如：在写文本的时候，马上要到双十一零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就过期了。从而对这批商品的访问查询，都落到了后台数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调
用量会暴增，造成存储层也会宕机的情况  

<img align='left' src="img/Redis.img/image-20210405223454537.png" alt="image-20210405223454537" style="zoom:40%;" />

其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务器节点的宕机，对数据服务器造成的压力是不可预估的，很可能瞬间就把数据库压宕机  

**解决方案**

==方案一：redis高可用==：这个思想就是说既然Redis有可能挂掉，我就多增设几台Redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建缓存服务器集群。（异地多活）  

==方案二：限流降级==：这个思想就是在缓存失效后，通过加锁或者队列来控制读写数据库的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。  

==方案三：数据预热==：数据预热的含义就是在正式部署之前，先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中，在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。  

# Redis的过期时间和过期删除机制

> [Redis的过期时间和过期删除机制](https://www.cnblogs.com/itplay/p/10162935.html)

**设置过期时间**

redis有四种命令可以用于设置键的生存时间和过期时间：

```shell
EXPIRE <KEY> <TTL> : 将键的生存时间设为 ttl 秒
PEXPIRE <KEY> <TTL> :将键的生存时间设为 ttl 毫秒
EXPIREAT <KEY> <timestamp> :将键的过期时间设为 timestamp 所指定的秒数时间戳
PEXPIREAT <KEY> <timestamp>: 将键的过期时间设为 timestamp 所指定的毫秒数时间戳.
```

**保存过期时间**

那么redis里面对这些key的过期时间和生存时间的信息是怎么保存的呢？？
答：在数据库结构redisDb中的expires字典中保存了数据库中所有键的过期时间，我们称expire这个字典为过期字典。

- 过期字典是一个指针，指向键空间的某个键对象。
- 过期字典的值是一个longlong类型的整数，这个整数保存了键所指向的数据库键的过期时间–一个毫秒级的 UNIX 时间戳

**移除过期时间**

PERSIST 命令可以移除一个键的过期时间：persist命令就是expire命令的反命令，这个函数在过期字典中查找给定的键,并从过期字典中移除。

```cpp
127.0.0.1:6379> set message "hello"
OK
127.0.0.1:6379> expire message 60
(integer) 1
127.0.0.1:6379> ttl message
(integer) 54
127.0.0.1:6379> persist message
(integer) 1
127.0.0.1:6379> ttl message
(integer) -1
```

**过期键的删除策略**

- 立即删除（对CPU不友好）：在设置键的过期时间时，创建一个回调事件，当过期时间达到时，由时间处理器自动执行键的删除操作。
- 惰性删除（浪费内存）：键过期了就过期了，不管。每次从dict字典中按key取值时，先检查此key是否已经过期，如果过期了就删除它，并返回nil，如果没过期，就返回键值。
- 定时删除。每隔一段时间，对expires字典进行检查，删除里面的过期键

第一、三种为主动删除，第二种为被动删除

redis使用的过期键值删除策略是：惰性删除加上定期删除，两者配合使用

# Redis的内存淘汰策略

> [彻底弄懂Redis的内存淘汰策略](https://zhuanlan.zhihu.com/p/105587132)
>
> <img src="img/Redis.img/image-20210506164117887.png" alt="image-20210506164117887" style="zoom:57%;" />

内存满了，下一次再插入的时候，就要淘汰掉一些已有的

- 1、noeviction：内存满了返回错误，不淘汰任何键
- 2、allkeys-lru：所有键通过LRU算法淘汰
- 3、volatile-lru：首先从设置了过期时间的键中使用LRU算法淘汰
- 4、allkeys-random：从所有键中随机删除
- 5、volatile-random：从过期键的集合中随机删除
- 6、volatile-ttl：淘汰马上要过期的键
- 7、volatile-lfu：过期键中淘汰使用频率最少的键
- 8、allkeys-lfu：从所有键中淘汰使用频率最少的键

**LRU**：Least Recently Used，最近最久未使用页面置换算法，如果数据最近被访问过，那么将来被访问的几率也更高，选择最近最久未使用的页面予以淘汰

<img align='left' src="img/Redis.img/image-20210506164348297.png" alt="image-20210506164348297" style="zoom:40%;" />

**LFU：**Least frequently used，最不经常使用，如果一个数据在最近一段时间内使用次数很少，那么在将来一段时间内被使用的可能性也很小。

