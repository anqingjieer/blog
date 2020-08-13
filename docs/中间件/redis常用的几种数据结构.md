# redis中常用的几种数据结构

## 一、sql和NoSql之间的区别

### 1.传统关系型数据库的优缺点？

#### 优点：

- 它以表格的形式，基于行存储数据，是一个二维的模式。
- 它存储的是结构化的数据，数据存储有固定的模式（schema），数据需要适应表结构。
- 表与表之间存在关联（Relationship）
- 大部分关系型数据库都支持SQL（结构化查询语言）的操作，支持复杂的关联查
- 大部分关系型数据库都支持SQL（结构化查询语言）的操作，支持复杂的关联查
  询。
- 通过支持事务（ACID 酸）来提供严格或者实时的数据一致性。

#### 缺点：

- [x] 1、要实现扩容的话，只能向上（垂直）扩展，比如磁盘限制了数据的存储，就要扩
  大磁盘容量，通过堆硬件的方式，不支持动态的扩缩容。水平扩容需要复杂的技术来实
  现，比如分库分表。
  2、表结构修改困难，因此存储的数据格式也受到限制。
  3、在高并发和高数据量的情况下，我们的关系型数据库通常会把数据持久化到磁盘，
  基于磁盘的读写压力比较大。

### 2.关系型数据库的优缺点？

### 优点

1. 纯内存操作，操作速度快

2. 单线程操作，减少线程上下文切换带来的开销

3. 存储非结构化的数据，比如文本、图片、音频、视频。

4. 表与表之间没有关联，可扩展性强。

5. 、保证数据的最终一致性。遵循BASE（碱）理论。Basically Available（基本
   可用）； Soft-state（软状态）； Eventually Consistent（最终一致性）
   
6. 支持分布式，能够对数据进行分片存储，扩缩容简单。

   ### 3.redis的安装和使用

   #### 3.1安装的话推荐官网的命令

   #### 3.2.基本操作

   默认有16个库，可以在配置文件中进行修改，默认使用第一个db0.

    可以通过命令进行切换。

   ```c
   select （）
   ```

   常用的命令。

   ```c++
   flushdb //清空当前数据库
   flushall //清空所有的数据
    keys *  //显示key值
   dbsize //一共有多少个数据
   exists qingshan   //key是否存在
       del  qignshan   //删除
   ```

   Redis 是字典结构的存储方式，采用key-value 存储。key 和value 的最大长度限制是512。

   ------

   

## 二、redis的基本数据类型

> 常用的基本数类型有 **String、Hash、Set、List、Zset**、Hyperloglog、Geo、Streams

### 1.string类型

#### 	            存储类型：可以    存字符串、浮点型类型、整数类型

#### 1）、操作命令

```c
///对不存在的值进行
redis> SET key "value"
OK
redis> GET key
"value"
//对存在的值进行覆盖
redis> SET key "new-value"
OK
redis> GET key
"new-value"
```

#### 2）、存储实现原理

字符串类型的内部编码有三种：

1、int，存储8 个字节的长整型（long，2^63-1）。
2、embstr, 代表embstr 格式的SDS（Simple Dynamic String 简单动态字符串），存储小于44 个字节的字符串。
3、raw，存储大于44 个字节的字符串（3.2 版本之前是39 字节）



#### 	3）、redis中字符串的设计

> set hello word 为例，因为Redis 是KV 的数据库，它是通过hashtable 实现的（我们把这个叫做外层的哈希）。所以每个键值对都会有一个dictEntry（源码位置：dict.h），里面指向了key 和value 的指针。next 指向下一个dictEntry。
>
> ```c++
> typedef struct dictEntry {
> void *key; /* key 关键字定义*/
> union {
> void *val; uint64_t u64; /* value 定义*/
> int64_t s64; double d;
> } v;
> struct dictEntry *next; /* 指向下一个键值对节点*/
> } dictEntry;
> ```
>
> 
>
> ![image-20200531203259543](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603171009.png)
>
> key 是字符串，但是Redis 没有直接使用C 的字符数组，而是存储在自定义的SDS中。value 既不是直接作为字符串存储，也不是直接存储在SDS 中，而是存储在redisObject 中。实际上五种常用的数据类型的任何一种，都是通过redisObject 来存储
> 的。
>
> redisObject
>
> ```c++
> typedef struct redisObject {
> unsigned type:4; /* 对象的类型，包括：OBJ_STRING、OBJ_LIST、OBJ_HASH、OBJ_SET、OBJ_ZSET */
> unsigned encoding:4; /* 具体的数据结构*/
> unsigned lru:LRU_BITS; /* 24 位，对象最后一次被命令程序访问的时间，与内存回收有关*/
> int refcount; /* 引用计数。当refcount 为0 的时候，表示该对象已经不被任何对象引用，则可以进行垃圾回收了
> */
> void *ptr; /* 指向对象实际的数据结构*/
> } robj;
> ```
>
> 可以使用type命令查看数据类型
>
> type key

##### 1.什么是SDS？

​               redis中字符串的实现。

#####  2.为什么Redis 要用SDS 实现字符串？

​				我们知道，C 语言本身没有字符串类型（只能用字符数组char[]实现）。

1、使用字符数组必须先给目标变量分配足够的空间，否则可能会溢出。
2、如果要获取字符长度，必须遍历字符数组，时间复杂度是O(n)。
3、C 字符串长度的变更会对字符数组做内存重分配。
4、通过从字符串开始到结尾碰到的第一个'\0'来标记字符串的结束，因此不能保
存图片、音频、视频、压缩文件等二进制(bytes)保存的内容，二进制不安全。
SDS 的特点：
1、不用担心内存溢出问题，如果需要会对SDS 进行扩容。
2、获取字符串长度时间复杂度为O(1)，因为定义了len 属性。
3、通过“空间预分配”（ sdsMakeRoomFor）和“惰性空间释放”，防止多
次重分配内存。
4、判断是否结束的标志是len 属性（它同样以'\0'结尾是因为这样就可以使用C

#####  3、embstr 和raw 的区别？

​         embstr 的使用只分配一次内存空间（因为RedisObject 和SDS 是连续的），而raw
需要分配两次内存空间（分别为RedisObject 和SDS 分配空间）。
​           因此与raw 相比，embstr 的好处在于创建时少分配一次空间，删除时少释放一次
空间，以及对象的所有数据连在一起，寻找方便。
​           而embstr 的坏处也很明显，如果字符串的长度增加需要重新分配内存时，整个
RedisObject 和SDS 都需要重新分配空间，因此Redis 中的embstr 实现为只读。

##### 4.String 类型的使用

###### 4.1 使用setNx可以作为分布式锁实现

```c++
// 初次设置一个key
127.0.0.1:6379> setNx gege 520
(integer) 1
127.0.0.1:6379> get gege
"520"
 //再次设置相同的key值的时候，设置错误
127.0.0.1:6379> setNx gege 520
(integer) 0
127.0.0.1:6379> 
```

##### 4.2、可以作为热点数据进行存储

> 对象缓存，列表缓存，热点数据

##### 4.3、可以进行分布式session

​				因为redis是一个共享的内存，可以将一些通用的信息放入到redis里面，比如 用户的登录信息

```java
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

### 2、hash类型



Redis 的Hash 本身也是一个KV 的结构，类似于Java 中的HashMap。外层的哈希（Redis KV 的实现）只用到了hashtable。当存储hash 数据类型时，我们把它叫做内层的哈希。内层的哈希底层可以使用两种数据结构实现：

**ziplist：OBJ_ENCODING_ZIPLIST（压缩列表）**
**hashtable：OBJ_ENCODING_HT（哈希表）**

```c
127.0.0.1:6379> hset h2 f aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
(integer) 1
127.0.0.1:6379> hset h3 f aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
(integer) 1
127.0.0.1:6379> object encoding h2
"ziplist"
127.0.0.1:6379> object encoding h3
"hashtable"
```

> ziplist 是一个经过特殊编码的双向链表，它不存储指向上一个链表节点和指向下一个链表节点的指针，而是     **存储上一个节点长度和当前节点长度**，通过牺牲部分读写性能，来换取高效的内存空间利用率，是一种时间换空间的思想。只用在字段个数少，字段值小的场景里面。

​			内部的结构分析：

```c
typedef struct zlentry {
unsigned int prevrawlensize; /* 上一个链表节点占用的长度*/
unsigned int prevrawlen; /* 存储上一个链表节点的长度数值所需要的字节数*/
unsigned int lensize; /* 存储当前链表节点长度数值所需要的字节数*/
unsigned int len; /* 当前链表节点占用的长度*/
unsigned int headersize; /* 当前链表节点的头部大小（prevrawlensize + lensize），即非数据域的大小*/
unsigned char encoding; /* 编码方式*/
unsigned char *p; /* 压缩链表以字符串的形式保存，该指针指向当前节点起始位置*/
} zlentry;
```

![image-20200603172449187](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603172449.png)

![image-20200603172542676](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603172542.png)

什么时候使用ziplist存储？

当hash 对象同时满足以下两个条件的时候，使用ziplist 编码：
1）所有的键值对的健和值的字符串长度都小于等于64byte（一个英文字母一个字节）；
2）哈希对象保存的键值对数量小于512 个。

src/redis.conf

```c
hash-max-ziplist-value 64 // ziplist 中最大能存放的值长度
hash-max-ziplist-entries 512 // ziplist 中最多能存放的entry 节点数量
```

#### hashtable(dict)

hashtable被称为字典，它是一个数组+链表的结构。

源码位置：dict.h

```c++
typedef struct dict {
dictType *type; /* 字典类型*/
void *privdata; /* 私有数据*/
dictht ht[2]; /* 一个字典有两个哈希表*/
long rehashidx; /* rehash 索引*/
unsigned long iterators; /* 当前正在使用的迭代器数量*/
} dict;
```

### 3.list类型

存储有序的字符串（从左到右），元素可以重复。可以充当队列和栈的角色。

![image-20200603204809109](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603204809.png)

![image-20200603204847237](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603204847.png)

在早期的版本是用ziplist和lindlist进行储存数据的。

后期的话是用quicklist(快速列表)

```c
typedef struct quicklist {
quicklistNode *head; /* 指向双向列表的表头*/
quicklistNode *tail; /* 指向双向列表的表尾*/
unsigned long count; /* 所有的ziplist 中一共存了多少个元素*/
unsigned long len; /* 双向链表的长度，node 的数量*/
int fill : 16; /* fill factor for individual nodes */
unsigned int compress : 16; /* 压缩深度，0：不压缩； */
} quicklist;
```

quicklistNode 中的*zl 指向一个ziplist，一个ziplist 可以存放多个元素。

```c
typedef struct quicklistNode {
struct quicklistNode *prev; /* 前一个节点*/
struct quicklistNode *next; /* 后一个节点*/
unsigned char *zl; /* 指向实际的ziplist */
unsigned int sz; /* 当前ziplist 占用多少字节*/
unsigned int count : 16; /* 当前ziplist 中存储了多少个元素，占16bit（下同），最大65536 个*/
unsigned int encoding : 2; /* 是否采用了LZF 压缩算法压缩节点，1：RAW 2：LZF */
unsigned int container : 2; /* 2：ziplist，未来可能支持其他结构存储*/
unsigned int recompress : 1; /* 当前ziplist 是不是已经被解压出来作临时使用*/
unsigned int attempted_compress : 1; /* 测试用*/
unsigned int extra : 10; /* 预留给未来使用*/
} quicklistNode;
```

![image-20200603205234036](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603205234.png)



### 4. set类型

储存实现原理

Redis 用intset 或hashtable 存储set。如果元素都是整数类型，就用inset 存储。

如果不是整数类型，就用hashtable（数组+链表的存来储结构）。

### 5、有序结合ZSet

**sorted set，有序的set，每个元素有个score。**
**score 相同时，按照key 的ASCII 码排序。**

![image-20200603210346524](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603210346.png)

同时满足以下条件时使用ziplist 编码：

- ​		元素数量小于128个
- ​      长度小于64个字节

超过阈值之后，使用skiplist+dict 存储。

![image-20200603211314865](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603211314.png)

```c++
typedef struct zskiplistNode {
sds ele; /* zset 的元素*/
double score; /* 分值*/
struct zskiplistNode *backward; /* 后退指针*/
struct zskiplistLevel {
struct zskiplistNode *forward; /* 前进指针，对应level 的下一个节点*/
unsigned long span; /* 从当前节点到下一个节点的跨度（跨越的节点数） */
} level[]; /* 层*/
} zskiplistNode
```

![image-20200603212124023](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603212124.png)

## 三、redis事务

### 1.为什么要用事务？

> redis的单个操作是事务的，如果是多个命令的时候的时候，需要把多个命令作为一个不可分割的处理序列。

​	redis事务的特点：

- 按照进入队列的顺序

- 不会受其他客户端的请求的影响

  Redis 的事务涉及到四个命令：multi（开启事务），exec（执行事务），discard（取消事务），watch（监视）

  > ```c
  > 127.0.0.1:6379> set gege 1000
  > OK
  > 127.0.0.1:6379> multi
  > OK
  > 127.0.0.1:6379> decrby tom 200
  > QUEUED
  > 127.0.0.1:6379> incrby gege 200
  > QUEUED
  > 127.0.0.1:6379> exec
  > 1) (integer) 800
  > 2) (integer) 1200
  > 127.0.0.1:6379> 
  > ```

  watch命令

  ​       在Redis 中还提供了一个watch 命令。
  ​       它可以为Redis 事务提供CAS 乐观锁行为（Check and Set / Compare andSwap），也就是多个线程更新变量的时候，会跟原值做比较，只有它没有被其他线程修改的情况下，才更新成新的值。
  ​      我们可以用watch 监视一个或者多个key，如果开启事务之后，至少有一个被监视
  key 键在exec 执行之前被修改了， 那么整个事务都会被取消（key 提前过期除外）。可
  以用unwatch 取消。
  
  ​	![image-20200601104901439](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200601105000.png)

### 2.执行事务可能遇到的问题

#### 2.1 执行exec命令之前发生的错误

​				比如：入队的命令存在语法错误，包括参数数量，参数名等等（编译器错误）。

​				在这种情况下，事务会拒绝执行，也就是所有的命令都不会得到执行。

```c++
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set tom 88888
QUEUED
127.0.0.1:6379> 
127.0.0.1:6379> set pb 888888
QUEUED
127.0.0.1:6379> discard
OK
127.0.0.1:6379> get tom
"800"
```



#### 	2.2、执行exec命令之后发生的错误

```c
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 1
QUEUED
127.0.0.1:6379> hset k1 a b
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"1"
127.0.0.1:6379> 
```

​       最后我们发现set k1 1 的命令是成功的，也就是在这种发生了运行时异常的情况下，只有错误的命令没有被执行，但是其他命令没有受到影响。
​		这个显然不符合我们对原子性的定义，也就是我们没办法用Redis 的这种事务机制
来实现原子性，保证数据的一致。

### 3.Lua脚本

​			Lua/ˈluə/是一种轻量级脚本语言，它是用C 语言编写的，跟数据的存储过程有点类
似。使用Lua 脚本来执行Redis 命令的好处：
​	1、一次发送多个命令，减少网络开销。
​	2、Redis 会将整个脚本作为一个整体执行，不会被其他请求打断，保持原子性。
​	3、对于复杂的组合命令，我们可以放在文件中，可以实现程序之间的命令集复
用。

```c
redis> eval lua-script key-num [key1 key2 key3 ....] [value1 value2 value3 ....]
 eval 代表执行Lua 语言的命令。
 lua-script 代表Lua 语言脚本内容。
 key-num 表示参数中有多少个key，需要注意的是Redis 中key 是从1 开始的，如果没有key 的参数，那么写0。
 [key1 key2 key3…]是key 作为参数传递给Lua 语言，也可以不填，但是需要和key-num 的个数对应起来。
 [value1 value2 value3 ….]这些参数传递给Lua 语言，它们是可填可不填的。
```

​	**		**需求：在X 秒内只能访问Y 次。**
**设计思路：用key 记录IP，用value 记录访问次数。**
**拿到IP 以后，对IP+1。如果是第一次访问，对key 设置过期时间（参数1）。否则判断次数，超过限定的次数（参数2），返回0。如果没有超过次数则返回1。超过时间，**
 **key 过期之后，可以再次访问。**
 KEY[1]是IP， ARGV[1]是过期时间X，ARGV[2]是限制访问的次数Y。**

```c++
-- ip_limit.lua
-- IP 限流，对某个IP 频率进行限制，6 秒钟访问10 次
local num=redis.call('incr',KEYS[1])
if tonumber(num)==1 then
redis.call('expire',KEYS[1],ARGV[1])
return 1
elseif tonumber(num)>tonumber(ARGV[2]) then
return 0
else
return 1
end
```



```c++
//执行参数
./redis-cli --eval "ip_limit.lua" app:ip:limit:192.168.8.111 , 6 10
```

## 四、redis为什么这么快？

- 纯内存操作
- 单线程
- I/O多路复用

​                   **是通**过一种机制一个进程能同时等待多个文件描述符，**
​                   **而这些文件描述符（套接字描述符）其中的任意一个进入读就绪（readable）状态，select()**
​                   函数就可以返回。**

## 五、内存回收

### 1.过期策略

1. 定时过期

   设置过期时间，redis里面会生成一个定时器，到期立即清除。

2. 惰性过期

   被动淘汰，当只有访问一个key时，才会判断key是否过期。

3. 定期过期

   每隔一定的时间，会扫描一定数量的数据库的expires 字典中一定数量的key，并清
   除其中已过期的key

### 2、淘汰策略

​		redis的上限：redis.conf   里面，进行设置   maxmemory

​		动态修改 config set maxmemory

1. ​		LRU  Least Recent Used  最近最少使用  设置目前最远的数据被淘汰。  ttl 
2. ​       LFU   Least Frequenty User   使用频率最少使用   不用设置 ttl
3. ​        vloatitle ttl   最近要过期的进行删除
4. ​      Random 随机删除数据

## 六、持久化

### 1.RDB快照  (Redis DataBase)

触发快照的生成：    dump.rdb

rdbcompression yes   对文件进行压缩

second  多少秒之内，修改了多少个key

save 60 10000 60秒之内修改了10000个key

- shutdown触发  flushall 

  保证服务器正常关闭

- 自动触发    只要有一个被触发的话，那就会声成

  ```c
  save 900 1 # 900 秒内至少有一个key 被修改（包括添加）
  save 300 10 # 400 秒内至少有10 个key 被修改
  save 60 10000 # 60 秒内至少有10000 个key 被修改
  ```

  上面的配置是不冲突的，只要满足任意一个就会触发

  下面是主动触发

- save 命令    （同步操作，影响业务线程操作）       bgsave(异步持久化)    lastsave（最后一次执行的时间）

优势：

1.RDB 是一个非常紧凑(compact)的文件，它保存了redis 在某个时间点上的数据集。这种文件非常适合用于进行备份和灾难恢复。
2.生成RDB 文件的时候，redis 主进程会fork()一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO 操作。
3.RDB 在恢复大数据集时的速度比AOF 的恢复速度要快。

缺点：

在一定间隔时间做一次备份，所以如果redis 意外down 掉的话，就会丢失最后一次快照之后的所有修改（数据有丢失）。

### 2.AOF

> 默认不开启的，需要在conf里面开启  append   Append Only File

![image-20200603213424864](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200603213424.png)

当文件越来越多的时候，怎么处理？

> 例如set gupao 666，执行1000 次，结果都是gupao=666。为了解决这个问题，Redis 新增了重写机制，当AOF 文件的大小超过所设定的阈值时，Redis 就会启动AOF 文件的内容压缩，只保留可以恢复数据的最小指令集。可以使用命令**bgrewriteaof** 来重写。**AOF 文件重写并不是对原文件进行重新整理，而是直接读取服务器现有的键值对，然后用一条命令去代替之前记录这个键值对的多条命令**，生成一个新的文件后去替换原来的AOF 文件。

自动重写：# 重写触发机制

```c
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

> 同时开启的话，aof的作用域大于rdb

   对比

> ​      **那么对于AOF 和RDB 两种持久化方式，我们应该如何选择呢？**
> ​    **如果可以忍受一小段时间内数据的丢失，毫无疑问使用RDB 是最好的，定时生成RDB 快照（snapshot）非常便于进行数据库备份， 并且RDB 恢复数据集的速度也要比AOF 恢复的速度要快。**
> ​     **否则就使用AOF 重写。但是一般情况下建议不要单独使用某一种持久化机制，而**
> **是应该两种一起用，在这种情况下,当redis 重启的时候会优先载入AOF 文件来恢复原始**
> **的数据，因为在通常情况下AOF 文件保存的数据集要比RDB 文件保存的数据集要完整。**

## 七、redis分布式操作

###  1、为什么需要集群了

- ### 性能

- ###  扩展

- ###   可用

### 2、主从复制原理：

- 建立连接时
  - 执行slave命令，会在自己本地保存 master node的信息，包括host ip
  - slave node内部有个定时任务 replication，每隔一秒检查是否有新的master node要连接和复制，如果发现建立一个socket节点，，，，，，接受rdb文件，接受命令传播
- 数据同步阶段
  - 第一次全量复制，通过bgsave命令在本地生成一份rdb文件，传给master。
  - 生成rdb文件时，如果发现有新的命令，将命令进行缓存。

- 命令传播

  - repl-disable-tcp-nodelay no设置为yes，默认将时间是40ms,如果设置为no,不会有延迟，立即将数据发送过去。

  > ```
  > 如果从节点有一段时间断开了与主节点的连接是不是要重新全量复制一遍？
  > 如果可以增量复制，怎么知道上次复制到哪里？
  > ```

  

  ```c++
  redis> info replication
  ```

  

slave  xx   xxx  从节点执行这命令，认xxx   为主节点，自己本身为从节点   ，全量复制

建立一个socket  ---master

master   bgsave   ----rdb     -----------slave

从节点接收到数据 进行解析

命令传播，保证实时性。   如果设置为yes，是打包发送



子节点和父节点断开联系，何如保证数据的一致性？

增量复制。





手动切换主从？

自动切换子节点为父节点？

### 3、Setinal   哨兵模式 

Sentinel 既监控所有的Redis 服务，Sentinel 之间也相互监控。如果主节点挂掉之后，那么就sentinel就会选择另外一个从节点。

服务下线

- 主观下线
  - 如果一个哨兵发现主节点挂了，那么下线自己所关注的节点。
-  客观下线
  - 当其他的哨兵也发现主节点都挂掉了，那么这个才是真正的下线

故障转移

​		如果发现master挂掉了，那么就会进行故障转移

共识算法

​		在分布式存储系统中，通常通过维护多个副本来提高系统的可用性，那么多个节点之间必须要面对数据一致性的问题。Raft 的目的就是通过复制的方式，使所有节点达成一致，但是这么多节点，以哪个节点的数据为准呢？所以必须选出一个Leader。

​		Raft 是一个共识算法（consensus algorithm）Raft 的核心思想：先到先得，少数服从多数。

​      动态演示：http://thesecretlivesofdata.com/raft/



![image-20200601205925405](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200601205925.png)



子节点不可以进行更改数据，主节点可以更改数据。

主从之间的的数据是怎么复制的呢？

#### 问题：怎么让一个原来的slave 节点成为主节点？

1、选出Sentinel Leader 之后，由Sentinel Leader 向某个节点发送slaveof no one
命令，让它成为独立节点。
2、然后向其他节点发送slaveof x.x.x.x xxxx（本机服务），让它们成为这个节点的
子节点，故障转移完成。

#### 问题：这么多从节点，选谁成为主节点？

​        关于从节点选举，一共有四个因素影响选举的结果，分别是断开连接时长、优先级排序、复制数量、进程id。
​        如果与哨兵连接断开的比较久，超过了某个阈值，就直接失去了选举权。如果拥有
选举权，那就看谁的优先级高，这个在配置文件里可以设置（replica-priority 100），数值越小优先级越高。如果优先级相同，就看谁从master 中复制的数据最多（复制偏移量最大），选最多的那个，如果复制数量也相同，就选择进程id 最小的那个。

Sentinel 功能总结：

监控：Sentinel 会不断检查主服务器和从服务器是否正常运行。
通知：如果某一个被监控的实例出现问题，Sentinel 可以通过API 发出通知。自动故障转移（failover）：如果主服务器发生故障，Sentinel 可以启动故障转移过程。把某台服务器升级为主服务器，并发出通知。
配置管理：客户端连接到Sentinel，获取当前的Redis 主服务器的地址。



![image-20200601210952696](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200601210952.png)

服务端进行分片服务？

1. 分部均匀？

![image-20200601213015354](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200601213015.png)

​     一致性hash。均匀分部

1. 客户端访问?
2. reshard?