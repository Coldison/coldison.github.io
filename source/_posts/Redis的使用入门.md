---
title: Redis的使用入门
date: 2022-10-01 01:26:43
tags:
- redis
categories:
- 教程
- Redis使用
banner_img: https://by3302files.storage.live.com/y4m2PWLpdxiPn1gzYfsRcBZhkSQe4F8FnncBlMEU_gsNXYnWtrQY3ANZnVYUphnCg0l-LVmUKup-TkpCv4n5LmDDRGMwQBv_z9ywI_XLme8U7odjA3R-d-kQikdfXpl0yid-7z9AxtOXelvT0qAN39qZMMUWaLYXa4mj0Cd2f2w1l7GfHprfzFX9GFIiUnzjSt7?width=1772&height=591&cropmode=none
---

[Redis入门及python操作视频课程](https://e-learning.51cto.com/course/12370)
[Redis Documents](https://redis.io/docs/about/)

Redis是一个开源的内存数据存储服务，可以用作实时数据存储，缓存与session存取，流数据与消息存取。

## Redis安装、配置与运行

### Redis安装

[Redis官方文档：安装Redis](https://redis.io/docs/getting-started/installation/install-redis-from-source/)
官网提供了在不同系统安装Redis的方式，这里是从源代码安装。

#### 从源代码安装

我参考的是[CentOS7redis安装教程](https://blog.csdn.net/a_rain2333/article/details/119567362)。

处于权限的考虑，最好切换到root用户，如果不是的话应该也没关系，redis-server的进程因此就属于用户。

安装gcc

```bash
yum install -y gcc 
```

下载并解压redis

```bash
wget https://download.redis.io/releases/redis-6.2.5.tar.gz
tar -zvxf redis-6.2.5.tar.gz
```

移动文件夹，不移动到该文件夹也没问题，只要记得自己的文件目录就可以。

```bash
mv redis-6.2.5 /usr/local/redis
```

cd到redis目录，键入``make``命令进行编译。编译完成最好``make test``一下，如果没有报错就没问题。然后

```bash
make PREFIX=/usr/local/redis install
```

#### 基于Docker

参考菜鸟教程[docker安装redis](https://www.runoob.com/docker/docker-install-redis.html)。

```bash
docker pull redis:latest
```

按照默认配置运行，docker的run命令可以参考我的[Docker的使用示例(一)](https://coldison.github.io/2022/09/15/Docker%E7%9A%84%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B(%E4%B8%80)/)。

```bash
docker run -itd --name redis-test -p 6379:6379 redis
```

容器中运行bash

```bash
docker exec -it redis-test /bin/bash
```

容器中运行redis-cli:
![容器中运行redis-cli](https://www.runoob.com/wp-content/uploads/2016/06/docker-redis7.png)

### redis的配置与运行

#### redis的配置

redis文件夹中有redis.conf文件。

我用到的有两个：

1. ``bind 127.0.0.1``：绑定的主机地址。
2. ``port 6379``： 指定Redis监听端口，默认端口为6379。

补充部分可以暂时不看：

1. ``daemonize yes``：Redis默认以守护进程运行，设置no关闭。
2. ``pidfile /var/run/redis.pid``：当Redis以守护进程方式运行时，Redis默认会把pid写入``/var/run/redis.pid``文件，可以通过pidfile指定。
3. ``timeout 300``：当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能。
4. ``loglevel verbose``：指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose。
5. ``logfile stdout``：日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给``/dev/null``。
6. ``database 16``：设置数据库的数量，默认数据库id为0，可以使用``SELECT <dbid>``命令在连接上指定数据库id。
7. ``save <seconds> <changes>``：指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合。默认配置文件：

    ```conf
    save 900 1
    save 300 10
    save 60 10000
    ```

分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
8. ``rdbcompression yes``：指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大。
9. ``dbfilename dump.rdb``：指定本地数据库文件名，默认值为dump.rdb。
10. ``dir ./``：指定本地数据库文件名，默认值为dump.rdb。
11. ``slaveof <masterip> <masterport>``：设置当本机为slave服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步。
12. ``masterauth <master-password>``： 当master服务设置了密码保护时，slave服务连接master的密码。
13. ``requirepass foobared``：设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过``AUTH <password>``命令提供密码，默认关闭。
14. ``maxclients 128``：设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回``max number of clients reached``错误信息。

#### redis的运行

运行服务端：

```bash
./bin/redis-server redis.conf
```

如果不指定conf文件，则redis会按照默认的conf运行，当然也有可能找不到defaultconf。

运行自带的redis客户端``./redis-cli``，可以加上``-p``指定端口，因为默认是6379而我这里被默认的Ray的redis给占用了，所以指定端口为6879。然后就可以输入Redis命令，接下来就是操作Redis基本数据类型的命令。

```bash
(base) [coldison@localhost bin]$ ./redis-cli
127.0.0.1:6379> set funnything standupcomedy
Error: Protocol error, got "\x00" as reply type byte
(base) [coldison@localhost bin]$ ./redis-cli -p 6879
127.0.0.1:6879> set funnything standupcomedy
OK
127.0.0.1:6879> 
```

## Redis的数据类型

[redis官方文档: 数据类型](https://redis.io/docs/data-types/)
redis的数据类型有：[string](#string-字符串类型)，[list](#list-列表类型)，[set](#set-集合数据类型)，[hash](#hash-数据类型)，类似于python的dict。还有[有序集合sorted sets](https://redis.io/docs/data-types/sorted-sets/)，[stream](https://redis.io/docs/data-types/streams)(主要用于日志)，[Geospatial indexes](https://redis.io/docs/data-types/geospatial/)，[Bitmaps](https://redis.io/docs/data-types/bitmaps/)，[Bitfields](https://redis.io/docs/data-types/bitfields/)，[HyperLogLog](https://redis.io/docs/data-types/hyperloglogs)(为大规模集合提供元素的概率预测)。数据结构可以基于Redis Stack自行扩展。

### string 字符串类型

+ stringg是redis最基本的类型，而且string类型是二进制安全的。
+ redis的string可以包含任何数据。包括jpg图片或者序列化的对象。
+ 最大上限是1G字节。
+ 如果只用string类型，redis就可以被看作加上持久化特性的memcached

例如在集群的进程管理中，ssession通常是以string的形式存取的。

1. ``set key value``：设置key对应的值为string类型的value,返回1表示成功，0失败。
2. ``setnx key value``：同上，如果key已经存在，返回0，表明设置失败。nx是not exist的意思。
3. ``get key``：获取key对应的string值,如果key不存在返回nil。
4. ``getset key value``：设置key的值，并返回key的旧值。如果key不存在返回nil，同时设置成功。
5. ``mget key1 key2 ... keyN``： 一次获取多个key的值，如果对应key不存在，则对应返 回nil。下面是个实验, nonexisting不存在，对应返回nil。
6. ``mset key1 value1 ... keyN valueN``： 一次设置多个key的值，成功返回1表示所有的值都设置了，失败返回0表示没有任何值被设置。
7. ``msetnx key1 value1 ... keyN valueN``： 同上，但是不会覆盖已经存在的key，返回0代表设置失败。

    ```bash
    127.0.0.1:6879> mset f1 t1 f2 t2 f3 sorry
    OK
    127.0.0.1:6879> mget f2 f3
    1) "t2"
    2) "sorry"
    127.0.0.1:6879> mget f2 f3 f4
    1) "t2"
    2) "sorry"
    3) (nil)
    127.0.0.1:6879> mset f1 t1 f2 t2 f3 toobad
    OK
    127.0.0.1:6879> msetnx f1 t1 f2 t2 f3 toobad
    (integer) 0
    ```

8. ``incr key``： 对key的值做加加操作,并返回新的值。注意incr一个不是int的value会返回错误，incr一个不存在的key，则设置key为1。
9. ``decr key``：同上，但是做的是减减操作，decr一个不存在key，则设置key为-1。
10. ``incrby key integer``：同incr，加指定值，key不存在时候会设置key，并认为原来的value是0。
11. ``decrby key integer``：同decr，减指定值。decrby完全是为了可读性，我们
完全可以通过incrby一个负值来实现同样效果，反之一样。

    ```bash
    127.0.0.1:6879> mset n1 1 n2 2
    OK
    127.0.0.1:6879> incr n1 n2
    (error) ERR wrong number of arguments for 'incr' command
    127.0.0.1:6879> incr n1
    (integer) 2
    127.0.0.1:6879> incr n3
    (integer) 1
    127.0.0.1:6879> incr f1
    (error) ERR value is not an integer or out of range
    127.0.0.1:6879> decr n1
    (integer) 1
    127.0.0.1:6879> incrby n3 100
    (integer) 101
    127.0.0.1:6879> decrby n3 23
    (integer) 78
    127.0.0.1:6879> incrby n5 9
    (integer) 9
    127.0.0.1:6879> decrby n6 12
    (integer) -12
    ```

12. ``append key value``：给指定key的字符串值追加value,返回新字符串值的长度。数字也可以被当作字符串看待。
13. ``substr key start end``：返回截取过的key的字符串值,注意并不修改key的值。下标是从0开始的。

    ```bash
    127.0.0.1:6879> get funnything
    "standupcomedy"
    127.0.0.1:6879> append funnything abroad
    (integer) 19
    127.0.0.1:6879> get funnything
    "standupcomedyabroad"
    127.0.0.1:6879> substr funnything 3 12
    "ndupcomedy"
    127.0.0.1:6879> get n3
    "78"
    127.0.0.1:6879> append n3 warriors
    (integer) 10
    127.0.0.1:6879> get n3
    "78warriors"
    127.0.0.1:6879> incr n3 
    (error) ERR value is not an integer or out of range
    ```

以上是string类型。

### list 列表类型

redis的list类型其实就是一个每个子元素都是string类型的双向链表。我们可以
通过push,pop操作从链表的头部或者尾部添加删除元素。这使得list既可以用
作栈，也可以用作队列。
list的pop操作还有阻塞版本的。当我们[lr]pop一个list对象是，如果list是空，
或者不存在，会立即返回nil。但是阻塞版本的b[lr]pop可以则可以阻塞，当然
可以加超时时间，超时后也会返回nil。为什么要阻塞版本的pop呢，主要是为
了避免轮询。举个简单的例子如果我们用list来实现一个工作队列。执行任务
的thread可以调用阻塞版本的pop去获取任务这样就可以避免轮询去检查是
否有任务存在。当任务来时候工作线程可以立即返回，也可以避免轮询带来的
延迟。

1. ``lpush key string``：在key对应list的头部添加字符串元素，**不存在key则创建**，空格为元素定界符，返回1表示成功，0表示key存在且不是list类型。
2. ``rpush key string``：同上，在尾部添加。
3. ``llen key``：返回key对应list的长度，key不存在返回0,如果key对应类型不是list返回错误。
4. ``lrange key start end``：返回指定区间内的元素，下标从0开始，负值表示从后 面计算，-1表示倒数第一个元素 ，key不存在返回空列表。
5. ``ltrim key start end``：截取list，保留指定区间内元素，成功返回1，key不存在返回错误。
6. ``lset key index value``：设置list中指定下标的元素值，成功返回1，key或者下标不存在返回错误。
7. ``lrem key count value``：从key对应list中删除count个和value相同的元素。 count为0时候删除全部。
8. ``lpop key``：从list的头部删除元素，并返回删除元素。如果key对应list不存在或者是空返回nil，如果key对应值不是list返回错误。
9. ``rpop key``：同上，但是从尾部删除。

    ```bash
    127.0.0.1:6879> lpush list1 a b c 
    (integer) 3
    127.0.0.1:6879> rpush list2 x y z
    (integer) 3
    127.0.0.1:6879> llen list1
    (integer) 3
    127.0.0.1:6879> lrange list2 0 -1
    1) "x"
    2) "y"
    3) "z"
    127.0.0.1:6879> rrange list2 0 -1
    (error) ERR unknown command `rrange`, with args beginning with: `list2`, `0`, `-1`, 
    127.0.0.1:6879> lpush list3 alpha beta gamma theta
    (integer) 4
    127.0.0.1:6879> ltrim list3 2 4
    OK
    127.0.0.1:6879> lrange list3 0 -1
    1) "beta"
    2) "alpha"
    ```

10. ``blpop key1...keyN timeout``：从左到右扫描返回对第一个非空list进行lpop操作并返回，比如``blpop list1 list2 list3 0``,如果``list1``不存在，``list2,list3``都是非空则对list2做lpop并返回从list2中删除的元素。如果所有的list都是空或不存在，则会阻塞timeout秒，timeout为0表示一直阻塞。当阻塞时，如果有client对key1...keyN中的任意key进行push操作，则第一在这个key上被阻塞的client会立即返回。如果超时发生，则返回nil。*返回两个值，一个是key，一个是value。*
11. ``brpop``：同blpop，一个是从头部删除一个是从尾部删除。
12. ``rpoplpush srckey destkey``：从srckey对应list的尾部移除元素并添加到 destkey对应list的头部,最后返回被移除的元素值，整个操作是原子的。如果srckey是空或者不存在返回nil。

    ```bash
    127.0.0.1:6879> blpop list1 list2 list3 0
    1) "list1"
    2) "c"
    127.0.0.1:6879> lrange list1 0 -1
    1) "b"
    2) "a"
    127.0.0.1:6879> lrange list2 0 -1
    1) "x"
    2) "y"
    3) "z"
    127.0.0.1:6879> blpop li1 li2 li3 li4 0
    1) "li2"
    2) "chen"
    (331.66s)
    127.0.0.1:6879> rpoplpush list2 list3
    "z"
    ```

以上是列表类型。

### set 集合数据类型

+ redis的set是string类型的无序集合。
+ set元素最大可以包含(2的32次方-1)个元素。
+ set的是通过hash table实现的，hash table会随着添加或者删除自动的调整
大小。
+ 关于set集合类型除了基本的添加删除操作，其他有用的操作还包含集合的取
并集(union)，交集(intersection)，差集(difference)。通过这些操作可以很
容易的实现sns中的好友推荐和blog的tag功能。

1. ``sadd key member``：添加一个string元素到,key对应的set集合中，成功返回1,如果元素以及在集合中返回0，key对应的set不存在返回错误。
2. ``srem key member``：从key对应set中移除给定元素，成功返回1，如果member在
集合中不存在或者key不存在返回0，如果key对应的不是set类型的值返回错误。如果既有存在的也有不存在的元素，则返回删去元素的数量。
3. ``spop key``：删除并返回key对应set中随机的一个元素,如果set是空或者key不存在返回nil。
4. ``srandmember key``：同spop，随机取set中的一个元素，但是不删除元素。

    ```bash
    127.0.0.1:6879> sadd set1 butter
    (integer) 1
    127.0.0.1:6879> sadd list1 beer
    (error) WRONGTYPE Operation against a key holding the wrong kind of value
    127.0.0.1:6879> sadd set1 beer
    (integer) 1
    127.0.0.1:6879> sadd set1 cup
    (integer) 1
    127.0.0.1:6879> sadd set1 cake muffin doughnut
    (integer) 3
    127.0.0.1:6879> srem set muffin cake
    (integer) 0
    127.0.0.1:6879> srem set1 muffin cake cap
    (integer) 2
    127.0.0.1:6879> smembers set1
    1) "doughnut"
    2) "cup"
    3) "beer"
    4) "butter"
    127.0.0.1:6879> spop set1
    "doughnut"
    127.0.0.1:6879> srandmember set1
    "butter"
    ```

5. ``smove srckey dstkey member``：从srckey对应set中移除member并添加到
dstkey对应set中，整个操作是原子的，意思是只能操作单个元素。成功返回1,如果member在srckey中不存在返回0，如果key不是set类型返回错误。

    ```bash
    127.0.0.1:6879> sadd set2 muffin doughnut sandwich croissant hotdog cheesecake
    (integer) 6
    127.0.0.1:6879> smembers set1
    1) "cup"
    2) "beer"
    3) "butter"
    127.0.0.1:6879> smembers set2
    1) "croissant"
    2) "sandwich"
    3) "muffin"
    4) "doughnut"
    5) "cheesecake"
    6) "hotdog"
    127.0.0.1:6879> smove set2 set1 muffin
    (integer) 1
    127.0.0.1:6879> smove set2 set1 muffin cake
    (error) ERR wrong number of arguments for 'smove' command
    127.0.0.1:6879> smove set2 set1 muffin 
    (integer) 0
    ```

6. ``scard key``：返回set的元素个数，如果set是空或者key不存在返回0。
7. ``sismember key member``：判断member是否在set中，存在返回1，0表示不存在
或者key不存在。

    ```bash
    127.0.0.1:6879> scard set2
    (integer) 5
    127.0.0.1:6879> sismember set2 taco
    (integer) 0
    127.0.0.1:6879> sismember set2 cheesecake
    (integer) 1
    ```

8. ``sinter key1 key2...keyN``：返回所有给定key的交集。
9. ``sinterstore dstkey key1...keyN``：同sinter，但是会同时将交集存到dstkey下，覆盖原来的集合。
10. ``sunion key1 key2...keyN``：返回所有给定key的并集。
11. ``sunionstore dstkey key1...keyN``： 同sunion，并同时保存并集到dstkey下，覆盖原来的集合。
12. ``sdiff key1 key2...keyN``：返回所有给定key的差集。
13. ``sdiffstore dstkey key1...keyN``：同sdiff，并同时保存差集到dstkey下。
14. ``smembers key``：返回key对应set的所有元素，结果是无序的。

    ```bash
    127.0.0.1:6879> smembers set1
    1) "burrito"
    2) "cup"
    3) "taco"
    4) "beer"
    5) "butter"
    6) "hotdog"
    7) "sandwich"
    8) "muffin"
    127.0.0.1:6879> smembers set2
    1) "doughnut"
    2) "croissant"
    3) "cheesecake"
    4) "sandwich"
    5) "hotdog"
    127.0.0.1:6879> smembers set3
    1) "avacado"
    2) "pickle"
    127.0.0.1:6879> sinter set1 set2 set3
    (empty array)
    127.0.0.1:6879> sinter set1 set2 
    1) "sandwich"
    2) "hotdog"
    127.0.0.1:6879> sinterstore set4 set1 set2 
    (integer) 2
    127.0.0.1:6879> smembers set4
    1) "hotdog"
    2) "sandwich"
    127.0.0.1:6879> sunion set1 set2 set3
    1) "burrito"
    2) "doughnut"
    3) "beer"
    4) "butter"
    5) "cheesecake"
    6) "sandwich"
    7) "pickle"
    8) "avacado"
    9) "cup"
    10) "taco"
    11) "croissant"
    12) "hotdog"
    13) "muffin"
    127.0.0.1:6879> sunionstore set4 set1 set2 set3
    (integer) 13
    127.0.0.1:6879> smembers set4
    1) "burrito"
    2) "doughnut"
    3) "beer"
    4) "butter"
    5) "cheesecake"
    6) "sandwich"
    7) "pickle"
    8) "avacado"
    9) "cup"
    10) "taco"
    11) "croissant"
    12) "hotdog"
    13) "muffin"
    127.0.0.1:6879> sadd set5 skittle kitkat
    (integer) 2
    127.0.0.1:6879> sunionstore set5 set1 set2 set3
    (integer) 13
    127.0.0.1:6879> smembers set5
    1) "burrito"
    2) "doughnut"
    3) "beer"
    4) "butter"
    5) "cheesecake"
    6) "sandwich"
    7) "pickle"
    8) "avacado"
    9) "cup"
    10) "taco"
    11) "croissant"
    12) "hotdog"
    13) "muffin"
    127.0.0.1:6879> sdiff set2 set1
    1) "doughnut"
    2) "cheesecake"
    3) "croissant"
    127.0.0.1:6879> sdiff set5 set2 set1
    1) "avacado"
    2) "pickle"
    127.0.0.1:6879> sdiffstore set6 set5 set2 set1
    (integer) 2
    ```

以上是集合类型。

### hash 数据类型

redis hash是一个string类型的field和value的映射表。hash特别适合用于存储对象。相较于将对象的每个字段存成单个string类型。将一个对象存储在hash类型中会占用更少的内存，并且可以更方便的存取整个对象。

1. ``hset key field value``：设置hash field为指定值，如果key不存在，则先创建。
2. ``hget key field``：获取指定的hash field。
3. ``hmget key field1....fieldN``：获取全部指定的hash filed。
4. ``hmset key field1 value1 ... fieldN valueN``：同时设置hash的多个field，成功则返回OK；``hset``也可以同时设置多个field，返回field数量。
5. ``hincrby key field integer``：将指定的``hash field``加上给定值，前提是field对应的value也同样是integer。
6. ``hexists key field``：测试指定field是否存在，存在则返回1。
7. ``hdel key field``：删除指定的hash field。
8. ``hlen key``：返回指定hash的field数量。
9. ``hkeys key``：返回hash的所有field。
10. ``hvals key``：返回hash的所有value
11. ``hgetall``：返回hash的所有field和value

    ```bash
    127.0.0.1:6879> hset k1 f1 v1 f2 v2
    (integer) 2
    127.0.0.1:6879> hgetall k1
    1) "f1"
    2) "v1"
    3) "f2"
    4) "v2"
    127.0.0.1:6879> hset k2 f2 v2 f3 v3
    (integer) 2
    127.0.0.1:6879> hget k2 f2
    "v2"
    127.0.0.1:6879> hmget k2 f2 f3
    1) "v2"
    2) "v3"
    127.0.0.1:6879> hmset k3 f3 v3 f4 v4 f5 v5
    OK
    127.0.0.1:6879> hincrby k2 f2 3
    (error) ERR hash value is not an integer
    127.0.0.1:6879> hset k4 f4 3
    (integer) 1
    127.0.0.1:6879> hincrby k4 f4 5
    (integer) 8
    127.0.0.1:6879> hgetall k4
    1) "f4"
    2) "8"

    127.0.0.1:6879> hexists k1 f1
    (integer) 1
    127.0.0.1:6879> hexists k1 f3
    (integer) 0

    127.0.0.1:6879> hexists k1 f1
    (integer) 1
    127.0.0.1:6879> hexists k1 f3
    (integer) 0
    127.0.0.1:6879> hdel k3 f3 f4
    (integer) 2
    127.0.0.1:6879> hgetall k3
    1) "f5"
    2) "v5"
    127.0.0.1:6879> hlen k2
    (integer) 2
    127.0.0.1:6879> hkeys k3
    1) "f5"
    127.0.0.1:6879> hvals k4
    1) "8"
    ```

以上是hash类型，还有一些其他数据类型可以参考官方文档。

## python操作redis

### python安装redis模块

python 操作redis的相关模块是 redis。
安装redis模块：

```bash
pip install redis 
```

### python使用redis

先与redis-server建立连接，由于我们之前设定``host=127.0.0.1``，端口6879，这里设置一下就可以。同样如果是远程存取也是没问题的。然后实例化``redis.Redis()``。这里只给出了简单存取string，更多的方法可以参见``Redis()``源码。

```python
import redis 
pool = redis.ConnectionPool(host='localhost', port=6879, decode_responses=True) 
r = redis.Redis(connection_pool=pool) 
r.set('food','apple') 
print(r.get('food')) 
r.hset('foods','apple','yuande') 
print(r.hget(‘foods','apple'))
```
