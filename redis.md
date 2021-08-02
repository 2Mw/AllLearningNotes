# Redis

Redis(==Re==mote ==Di==ctionary ==S==erver)远程服务字典

## 安装Redis

1. 首先有java环境，最好在CentOS系统下

2. 下载Redis文件，以及看看是否有`make`命令，没有则使用`yum install gcc-c++`

3. 解压redis文件后进入输入`make`命令，得到一个`src`目录。进行`make install`

4. 安装会默认安装再`/usr/local/bin`目录下

   ![image-20210407202614681](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210407202614681.png)

5. 将`redis.conf`复制到`/usr/local/bin`目录下，并且修改`deamonize yes`
6. 通过指定配置文件启动redis服务。`redis-server redis.conf`
7. 打开连接`redis-cli -p 6379`
8. 查看redis进程`ps -ef | grep redis`

## 性能测试

> redis-benchmark压力测试服务

```
 -h <hostname>      Server hostname (default 127.0.0.1)
 -p <port>          Server port (default 6379)
 -s <socket>        Server socket (overrides host and port)
 -a <password>      Password for Redis Auth
 --user <username>  Used to send ACL style 'AUTH username pass'. Needs -a.
 -c <clients>       Number of parallel connections (default 50)
 -n <requests>      Total number of requests (default 100000)
 -d <size>          Data size of SET/GET value in bytes (default 3)
 --dbnum <db>       SELECT the specified db number (default 0)
 --threads <num>    Enable multi-thread mode.
 --cluster          Enable cluster mode.
 --enable-tracking  Send CLIENT TRACKING on before starting benchmark.
 -k <boolean>       1=keep alive 0=reconnect (default 1)
 -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD,
                    random members and scores for ZADD.
  Using this option the benchmark will expand the string __rand_int__
  inside an argument with a 12 digits number in the specified range
  from 0 to keyspacelen-1. The substitution changes every time a command
  is executed. Default tests use this to hit random keys in the
  specified range.
 -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
 -e                 If server replies with errors, show them on stdout.
                    (no more than 1 error per second is displayed)
 -q                 Quiet. Just show query/sec values
 --precision        Number of decimal places to display in latency output (default 0)
 --csv              Output in CSV format
 -l                 Loop. Run the tests forever
 -t <tests>         Only run the comma separated list of tests. The test
                    names are the same as the ones produced as output.
 -I                 Idle mode. Just open N idle connections and wait.
 --help             Output this help and exit.
 --version          Output version and exit.
```

比如：

```sh
redis-benchmark -h localhost -p 6379 -c 100 -n 50000
```

## 基础知识

* redis默认有16个数据库，使用`select [index]`选择第index个数据库。
* `DBSIZE` 查看数据库的大小
* `keys *` 查看所有的键值
* `flushdb` 清空当前数据库
* `flushall` 清空所有的数据库
* Redis是单线程的，Redis是基于内存操作的，Redis的瓶颈是根据机器的内存和网络带宽据欸的那个的。
* 为什么redis这么快？redis是将所有的数据全部放在内存中，所以使用单线程操作效率最高，不需要上下文切换。 

## 五大数据类型

Redis可以用作数据库、缓存和消息中间件。

### Redis-key

* `set key value [EX seconds | PX millonseconds]`
* `get key`
* `move key [index]` 将某个键值移动到第n的数据库
* `EXISTS key` 查看某个值是否存在
* `EXPIRE Key seconds` 让key值在n秒内消失。`ttl key`查看剩余的秒数 
* `type key` 查看key值的类型

### String类型

* `APPEND key value` 在已有key值的情况下，连接字符串value；如果key不存在相当于创建字符串
* `STRLEN key` 获得某个key的字符串长度
* `GETRANGE key start end` 相当于substring。但是范围为`[start, end]`
* `SETRANGE key start value` 替换字符串从start位置开始替换目标字符串的长度。
* `setnx key value` 如果key值不存在，则设置为value返回为1；存在则创建失败且返回为0。
* `mset key1 value1 key2 value2 ...` 批量设置key-value
* `mget key1 key2 key3` 批量获取
* `msetnx key1 value1 key2 value2 ...` 批量设置，原子操作，一个失败则全部失败
* key值可以巧妙设计：`set user:{id}:{name} value`
* `getset key value` 先get在set，返回get后的值。类似于CAS(compare And Swap)操作

对于数值类型：

* `INCR key` 相当于key++  `DECR key` 相当于key--
* `INCRBY key steps` 相当于key+=steps  `DECRBY key steps` 相当于key-=steps

### List类型

> 可以将list实现栈或者队列

`LPUSH`和`RPUSH`，放到list的第一个和最后一个

`LPOP`和`RPOP`，移除list的第一个和最后一个元素

`LRANGE list start end` 获取list中的值[start, end]

`LINDEX list [index]` 获取list指定索引下标的元素

`LLEN list` 获取list列表长度

`LREM list count value` 删除list中指定个数的值

`LTRIM list start end` 截取原来数组指定index的元素

`RPOPLPUSH old new` 从原来列表中移除最后一个元素并放入新的列表中

`LSET list index value` 指定对应索引的值，必须对应索引存在

`LINSERT list [before|after] pivot value` 在list中某个pivot前后插入对应的值。

### Set类型

不可重复

`sadd myset value`

`SMEMBERS set` 获取set内的值

`SISMEMBER set value` 查看value是否在set集合内

`SREM set value`	移除

`SCARD SET` 查看set的个数

`SPOP` 随机删除一个元素 `SRANDMEMBER set [count]` 随机选取n个元素

`SMOVE source dest element` 移动元素

**集合操作**

差集：`SDIFF SET1 SET2`

交集：`SINTER SET1 SET2` 找共同好友或者共同关注之类的

并集：`SUNION SET1 SET2`

### Hash 类型

类似于存储js中的对象类型。

在redis类似于`string`类型

```json
{js:{field1:value,field2:value2}}
```

`hset key field value` 设置值

`hget key field`

`hmset` `hmget` 批量设置和获取。`hgetall` 获取所有键值对

`hdel obj field` 删除某个属性

`HLEN obj` 获取hash的长度

`HEXISTS obj field` 某个属性是否存在

`HKEYS`  `HVALS` 获取所有的键或值

`HINCR` `HDECR` `HSETNX`

### Zset

在set基础上进行排序。按照key来进行排序

`zadd myset 100 xiaohong`

### Bitmap位存储类型

统计用活跃不活跃、登陆未登录，只有1和0两种状态。

`setbit sign bit value`

`getbit sign bit`

统计操作：

`bitcount key` 记录key中1的个数

## Redis 的事务集合

Redis的单条命令是原子性的，但是事务不保证原子性，也没有隔离级别的概念。

redis事务的三个阶段：

* 开始事务（multi）
* 命令入队
* 执行事务（exec）或者放弃事务（discard）

```bash
127.0.0.1:6379> MULTI	# 开启事务
OK
127.0.0.1:6379(TX)> set key1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> exec # 执行事务
1) OK
2) OK
```

如果事务执行过程中出现错误：

* 如果出现语法命令型错误会直接放弃事务
* 运行时错误：其他正确的命令不影响

### 锁

**悲观锁：**

**乐观锁：**

