# 数据库高阶技巧

[TOC]

[BV1Xf4y1C7WT](https://www.bilibili.com/video/BV1Xf4y1C7WT)

[BV1X64y1x7HT](https://www.bilibili.com/video/BV1X64y1x7HT)

## MySQL架构

参考：[MySQL 性能优化](http://blog.timwang.top/2019/08/02/mysql/)

![img](https://i.loli.net/2021/08/02/dqQtzPKAifD4bwN.png)

存储引擎：MyISAM, InnoDB，支持的引擎可以使用`show engines`查看。

数据库后缀名为`MYI` -> MyISAM，`ibd` -> InnoDB

| 存储引擎 | 优点                                                         | 缺点                                                         |
| :------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| InnoDB   | 5.5版本后MySQL默认数据库，支持事务，比MyISAM处理速度稍慢     | 非常复杂，性能较一些简单的引擎要差一点儿。空间占用比较多。   |
| MyISAM   | 高速引擎，拥有极高的插入，查询速度                           | 不支持事务，不支持行锁、崩溃后数据不容易修复                 |
| Archive  | 将数据压缩后存储，非常适合存储大量的独立的，作为历史记录的数据 | 只能进行插入和查询操作，非事务型                             |
| CSV      | 是基于CSV格式文件存储数据（应用于跨平台数据交换）            |                                                              |
| Memory   | 内存存储引擎，拥有极高的插入，更新和查询效率                 | 占用和数据量成正比的内存空间，只在内存上保存数据，意味着数据可能会丢失，并发能力低下。不支持BLOB或TEXT类型的列 |
| Falcon   | 一种新的存储引擎，支持事务处理，传言可能是InnoDB的替代者     |                                                              |

## 执行计划

explain关键字。

```sql
explain select * from user;
```

输出：

[EXPLAIN Output Format](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-join-types)

```
id, select_type, `table`, partitions, type,  possible_keys, `key`, key_len, ref, rows, filtered, Extra
1, 'SIMPLE',     'user',  null,       'ALL', null,           null, null,    null, 3,   100,  null
```

`type`：具体的值可以是`system`, `const`, `ref`, `range`, `index`, `all`。从左到右效率逐次降低。`all`代表全表扫描，效率很低，要避免出现`all`。

`key`：选择使用到的索引。

`Extra`：其他的关键信息。

## 索引

<img src="https://i.loli.net/2021/08/02/EfLKN9SI8e4uxl6.png" alt="image-20210802183909781" style="zoom:67%;" />

索引和实际的数据都是存储在磁盘中，在进行数据读取的时候会优先将索引读取到内存当中。

### 数据结构

索引的数据结构：二叉树、红黑树、Hash表、B树、**B+树**。

非B树存储当数据变多的时候树会变得非常高。

hash表无法进行范围查询，否则要进行挨个查询。

为什么使用B+树而不使用B树？

* B+树的磁盘读写代价更小。B+树的内部节点并没有指向关键字具体信息的指针，因此其内部节点相对B树更小，通常B+树矮更胖，高度小查询产生的I/O更少。
* B+树查询效率更高。B+树使用双向链表串连所有叶子节点，区间查询效率更高（因为所有数据都在B+树的叶子节点，扫描数据库 只需扫一遍叶子结点就行了），但是B树则需要通过中序遍历才能完成查询范围的查找。
* B+树查询效率更稳定。B+树每次都必须查询到叶子节点才能找到数据，而B树查询的数据可能不在叶子节点，也可能在，这样就会造成查询的效率的不稳定。

如果B+树3-4层都存储满索引，数据量起码有千万级。

MySQL中的B+树索引：

* 非叶子节点不存储data，只存储索引。
* 叶子节点包含索引字段和data

查看mysql中innodb叶节点数量大小：`show global status like "Innodb_page_size"`

查看对应表中的索引：`show index from <table>`

### 聚簇索引和非聚簇索引

<img src="https://i.loli.net/2021/08/02/nZDXrkofUjGv2I5.png" alt="image-20210802223248752" style="zoom:67%;" />

<img src="https://i.loli.net/2021/08/02/Q2D8VXN3qAgkMwG.png" alt="image-20210802223441721" style="zoom:67%;" />

### 回表、索引覆盖、最左匹配、索引下推

* **回表**：

  id, name, age, gender. id为自增主键索引，name为普通索引。

  `select * from user where name = "john"`。这个表有两个B+树。首先根据name查其B+树得到对应的id值，在根据id值查询id对应的B+树来检索对应的数据记录，这个过程叫做回表。**尽可能避免回表操作**。

* **索引覆盖**：

  `select id, name from user where name = "john"`。此操作根据name的值去查询name对应的B+树，能获取到id的值，不需要获取到所有列的值，因此不需要回表，这个过程叫做索引覆盖。使用`explain`会有`using index`的提示，**推荐使用**。

  在某些场景中，可以考虑将所有的列变成组合索引，就无需回表，加快查询速度。

  `select id, name, age from user where name = "john"` 则需要回表。

* **最左匹配：**

  首先解释一下**组合索引**或者**联合索引**：可以选择多个列来共同组成索引，但是要遵循最左匹配原则。

  id, name, age, gender. id为自增主键索引，name， age为联合索引。

  `select * from user where name = "john" and age = 12`		->   会走组合索引

  `select * from user where name = "john"`								->   会走组合索引

  `select * from user where age = 12`										->   不会走组合索引

  `select * from user where age = 12 and name = "john"`		->   会走组合索引，mysql自动优化顺序

* **索引下推**：

  `select * from user where name = "john" and age = 12`

  无索引下推：先根据name过滤并将数据拉取到server层，如何在server层对age进行过滤。

  有索引下推：直接根据两个条件进行过滤，筛选后再返回server层。

### 如何回答优化问题：

1. 加索引
2. 看执行计划`explain`语句
3. 优化sql语句，减少回表，增加索引覆盖，满足最左匹配等问题
4. 分库分表
5. 表结构设计优化

> "在实习或项目过程中做个一段sql优化，一般优化并不是出了问题才进行优化，在进行数据库建模或者数据库设计的时候要预先考虑到一些问题，比如字段的类型，字段的长度等，要创建合适的索引；在发生SQL问题的时候，还需要对SQL语句进行性能监控，观察他的执行计划，索引的创建和维护，SQL语句的调整、参数的设置多方面考虑"， 结合实际进行表达灵活变化。

## 事务，锁，MVCC

### MVCC

> Multi-Version Concurrency Control(多版本并发控制). 围绕InnoDB展开，不支持MyISAM。为了在并发控制的情况提高读写效率。

**当前读**：读取到的数据总是最新的数据。

什么时候会触发当前读？

```sql
select * from user lock in share mode;			-- select ... lock in share mode
select * from user for update;				    -- select ... for update
update ...
delete ...
insert ...
```

**快照读**：读取到的是历史版本的记录。

什么时候会触发快照读？

```sql
select ...   -- 朴素select语句
```

<h4>mvcc的组成部分</h4>

1. 隐藏字段

### 隔离级别

[彻底搞懂 MySQL 事务的隔离级别 - 阿里云社区](https://developer.aliyun.com/article/743691)

[Innodb中的事务隔离级别和锁的关系 - 美团技术团队](https://tech.meituan.com/2014/08/20/innodb-lock.html)

>MySQL的事务隔离级别一共有四个，分别是读未提交(RU, read uncommitted)、读已提交(RC, read committed)、可重复读(RR, repeatable read)以及可串行化(Serializable)。
>
>MySQL的隔离级别的作用就是让事务之间互相隔离，互不影响，这样可以保证事务的一致性。

隔离级别比较与对性能的影响：可串行化>可重复读>读已提交>读未提交

MySQL默认的事务隔离级别是RR可重复读。
