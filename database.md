# 数据库笔记

[TOC]

[BV1X64y1x7HT - over](https://www.bilibili.com/video/BV1X64y1x7HT)

锁+主从复制等

[BV1J5411A7ei](https://www.bilibili.com/video/BV1J5411A7ei)

[BV1eD4y1D7pR](https://www.bilibili.com/video/BV1eD4y1D7pR)

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
| Memory   | 内存存储引擎，拥有极高的插入，更新和查询效率，适用于临时表   | 占用和数据量成正比的内存空间，只在内存上保存数据，意味着数据可能会丢失，并发能力低下。不支持BLOB或TEXT类型的列 |
| Falcon   | 一种新的存储引擎，支持事务处理，传言可能是InnoDB的替代者     |                                                              |

## SQL基础

### 普通查询语句

🔵普通查询：

```sql
select A,B,C,D from t1,t2,t3 where p;
```

🔵去重查询：

```sql
select distinct A,B,C,D from t1,t2,t3 where p;
```

🔵字符串运算：

`%` 匹配任意字符（`NULL`除外）， `_` 匹配一个字符

排序：

```sql
select A,B,C,D from t1,t2,t3 where p order by A desc, B asc;	-- 默认升序
```

🔵限制行数：

```sql
select A,B,C,D from t1,t2,t3 where p limit m,n;	-- 查询从第m行开始的n行数据
select A,B,C,D from t1,t2,t3 where p limit n offset m;
-- 注意 limit 1,1 为第二行
```

🔵范围值查询：

```sql
select * from customer where age between 10 and 20;
```

🔵空值检查：

❗注意：判断空值不能使用`col = NULL`，而应该是`col is NULL`

```sql
select * from toys where price is NULL;
```

逻辑操作符：`OR` , `AND`, `NOT`

范围操作符：`IN`, `LIKE`

🔵正则表达式匹配：               

```sql
select * from customer where tel regexp ".*@qq\\.com";  -- 相当于like的正则表达式版
select * from customer where name binary regexp "Lisa[a-z]{1,3}";  -- 区分大小写的正则
```

🔵拼接字段：`concat()`

```sql
select concat(name,':', tel, '.') as info from customer where p
-- ———————
-- |   info   |
-- ———————
-- | John:53477.|
-- ———————
```

🔵其他字符串函数：

* 删除空格：两边空格`Trim(col)`，左边空格`LTrim(col)`，右边空格`RTrim(col)`
* 字母大小写：`Upper(col)` `Lower(col)`
* 返回字符串长度：`Length(col)`，对应长度为len的字符串`Substring(col, pos, len)`或者`Substring(col, pos)`，这里的`pos`从1开始是第一个字符。

🔵使用子查询

> 多个select语句嵌套

举例：

```sql
select a from t1 where b in (select b from t2 where c = 'x'); -- 子查询过滤
select a (select count(*) from t2 where t1.a = t2.a) from t1; -- 子查询字段
```

**列必须匹配**：外围`where`语句中的列数目必须要和子查询中选择的列要相同，类型要兼容。

🔵组合查询：

> 俗称交、并、差（MySQL不直接支持后两种）

并（`union`, `union all`）：

```sql
select a, b from t1 union select a, b from t2;		-- 自动去重
select a, b from t1 union all select a, b from t2;	-- 不去重
```

交：

```sql
select a, b from t1 inner join t2 using (a, b);		-- 保证这n列的内容相同即可
-- 等价于
select a, b from t1 inner join t2 on t1.a = t2.a and t1.b = t2.b;
```

差：

```sql
select a, b from t1 left join t2 using (a,b) where t2.col is NULL; -- a-b
```

### 聚集过滤

常见的聚集函数`MAX()` `MIN()` `AVG()` `SUM()`，都不将`NULL`值记录在内，`count(col)`也会忽略`NULL`值，`count(*)`除外。

分组过滤：

> 未出现在`group by`子句中的属性，在select子句中必须由聚集函数包围。

```sql
select age, avg(b) from users group by age;	-- 字段b被聚集函数包围
select age, avg(b) from users group by age having age > 20;  -- group by 字段要使用having过滤
select age, avg(b) from users where b > 3 group by age having age > 20 order by age; 
```

> `group by`子句必须出现在`where`子句之后，在`order by`子句之前！`where...group by...order by...`
>
> * `where`是在分组前过滤，`having`是在分组后过滤。
>
> * `order by`是对输出后的结果进行排序，而`group by`不对数据进行排序。

### 表联结(join)

> 有时候联结要比子查询快得多

内部联结：

```sql
select * from users inner join commodity on user.id = commodity.id;
```

外部联结：

```sql
-- left outer join 可简写为 left join
select * from users left join commodity on user.id = commodity.id;
select * from users right join commodity on user.id = commodity.id;
-- 等价于using,但是using在会自动省略相同的列
select * from users right join commodity using(id);
```

### 更改数据(增删改)

insert：

```sql
insert into t1 values (NULL, 'a', 1, true); 	-- 方便但不安全，不能保证表结构改变后也是这个次序
insert into t1 (name, age, is_married) values ('a', 1, true);	-- 安全明确但繁琐
-- 一般不要使用没有明确列的insert的语句
```

insert插入多值：

```sql
-- 单条insert多数据插入要比多个单数据insert效率高
insert into t1 (name, age, is_married) values ('a', 1, true),
											 ('b', 2, false),
											 ('c', 3, false);
-- insert与select配合
insert into t1 (a, b, c) select (d, e, f) from t2 where p;
```

update:

```sql
update t1 set col='a' where p;
update ignore t1 set col='a' where p;	-- update过程中发生错误也继续执行。
-- update 支持case结构
update user set balance = case
    when age < 21 then balance + 100000
    when age >= 21 then balance * 2
    else balance + 9.99
    end
```

delete:

```sql
delete from t1 where p;
truncate t1;			-- 清空；实际上是删除表再重新创建一个表
```

### 创建和操作表

创建：

```sql
create table if not exists t1(
    id int not null auto_increment,									-- 设置自增
    name varchar(255) not null,
    age int not null check (age > 0),								-- 设置check语句
    vendor_id int not null,
    quantity int not null default 1,
    has_commodity bool null,										-- 可空
    update_time timestamp not null default CURRENT_TIMESTAMP,		 -- 设置默认值
   	primary key(id),												-- 设置主键
   	constraint vid_fk foreign key (vendor_id) references t2(id)		 -- 设置外键
   	index(name)														-- 设置索引
)engine=innodb charset=utf8mb4 comment 't1 commment';				 -- 设置引擎和字符编码
```

更改表结构：

可参考：[MySQL ALTER TABLE Statement](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html)

```sql
alter table t2 rename r1;										-- 重命名表
alter table t1 add name varchar(255);							-- 增加一列
alter table t1 drop column name;								-- 删除一列
-- modify 列名 类型， change 旧列名 新列名 类型
alter table t1 modify name varchar(128), change age years int; 	-- 更改列类型
alter table t1 add primary key (id);							-- 添加主键
alter table t1 drop primary key;								-- 删除主键
alter table t1 add index(name),add unique(card_id);				-- 添加普通索引和唯一索引
alter table t1 drop index name									-- 删除索引;
-- 添加外键
alter table t1 add constraint vid_fk foreign key(vid) references vendor(id) on delete [options];
alter table t1 drop constraint vid_fk;							-- 删除外键
```

删除表：

```sql
drop table t1;
```

重命名表：

```sql
rename table t1 to t2;
```

### 约束

🔵check子句：

```sql
create table if not exists t1(
    id int not null auto_increment,									-- 设置自增
    name varchar(255) not null,
    age int not null check (age > 0)								-- 设置check语句
)engine=innodb charset=utf8mb4;		
```

🔵外键约束：

```sql
alter table t1 add constraint vid_fk foreign key(vid) references vendor(id) on delete [options];
```

这里的options包含：（MySQL默认为`restrict`或者`no action`）

* `cascade`：父表update/delete记录时，同步update/delete掉子表的匹配记录 
* `set null`：父表update/delete记录时，将子表上匹配记录的列设为null (子表外键列不为not null)  
* `no action`：如果子表中有匹配的记录，停止对父表对应候选键进行update/delete操作 
* `restrict`：同no action, 都是立即检查外键约束

### 视图

优点：

* 重用SQL语句，简化复杂的SQL操作。
* 使用表的一部分数据，而不是整个表，可以保护数据。

操作视图：

```sql
-- 创建视图
create view uc_view as select * from users inner join commodity on user.id = commodity.id;
drop view uc_view;									-- 删除视图
create or replace view uc_view as select ...;		-- 不存在则创建，存在则替换
show create view uc_view;							-- 展示视图
```

查询操作类似普通表的查询。

> 视图一般用于检索，不用于增删改。

### 存储过程

> 相当于函数，以后再填坑

### 触发器

使用场景：

* 检查用户电话格式是否正确，地址字符是否大写
* 每订购一个产品，从仓库中减少一个产品
* 删除用户时，再存档表中增加一个记录等

支持触发器的语句：`update`, `insert`, `delete`

创建触发器的4个条件：

* 唯一的触发器名字
* 触发器关联的表
* 触发器响应的活动`update`, `insert`, `delete`
* 触发器执行的时机`before`, `after`

> 每个表最多支持6个触发器，

```sql
-- 创建触发器, 在插入表t1之后显示当前时间
create trigger insert_trigger after insert on t1 for each row select now() into @param;
select @param;		-- 显示insert记录后的结构,MySQL5之后不允许返回结果集
-- 删除触发器
drop trigger insert_trigger
-- 使用begin end包围
create trigger del_trigger before delete on orders o for each row
begin
	insert into archives(name, age, balance) values (o.name, o.age, o.balance)
end;
-- before update
create trigger up_tri before update on orders o for each row set o.name = Upper(o.name);
```

### 全局变量

```sql
show character set;							-- 展示支持的所有字符
show variables like 'character%';			-- 查看默认的字符集

show global status like "Innodb_page_size"	-- 查看数据库B+树节点支持的大小
show variables like 'autocommit';			-- 是看是否提交事务
show engine innodb status;					-- 可以查看锁的状态
select @@transaction_isolation;				-- 查看数据库当前隔离级别
```

### 账户安全管理

🔵查看当前数据库的所有用户：

```sql
use mysql;
select user from user;
```

🔵操作用户：

```sql
create user 'jack'@'host' identified by 'passwd';		-- 创建用户, % 表示可以任意主机
drop user jack;										   -- 删除用户
```

🔵授权：

```sql
show grants for root;							-- 查看授权给root的信息
grant select on demo_db.demo_table to jack;		-- 授予jack在demo_db数据库demo_table上查询权限
grant all on demo_db.* to 'jack'@'%';			-- 授予jack在demo_db数据库上所有权限
grant all on *.* to 'jack'@'%';					-- 授予jack所有权限
revoke all on *.* f 'jack'@'%';				-- 撤销jack所有权限
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');	-- 更改用户密码
```

### 导入与导出

🔵导入sql文件：

```sql
mysql -uroot -p -D customer < userinfo.sql		-- 向customer数据库导入表名为userinfo的数据
```

🔵导入csv：

```sql
load data infile '/usr/local/user.csv' 	-- CSV文件存放路径
into table student							-- 要将数据导入的表名
fields terminated by ',' optionally enclosed by '"' escaped by '"'
lines terminated by '\r\n';
```

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

导入数据的时候不应该使用索引，应用所有数据索引的总时间 < 导入每行数据分别进行添加索引总时间

### 数据结构

索引的数据结构：二叉树、红黑树、Hash表、B树、**B+树**。

🔵B树与B+树的对比：

* B树是从上到下构建树的，B+树是从下到上构建树的
* B+树的叶子节点包含所有的数据信息，支持顺序读取。
* B+树的非叶子节点只存储其子树中的最大值，因此B+树在相同大小的磁盘中能存储更多的索引。
* B+树方便取出某个范围的数据，而B树不方便。

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

- 聚簇索引：数据是和索引存放在一起的，找到索引也就找到了数据。主键索引的叶节点存储的是行数据，二级索引的叶节点存放的是对于存储行的主键值。

  当使用二级索引查找数据的时候，查找到叶节点拿到对应行的主键索引，再通过主键索引查找对应的行(**回表**)

- 非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam通过key_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在key buffer命中时，速度慢的原因

<img src="https://i.loli.net/2021/08/02/nZDXrkofUjGv2I5.png" alt="image-20210802223248752" style="zoom:67%;" />

<img src="https://i.loli.net/2021/08/02/Q2D8VXN3qAgkMwG.png" alt="image-20210802223441721" style="zoom:67%;" />

### 回表、索引覆盖、最左匹配、索引下推

* **回表**：

  id, name, age, gender. 其中id为自增主键索引，name为普通索引。

  `select * from user where name = "john"`。这个表有两个B+树。首先根据name查其B+树得到对应主键id值，在根据id值查询id对应的B+树来检索对应的数据记录，这个过程叫做回表。**尽可能避免回表操作**。`select id from user where name = "john"`，无需回表。

* **索引覆盖**：

  > 更简单理解就是：使用二级索引查询的时候，想要查询的恰好是主键，因此无需回表，这是索引覆盖。

  `select id, name from user where name = "john"`。此操作根据name的值去查询name对应的B+树，能获取到id的值，不需要获取到所有列的值，因此不需要回表，这个过程叫做索引覆盖。使用`explain`会有`using index`的提示，**推荐使用**。

  在某些场景中，可以考虑将所有的列变成组合索引，就无需回表，加快查询速度。

  `select id, name, age from user where name = "john"` 则需要回表。

* **最左匹配：**

  首先解释一下**组合索引**或者**联合索引**：可以选择多个列来共同组成索引，但是要遵循最左匹配原则。

  ![image-20220510214026139](database.assets/image-20220510214026139.png)

  比如对于联合索引 (a,b,c)，其中只有 a | a,b | a,b,c 使用查询索引才会生效。对于 b | c | b, c 这种是不会生效的。

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

```sql
-- 查看事务是否自动提交
show variables like 'autocommit';

-- 设置是否自动提交
set autocommit = 0;		-- 0表示off，1表示on
```

### 事务

<h4>四个特性(ACID)：</h4>

可参考：[深入学习MySQL事务：ACID特性的实现原理](https://www.cnblogs.com/kismetv/p/10331633.html)

🔵原子性（Atomic）：即指事务是一个不可分割的单位，其中的操作要么全部执行，要么就都不执行。其实现是基于undolog(有了undo就能够回滚)。

🔵一致性（Consistency）：数据库的完整性约束没有被破坏，事务执行前后都是合法的数据状态。其实现是**基于其他三个特性**。

🔵隔离性（Isolation）：保证事务的执行尽量不受其他事务的影响。其实现是基于锁机制和MVCC。

🔵持久性（Durability）：证事务提交之后不会因为宕机等其他因素的影响而导致数据丢失。由redolog实现，二阶段提交，WAL(write ahead log)，先写日志，再写数据。

### 并发事务的四个问题：

[并发事务带来的问题](https://www.cnblogs.com/xzsj/p/xzsj-database-transaction.html)

* 丢失数据（Lost to modify）

  指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。 例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。

* 脏读（dirty read）

  当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。

  |                    Session-1(RU)                     |              Session-2(RR)              |
  | :--------------------------------------------------: | :-------------------------------------: |
  | begin;<br/>select salary from t1 where id = 1;(5000) |                                         |
  |                                                      | update t1 set salary=4500 where id = 1; |
  |      ❌select salary from t1 where id = 1;(4500)      |                                         |
  |                                                      |                rollback;                |
  |      select salary from t1 where id = 1;(5000)       |                                         |

  处于Read-Uncommited模式下的会话一会出现脏读的情况。

* 不可重复读（unrepeatable read）

  指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。**这就发生了在一个事务内两次读到的数据是不一样的情况**，因此称为不可重复读。

  在RC隔离机制下，每次在进行快照读(select)的时候每次都会生成新的readview，在RR隔离机制下，只有第一次读的时候才会生成新的readview，来避免不可重复读的问题。

  |                    Session-1(RC)                     |              Session-2(RR)              |
  | :--------------------------------------------------: | :-------------------------------------: |
  | begin;<br/>select salary from t1 where id = 1;(5000) |                                         |
  |                                                      | update t1 set salary=4500 where id = 1; |
  |      select salary from t1 where id = 1;(5000)       |                                         |
  |                                                      |                 commit;                 |
  |      ❌select salary from t1 where id = 1;(4500)      |                                         |
  |                       commit;                        |                                         |

* 幻读（phantom read）

  幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，可能使用到了`insert` `delete` `update`语句更新了readview，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

  解决方法：添加表锁。

总结：

|     隔离级别     | 脏读 | 不可重复读 | 幻读 |
| :--------------: | :--: | :--------: | :--: |
| READ-UNCOMMITTED |  √   |     √      |  √   |
|  READ-COMMITTED  |  ×   |     √      |  √   |
| REPEATABLE-READ  |  ×   |     ×      |  √   |
|   SERIALIZABLE   |  ×   |     ×      |  ×   |

### 锁：

MySQL中锁大致可以分为全局锁、表锁和行锁三类。

* 全局锁：全局锁即对整个数据库实例加锁。使用命令 `flush tables with read lock`，就可以将整个数据库处于只读状态，使用全局锁的典型场景就是做全库逻辑备份。

* 表锁：MySQL 中有表级别的锁有两种：表锁和元数据锁(meta data lock, MDL)。

  表锁的语法是 `lock tables t1 read/write`，可以使用 `unlock tables` 主动释放锁。

  MDL 锁(MySQL 5.5 之后)不需要显式调用，在访问一个表的时候会自动被加上。当对表进行增删改查的时候加 MDL 读锁，党对表的结构进行变更的时候加 MDL 读锁。MDL锁即用来保证表结构的一致性。

* 行锁：并不是所有引擎都支持行锁（不如 MyISAM），在 InnoDB 事务中，行锁是在需要的时候才加上，当代事务结束才释放，也是两阶段锁协议。

```sql
show engine innodb status;	-- 可以查看锁的状态
```

分类：从对数据操作粒度上可分为表锁和行锁，从操作类型上可分为读锁（共享锁）和写锁（排他锁）。

🔵InnoDB行锁

对于update，delete，insert语句，innoDB会自动添加排他锁，普通select语句不会，可以使用以下语句加锁。

```sql
-- 共享锁
select * from t1 where col = 'condition' lock in share mode;
-- 排他锁
select * from t1 where col = 'condition' for update;
```

🔵行锁升级为表锁

```sql
-- 假定name为varchar类型，但是这里的1996为数字类型，会导致索引失效，行锁升级为表锁。
update t1 set age = 20 where name = 1996;
```

🔵间隙锁的影响

当使用范围条件查询而不是相等条件进行查询的时候，在请求共享或排他锁同时，innoDB会对符合条件的数据进行加锁，对于键值在条件范围内但不存在的记录，叫做“间隙”，InnoDB也会对这个间隙加锁，即间隙锁（next-key锁）

|              session-1               |                  session-2                  |
| :----------------------------------: | :-----------------------------------------: |
|          set autocommit=0;           |              set autocommit=0;              |
| update t1 set name="Q" where id < 4; |                                             |
|                                      | insert into t1 (id, name)values(2, "ring"); |
|               commit;                |                                             |

对于session-2中的`insert`语句中，其id<4，满足session1中间隙的条件，因此会阻塞直至session-1提交。

🔵查看InnoDB行锁的争用情况

`show status like 'innodb_row_lock%'`

```sh
mysql> show status like "innodb_row_lock%";
+-------------------------------+-------+
| Variable_name             | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0		|		# 当前为等待行锁的数量
| Innodb_row_lock_time			| 0		|		# 总锁定时长
| Innodb_row_lock_time_avg		| 0		|		# 每次等待平均时长
| Innodb_row_lock_time_max		| 0		|
| Innodb_row_lock_waits			| 0		|		# 系统启动后一共等待的次数
+-------------------------------+-------+
5 rows in set (0.00 sec)
```

🔵死锁以及死锁检测

<img src="database.assets/image-20220511104537334.png" alt="image-20220511104537334" style="zoom:67%;" />

出现死锁后的两种策略：

1. 等待直到超时，可以通过参数：`innodb_lock_wait_timeout` 进行设置，默认值 50s。
2. 死锁检测后主动回滚死锁链条中的一个事务。设置参数 `innodb_deadlock_detect` 表示开启这个逻辑。

一般情况下，使用第一种策略的等待时间是很难决定的，因此正常情况下通常采用第二种策略。

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

1. 隐藏字段：每一行的字段上都有用户不可见的字段

   * `DB_TRX_ID`：创建或者最后一次修改该记录的事务ID
   * `DB_ROW_ID`：即为隐藏主键，6字节的row id。
   * `DB_ROLL_PTR`：事务回滚指针，事务失败的话需要回滚到上一个历史记录，需要配合`undolog`使用。

   | name | age  | gender | DB_TRX_ID | DB_ROW_ID | DB_ROLL_PTR |
   | :--: | :--: | :----: | :-------: | :-------: | :---------: |
   | john |  18  | female |     1     |     1     |    null     |

   <img src="https://ask.qcloudimg.com/http-save/yehe-1263954/87ab1211224cb9229424fb5cb4580faa.png?imageView2/2/w/1620" alt="img" style="zoom: 80%;" />

2. undolog(回滚日志)

   如果使用update语句，会将更改的数据保存到undolog中。一个undolog会存储多个历史记录，多个事务对同一条事务进行处理的时候，会形成一个历史记录链表。

   问题：当有多个历史记录的时候，可重复读读取的是哪一个历史记录？

3. readview(读视图)

   事务在进行快照读的时候产生的读视图。

   其包含几个组件：

   * `trx_list`：系统活跃的事务ID
   * `up_limit_id`：列表中事务最小的ID
   * `low_limit_id`：系统尚未分配的下一个事务ID

   **举例：**
   情况1：

   | 事务1  | 事务2   |          事务3          |
   | :----: | :----: | :---------------------: |
   | begin; | begin; |         begin;          |
   |        |        | update ...;<br/>commit; |
   |        | 快照读 |                         |

   情况2：

   | 事务1  | 事务2   |          事务3          |
   | :----: | :----: | :---------------------: |
   | begin; | begin;<br/>快照读 |         begin;          |
   |        |        | update ...;<br/>commit; |
   |        | 快照读 |                         |

   **问题**：在RR隔离级别下，这两种情况下是否能读到最新的值？

   **readview生成时机**：

   * 在RC隔离机制下，每次在进行快照读的时候每次都会生成新的readview。
   * 在RR隔离机制下，只有第一次读的时候才会生成新的readview。

   **可见性原则算法**：

   > 当进行快照读的时候

   1. 如果`DB_TRX_ID` < `up_limit_id`， 则表示当前事务在此记录的事务发生之后开启，`DB_TRX_ID`所在记录当然对当前事务可见，如果大于等于进入下一步；
   2. 如果`DB_TRX_ID` > `low_limit_id`，则表示当前事务还发生在此记录所提交的事务之前，`DB_TRX_ID`所在的记录在`readview`生成之后才出现的，那么对当前事务不可见，如果小于进入下一步
   3. 判断`DB_TRX_ID`是否仍在活跃事务中，如果在，则表示还在`readview`的生成时刻，还没有commit，修改的数据当前事务是不可见的，如果事务在`readview`生成之前`commit`是可见的，生成之后不可见。

   分析：

   * 当前`trx_list`为事务`1, 2`，`up_limit_id`为`1`，`low_limit_id`为`4`（因为事务3已经提交）
   * 所修改记录对应的隐藏字段`DB_TRX_ID`值为`3`。
   * 因此在情况1下是可以读取到最新值，在情况2下不可以读到最新值。

   > 如果事务中存在一次当前读的时候，下一次再进行快照读的时候会更新readview。这个现象叫“幻读”，在RR事务中需要配合间隙锁来解决幻读的问题。

### 隔离级别

参考：

[彻底搞懂 MySQL 事务的隔离级别 - 阿里云社区](https://developer.aliyun.com/article/743691)

[Innodb中的事务隔离级别和锁的关系 - 美团技术团队](https://tech.meituan.com/2014/08/20/innodb-lock.html)

>MySQL的事务隔离级别一共有四个，分别是读未提交(RU, read uncommitted)、读已提交(RC, read committed)、可重复读(RR, repeatable read)以及可串行化(Serializable)。
>
>MySQL的隔离级别的作用就是让事务之间互相隔离，互不影响，这样可以保证事务的一致性。

隔离级别比较与对性能的影响：可串行化>可重复读>读已提交>读未提交

MySQL默认的事务隔离级别是RR可重复读。

查看MySQL的事务级别：`select @@transaction_isolation;`

* READ-UNCOMMITTED(读取未提交)： 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
* READ-COMMITTED(读取已提交)： 只允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
* REPEATABLE-READ(可重复读)： 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
* SERIALIZABLE(可串行化)： 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。

## MySQL 进阶

### 1. 一条SQL查询语句是如何执行的

![image-20220428215100120](database.assets/image-20220428215100120.png)

注意：MySQL 8 之后将其中的查询缓存模块删除。

其实就是考察对MySQL整体架构的了解。

在当客户端连接到数据库的时候，首先到查询缓存中查找是否命中，之前的查询结果会以 K-V 对的形式缓存在内存中，如果命中直接返回客户端。在MySQL 8.0 之前的话会查询缓存的流程，8.0 之后就没有查询缓存这个模块了。如果没有命中缓存，就开始真正执行SQL语句了，首先会经过分析器，来对SQL语句进行词法分析，查看SQL语句中是否有语句错误；经过分析器之后会进入优化器，分析SQL语句的执行逻辑和执行效率，选择一个较优的执行方案；经过优化器之后进入SQL语句执行器，开始执行的时候会判断该用户是否有对应的执行权限，如果有权限，执行器就会根据这个表的引擎定义，调用对应的存储引擎接口，最终返回SQL语句执行的结果。

### 2. 一条SQL更新语句是怎么执行的

参考：

1. [必须了解的mysql三大日志-binlog、redo log和undo log](https://segmentfault.com/a/1190000023827696)

对于SQL更新语句来说，查询语句走过的流程更新语句还是要走一遍，但是与之不同的是，更新流程还涉及两个重要的日志模块：redolog(重做日志) 和 binlog(归档日志)。

🔵重要的日志 redolog

redolog 是 InnoDB 引擎层特有的日志，记录的是**物理日志**（即SQL语句执行后的结果）。

在MySQL中更新操作存在一个问题，如果每一次更新操作都写进磁盘，首先磁盘要找到对应的那条记录然后在更新，整个过程中IO成本和查找成本都很高，为了解决这个问题，MySQL 的设计者使用了 WAL(write-Ahead logging) 技术，技术关键就是**先写日志，再写磁盘**。

redolog 包括两部分：内存中的日志缓存(redolog cache)和磁盘上的日志文件(redolog file)，向将记录写入到缓冲区，后续某个时机写入磁盘。具体来说，当更新一条记录的时候，InnoDB引擎首先会将记录写至redolog中并且更新内存缓存，这个时候就算更新完成了，同时 InnoDB 引擎会在系统比较空闲的时候将这个操作记录更新到磁盘中。

❓redolog 何时将缓存刷盘？

用户空间的缓冲区是无法直接写入磁盘的，中间必须经过操作系统的内核空间缓冲区(OS buffer)。因此 redolog buffer 刷盘的过程是先将 redolog buffer 写入 os buffer，然后通过系统调用 fsync() 将其刷到 redolog file。

![img](database.assets/1460000023827701.png)

redolog buffer 写入 redolog file的时机可以通过 `innodb_flush_log_at_trx_commit ` 参数进行配置：

* 0（延时写）：事务提交时不会立即将 redolog buffer 写入到 os buffer，而是每秒写入 os buffer 并且调用 fsync() 写入到 redolog file 中。系统崩溃的时候只会丢失 1s 的数据。
* 1（实时写，实时刷）：每次提交都会将 redolog buffer -> os buffer -> redolog file。即使系统崩溃也不会丢失数据，但是 IO 性能较差。
* 2（实时写，延迟刷）：每次提交都仅进行 redolog buffer -> os buffer，然后每秒执行 os buffer -> redolog file。

❓ 如果 redolog 被写满了怎么办？

在 InnoDB 中 redolog 是大小固定的。假如配置了一组 4 个文件，每个文件大小是 1GB，那么 redolog 就可以记录 4 GB 的操作。如果 redolog 写满就不得不将部分redolog中的数据写入磁盘，然后才能继续更新。

<img src="database.assets/image-20220508202656434.png" alt="image-20220508202656434" style="zoom:50%;" />

其中 write pos 标识当前记录的位置，checkpoint 是当前擦除的位置，擦除记录前首先要将记录更新到磁盘文件。如果 write pos 之间还有空的部分，就表示还可以继续记录新的 redolog，否则就需要停止新的更新操作，将 redolog 中的数据同步到磁盘中，将 checkpoint 推进。

如果期间数据库发生异常崩溃，之间提交的记录也不会丢失，因此 MySQL 具有崩溃后保证数据安全(crash-safe)的能力。

🔵重要的日志 binlog

binlog 是 server 层的日志，用于记录数据库执行的写入性操作（不包括查询），以二进制的方式保存在磁盘中，记录的是**逻辑日志**（可以简单理解为就是sql语句）。

binlog 是通过追加的方式写入的，可以通过 `max_binlog_size` 参数设置每个 binlog 文件的大小，当大小达到给定值之后，会生成新的文件来保存日志。

事务执行过程中，先把日志写到 binlog cache，等到事务提交的时候将 cache 中数据写到binlog文件中。

无论事务多大也要保证能够一次性写入，系统给每个线程的 binlog cache 分配一片内存，参数 `binlog_cache_size` 用于控制 binlog cache 的大小，如果超过大小就需要暂存到磁盘中。

<img src="database.assets/image-20220607165513216.png" alt="image-20220607165513216" style="zoom: 80%;" />

❓为什么要有两个日志

最初 MySQL 并没有 InnoDB 引擎，最初的 MyISAM 引擎没有 crash-safe 能力，binlog 日志只能用于归档。

binlog 的使用场景：

1. 主从复制：在 Master 端开启 binlog，然后将 binlog 发送给各个 slave，slave 端重放 binlog 从而达到主从数据一致。
2. 数据恢复：通过使用 `mysqlbinlog` 工具来恢复数据。

binlog 的刷盘时机：

对于 innodb 引擎而言，只有在事务提交的时候才会记录 binlog，那么 什么时候将 binlog 刷盘呢？ MySQL通过控制参数 `sync_binlog` 控制刷盘时机，取值范围是 `0-N`:

* 0：不强制要求，由系统自行判断何时写入磁盘
* 1：每次 commit 的时候都要将 binlog 写入磁盘
* N：每 N 个事务提交，才会将 binlog 写入磁盘

binlog 的日志格式（通过参数 `binlog-format` 指定）：

* `STATEMENT`：即SQL语句，mysql 5.7.7 之前的默认选项
* `ROW`：基于行的复制，仅记录那条数据被修改了
* `MIXED`：上述两者的混合模式，`STETEMENT` 模式无法复制的操作使用 ROW 模式保存。

🔵redolog 和 binlog 的区别

|          | redolog                              | binlog                                          |
| -------- | ------------------------------------ | ----------------------------------------------- |
| 文件大小 | redolog 文件大小是固定的             | binlog 可以通过参数进行调整每个binlog文件的大小 |
| 实现方式 | InnoDB引擎层实现的                   | server 层实现的，所有引擎都可以使用             |
| 记录方式 | redolog 采用的是循环写的方式进行记录 | binlog 是通过追加的方式进行记录                 |
| 使用场景 | redolog 可以进行崩溃恢复             | binlog 用于主从复制和数据恢复                   |
| 记录内容 | redolog 记录的是物理日志             | binlog 记录的是逻辑日志                         |

🔵更新语句的执行流程

1. 执行器调用引擎接口找到 ID = 2 的一行。由于 ID 是主键，引擎直接用 B+ 树找到对应的行。如果 ID = 2 的行数据在内存中，直接返回给执行器，否则从磁盘读取到内存中再返回。
2. 执行器拿到引擎提供的数据之后，进行update执行。
3. 引擎将这行新数据更新到 redolog buffer中，此时 redolog 处于 prepare 状态，等待执行器完成后随时提交事务。
4. 执行器生成这个 binlog，并将 binlog 写入磁盘
5. 执行器调用引擎的提交事务接口，引擎将 redolog 状态改成提交状态，更新完成。

<img src="database.assets/image-20220508215152835.png" alt="image-20220508215152835" style="zoom:67%;" />

其中深色表示在执行器中执行，浅色表示在引擎中执行。

🔵两阶段提交2PC

在进行更新语句写入 redolog 中时候，redolog 的状态分为 prepare 和 commit 状态，这就是两阶段提交。

为什么要进行两阶段提交？

假设如果不用两阶段提交：

* 如果先写 redolog 再写 binlog：如果在写 binlog 的时候系统崩溃，会出现重放redolog存在而重放binlog不存在的数据。
* 如果先写 binlog 再写 redolog：由于崩溃之后，redolog 中的这个事务无效，然后 binlog 中却存在这个数据，导致数据不一致。

如果使用两阶段提交，如果 binlog 中不存在对应的标识事务的 XID，redolog 就进行回滚，存在就进行提交。



### 3. 事务隔离级别的实现

参考：

1. [深入理解MySQL底层事务隔离级别的实现原理](https://cloud.tencent.com/developer/article/1892952)

`begin` 和 `start transaction with consistent snapshot` 的区别：

* 前者是当执行第一个操作SQL语句的时候才会启动事务，第一句快照读时生成一致性视图
* 后者是立马开启事务并且立即生成一致性视图

在 MySQL 中 每条记录在更新的时候都会同时记录一条回滚操作。事务每次开启之间，都会从数据库中获取一个自增长的事务 ID，可以根据事务 ID 判断事务执行的先后顺序。

![image-20220509100542712](database.assets/image-20220509100542712.png)

基于 MVCC 查询一条记录的流程：

1. 获取事务自己的版本号
2. 获取到 read view
3. 查询得到的数据记录，比较记录事务 ID 与 read view 中事务 ID
4. 如果不符合 Read View 可见性原则，使用 Undolog 中的记录历史快照返回符合规则的数据

InnoDB 实现 MVCC 使用过 Read View + Undo Log 实现的，Undolog 保存了记录的历史快照，Read view 中可见性原则保证了当前版本数据是否可见。

在 RC 隔离级别下，每一次查询都会生成一个新的 read view，而在 RR 隔离级别下，只有第一次查询会生成 read view；对于更新操作**都**会更新 read view。

具体样例可以参考引用链接。

🔵隔离可见性原则：

1. 版本未提交，不可见； 
2. 版本已提交，但是是在视图创建后提交的，不可见
3. 版本已提交，而且是在视图创建前提交的，可见。

### 4. InnoDB索引

索引分类**主键索引**和**非主键索引**。每个字段索引都对应一个B+树，主键索引叶子节点中存储的是整行数据，而非主键索引叶子节点中存储的是主键的值，因此索引分为**聚簇索引**和**非聚簇索引**（二级索引）。

❓什么情况下适合使用业务字段做主键（KV 场景）

1. 只有一个索引
2. 且该索引是唯一索引

使用索引的时候需要考虑回表、索引覆盖、最左匹配以及索引下推的情况。

❓为什么需要重建索引

索引可能因为删除或者页分裂的原因导致数据页有空洞，重建索引的过程会创建新的索引，这样页面的利用率会更高，索引更加紧凑且省空间。

❓选择唯一索引还是普通索引

1. 查询：

   唯一索引和普通索引的查询性能差别基本不大，唯一索引在 B+ 树上查到一个结果就会返回，普通索引会一直查找满足条件的所有记录，即使数据在下一个数据页上，对于 CPU 来说也可以忽略不计。

2. 更新：

   当更新数据页的时候，如果数据页在内存中就可以直接更新；如果不在数据页中，InnoDB 会考虑首先将更新操作缓存在 change buffer 中，等到下次查询到这个数据页的时候再将 change buffer 中的数据更新到数据页中。将 change buffer 更新到数据页的过程叫做 merge，数据库进行正常关闭的时候也会进行 merge 操作。显然如果将更新操作记录在 change buffer 中会减少读磁盘并且执行速度会提高，可以提高内存利用率。change buffer 是使用 buffer pool 中的内存，可以通过 `innodb_change_buffer_max_size` 参数设置 change buffer 占用 buffer pool 的比例。

   如果更新目标不在内存中：

   * 对于唯一索引，首先需要将数据页读取到内存中，判断是否有冲突，然后决定插入记录。
   * 对于普通索引，将记录更新到 change buffer 中，完毕。

   很明显change buffer 减少了磁盘 io 。

因此建议在选择唯一索引和普通索引的时候，如果写多读少，还是尽量选择普通索引；如果业务更新模式是数据写入后马上进行查询，即使使用了 change buffer，查询的时候会立马 merge，这非但磁盘 io 次数不会减少，反而会增加 change buffer 访问的次数。

❓字符串的前缀索引

对于字符串的索引，MySQL支持前缀索引即定义字符串的一部分作为索引。

```sql
alter table SUser add index index2(email;		# 全文索引
alter table SUser add index index2(email(6));	# 前缀索引
```

根据 B+ 树的性质可以知道，在相同条件下全文索引扫描的行数比前缀索引要少，并且全文索引可以使用索引覆盖，而前缀索引由于不知道字符串是否被截断因此必须回表。

因此，在使用前缀索引的时候需要定义好长度，才能做到技能节省空间又不用额外增加太多查询成本。

但是如果遇到前缀区分程度不够好的情况怎么办：

1. 倒序存储

   ```sql
   select field_list from t where id_card = reverse('input_id_card_string');
   ```

2. 添加 hash 字段

   ```sql
   alter table t add id_card_crc int unsigned, add index(id_card_crc);
   select field_list from t where id_card_crc=crc32('input') and id_card = `input`
   ```

​	但是这两种情况都不支持范围查询，只支持等值查询。



### 5. 数据库锁

❓如何安全的给表加字段

<img src="database.assets/image-20220511101254710.png" alt="image-20220511101254710" style="zoom:67%;" />

对于上图中的例子，Session A B 查询操作会给表 t 加入 MDL 读锁，因此 Session C 中该表表结构的操作需要阻塞等待执行完毕。轮到 Session C 执行后会将此表加入 MDL 写锁，之后的业务操作都会被阻塞，假如业务对于表 t 的查询频繁，查询超时后会一直进行重试，导致这个库的线程很快会爆掉。

解决此问题的就是需要解决长事务不提交就会一直占着 MDL 锁。可以考虑暂停 DDL 或者 kill 掉这个长事务。较为理想的机制是在 alter table 的时候设定等待的事件，如果在指定事件拿到 MDL 写锁就好，拿不到就不要阻塞后面的业务语句。之后通过重试命令重复这个操作。

```
ALTER TABLE tbl_name NOWAIT add column ...
ALTER TABLE tbl_name WAIT N add column ... 
```

🔵同一事务中语句不同的执行顺序也会影响性能

> 如果你的事务中需要锁多个行， 要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。

假设你负责实现一个电影票在线交易业务，顾客 A 要在影院 B 购买电影票。这个业务需要涉及到以下操作：

1. 从顾客 A 账户余额中扣除电影票价
2. 给影院 B 的账户余额增加这张电影票价
3. 记录一条交易日志。

如果同时有另外一个顾客 C 要在影院 B 买票，那么这两个事务冲突的部分就是语句 2 了。因为它们要更新同一个影院账户的余额，需要修改同一行数据。由于行锁都是在事务提交的时候才会释放，假如按照 312 的顺序那么影响影院账户余额的行锁事件就会表少，最大程度上减少了事务之间锁的等待，提高了并发度。

❓如何解决死锁导致的热点行性能问题

每个被事务锁住的线程，都要判断自己的加入会不会导致了死锁，这是 O(n) 的操作。假如有 1000 个并发线程同时更新一行，那么死锁检测操作就是 $O(n^2)$ 级别的，即使未检测出死锁也要耗费大量 CPU 资源，因此会看到 CPU 利用率很高但是处理不了几个事务。

如何解决：

1. 对于根本不会出现死锁的业务，可以临时把死锁检测关掉（有风险）。
2. 控制并发度，考虑中间件。
3. 将一行数据改成逻辑上的多行。以影院账户为例可以考虑余额放在多条记录上，影院的账户总额等于多个记录的值的总和。

### 6. MySQL有时候为什么会选错索引

对一个表，存有10万行数据，即(1,1,1), (2,2,2).... (100000,100000,100000)。

表的定义如下：

```sql
-- auto-generated definition
create table t
(
    id int not null
        primary key,
    a  int null,
    b  int null
);
create index a on t (a);
create index b on t (b);
```

插入数据可以使用存储过程进行复现：

```sql
delimiter ;;
create procedure idata()
begin
    declare i int;
    set i = 1;
    while (i < 100000)
        do
            insert into t
            values (i, i, i),
            set i = i + 1;
        end while;
end ;;

delimiter ;
call idata();
```

首先分析一条SQL语句：

```sql
explain select * from t where a between 10000 and 20000;
```

![image-20220514111248458](database.assets/image-20220514111248458.png)

可以看到扫描的行数时 10000 行左右，使用了索引。

但是实际应用中不会这么简单。

| SessionA                                  | SessionB                                                 |
| ----------------------------------------- | -------------------------------------------------------- |
| start transaction with consisten snapshot |                                                          |
|                                           | delete * from t;<br/>call idata()                        |
|                                           |                                                          |
|                                           | explain select * from t where a between 10000 and 20000; |
| commit                                    |                                                          |

Session B将数据全部删除之后又重新插入10万行的数据，但是此时却不会再使用索引a了。可以通过慢查询日志来具体查看执行状况。为了说明优化器选择是否正确，可以增加一个对照即使用 `force index(a)` 来让优化器强制使用索引a。

```sql
set long_query_time = 0;	# 慢查询日志开启
select * from t where a between 10000 and 20000;
select * from t force index(a) where a between 10000 and 20000;
```

看到前者时全表扫描后者只使用10001行。

🔵优化器的逻辑

优化器选择索引的目的是用最小代价执行语句。扫描行数是影响执行代价的因素之一，扫描行数越少磁盘扫描越少，占用CPU资源越少。

那么**扫描行数是怎么判断**的？

MySQL并不能精确知道满足条件的记录数目，而是通过统计信息来估计记录数目。这个统计信息就是索引的区分度，一个索引上不同的值越多说明区分度越好，称为“基数”。

可以使用 `show index` 方法查看表上索引的基数，得到的结果都是**采样**得到了，精确结果不切实际。

![image-20220514113633573](database.assets/image-20220514113633573.png)

🔵如何修正索引结果

使用analyze语句：

```sql
analyze table t;
```

🔵索引选择异常和处理

对于以下SQL语句，如果选择索引，会选择哪个索引：

```sql
select * from t where a between 1 and 1000 and b between 50000 and 100000 order by b limit 1;
```

如果使用索引a进行查询，那么就是扫描a的前1000行，然后再过滤字段b；

如果使用索引b查询，则需要扫描50000行的数据。如果使用索引a进行会更好。

但是 explain 以下发现，优化器又选错索引了。

![image-20220514115834677](database.assets/image-20220514115834677.png)

之所以优化器选择索引b，是因为他认为使用索引b可以避免排序，因为b本身就是索引已经是有序的了，遍历的时候就不需要进行排序只需要遍历。

因此可以诱导优化器选择合适的索引比如：

```sql
select * from t where a between 1 and 1000 and b between 50000 and 100000 order by b,a limit 1;
```

如果使用 `order by b,a` 的话，对两个索引字段就都需要进行排序，因此优化器就会选择需要扫描行数较少的索引了。

### 7. 删除一半表数据，表文件大小不变

因为 innodb 引擎将数据所在的位置标记为删除，但是如果之后再插入一个数据可能会复用这个数据，因此磁盘的大小不会缩小；如果删除的数据是整个数据页上的数据，则数据页可以被复用。

如果只是一条记录被删除，只有满足相应条件的记录才能插入 B+ 树；而对于数据页的删除，可能任何记录都能被插入，并且如果相邻两个数据页利用率都比较低会合并成新的数据页，将另一个数据页标记为可复用。

因此，delete 语句只是把记录或者数据页的位置标记为“可复用”，但是磁盘文件的大小是不会变的。

而且，insert 语句也会造成出现可以复用而没有被使用的空间，比如下图：

<img src="database.assets/image-20220517092712349.png" alt="image-20220517092712349" style="zoom: 50%;" />

🔵如何解决

即可以**重建表**，新建一个临时表，将原表中的数据重新插入临时表，完成后临时表替换原表即可。

在MySQL中可以使用：`alter table A engine = InnoDB` 完成上述操作。

然而插入数据的过程是较为耗时的，因此如果在此过程中如果有新的数据需要写入的话就会造成数据丢失，因此整个 DDL 的过程中表 A 不能更新，称这个 DDL 不是 online 的。

🔵online DDL

在 MySQL 5.6 之后引入了 online DDL，对流程进行了优化：

![image-20220517093817744](database.assets/image-20220517093817744.png)

1. 建立一个临时文件，扫描表 A 主键的所有数据页。
2. 使用数据页中表 A 的记录生成 B+ 树，存储到临时文件中
3. 生成临时文件的过程中，所有对表 A 的操作记录在一个日志文件（row log）中
4. 临时文件生成后，将日志文件中的操作应用到临时文件中，得到逻辑数据上与表 A 相同的数据文件。
5. 使用临时文件替换原表 A 数据文件。 

由于 alter 语句执行的过程中，给表 A 上了 MDL 锁，但是 MySQL 为了实现 online DDL，在拷贝数据的时候将 MDL 锁退化为了读锁，因此可以实现重建表期间的增删改查操作。

### 8. count(*) 

MySQL的不同引擎count(*)的实现方式也不同，MyISAM将表的行数存放在磁盘上，因此可以直接返回；而innodb需要每行进行读取。

❓为什么innodb不直接存储行数

这个innodb的事务有关，每一行记录都要判断是否对当前的会话可见，有些行对当前会话是不可见的。

并且MySQL对于count(*)查询也是进行过优化的，由于普通索引树比主键索引树要小很多，MySQL优化器会找到最小普通索引树来进行遍历，保证在逻辑正确的前提下减少扫描的数据量。

🔵不同count的效率

count(字段)<count(主键 id)<count(1)≈count(*)，对于 count(字段) 的做法，如果字段允许为null的话，是不会计数的。

❓怎么解决快速取得计数

可以使用以子之矛攻子之盾的方法，令设一个计数表用来存储数据数目。

| 会话A               | 会话B                |
| ------------------- | -------------------- |
| begin;<br/>计数值+1 |                      |
|                     | begin;<br/>查询计数; |
|                     | commit               |
| commit              |                      |

可以看到在RR隔离级别下，数据读取还是一致的。

### 9. order by 怎么工作的

对于以下SQL语句（city字段为索引）：

```sql
explain select city,name,age from t where city='杭州' order by name limit 1000;
```

查看执行计划后可以看到 `Using index condition; Using filesort`，using filesort 就是表示需要进行排序，MySQL会给每个线程分配内存用于排序，称为 sort buffer。

<img src="database.assets/image-20220518203805126.png" alt="image-20220518203805126" style="zoom:67%;" />

由于 sort buffer 的大小有限，因此如果遇到数据量较大需要排序的情况下就需要使用到外部排序，将排序结果输出到多个小文件中，然后使用归并排序合并成大文件，最终返回执行结果。

🔵全字段排序和rowid排序

对于上述查询语句，MySQL假如查询到符合条件的行记录之后会将`city,name,age`一起放到 sort buffer 中，但是如果遇到字段长度很长的话，就会导致临时文件较多排序效率降低。

<img src="database.assets/image-20220518203824088.png" alt="image-20220518203824088" style="zoom:67%;" />

对于这种情况，MySQL在存放到 sort buffer 的时候会只存储 name 和 id 主键字段，排序后再将按照索引取出 `city,name,age` 三个字段的数据返回客户端，称为 rowid 排序。

相比于全字段排序，rowid 排序要多进行一次查表。

🔵如何优化

如果原来的数据本来就是有序的，就无需进行排序。可以对 city, name 字段添加联合索引：

```sql
alter table t add index city_user(city, name)
```

查看其对应的 B+ 树数据结构可以观察到，在 city 相同的情况下，name 字段已经排序好了，就无需进行后续进入 sort buffer 的操作，遍历完毕 city 字段然后取出主键 id 找到 age 字段返回即可。

<img src="database.assets/image-20220518204432465.png" alt="image-20220518204432465" style="zoom:80%;" />

查询过程如下：

<img src="database.assets/image-20220518204644643.png" alt="image-20220518204644643" style="zoom:67%;" />

### 10. 如何正确的选择随机数据

对于英语学习 App 首页有一个随机显示单词的功能，用户级别有单词表，用户每次访问首页的时候都会随机滚动显示三个单词。随着单词表变大，选单词逻辑变得越来越慢，甚至影响到了首页的打开速度。

对应的SQL语句为：

```sql
select word from words order by rand() limit 3;
```

SQL语句虽然简单但是执行流程是复杂的。

查看其执行计划，可以发现 Extra 字段显示 `Using Temporary; Using filesort` 表示需要使用临时表和排序操作

这条语句的执行流程是这样的：

1. 创建引擎为 memory 的临时表，表里有两个字段：第一个字段为 double 类型，记为字段R，第二个字段是 varchar(64) 类型记为字段W，该表无索引。
2. 从 words 表中按照主键顺序取出所有 word 值。对于每个 word 值调用 rand() 生成 (0,1) 之间的小数，将这个小数和 word 存入到临时表中。假设全表有10000行数据，此时扫描行数为10000行。
3. 初始化 sort buffer，将内存临时表中的R值和**位置信息**存入 sort buffer 中，由于存入过程中需要扫描整个内存表，扫描行数变为20000。
4. sort buffer 根据 R 的值进行排序，排序完成后取前三个结果，根据位置信息从内存临时表取出，返回给客户端。(20003)

🔵如何解决

随机算法1：

假如只选择一个单词，该怎么做？

1. 取这个表id 的最大值M和最小值N
2. 使用随机函数生成中间数 X
3. 取不小于X的第一个ID行

```sql
select max(id), min(id) into @M, @N from t;
set @X = floor((@M-@N+1)*rand()+@N);
select * from words where id >= @X limit 1;
```

但是这种算法可能存在一个问题，即假如 ID 中间有空洞，那么选取不同行的概率就不一样了。

随机算法2：

1. 取整表行数C
2. 令 Y = floor(C*rand())
3. 再用 limit Y,1

```sql
select count(*) into @C from t;
set @Y = floor(@C * rand());
set @sql = concat("select * from words limit", @Y, ",1");
prepare stmt from @sql;
execute stmt;
DEALLOCATE prepare stmt;
```

### 11. 为什么SQL语句逻辑相同但是性能差异较大？

1. 函数

   假设数据库中存储了n年以来所有的交易数据，要统计所有年份7月份的交易总数，SQL语句大致为：

   ```sql
   select count(*) from tradelog where month(t_modified)=7;
   ```

   其中 `t_modified` 字段为索引，但是会出现SQL语句执行很久，是因为对字段做函数运算会导致索引搜索功能失效，因为对索引字段做函数操作可能会破坏索引的有序性，因此优化器放弃走B+树搜索功能。

   因此改为：

   ```sql
   select count(*) from tradelog where t_modified >= '2016-7-1' and t_modified<'2016-8-1';
   ```

2. 隐式类型转换

   ```sql
   select * from tradelog where tradeid=110717;
   ```

   比如上述SQL语句，tradeid 是varchar类型的索引字段，在MySQL中如果字符串和数字做比较的话，就会将字符串转为数字。

   ```sql
   select * from tradelog where CAST(tradid AS signed int) = 110717;
   ```

   由1可知，使用函数操作可能会导致索引搜索功能失效从而导致全表扫描。

3. 隐式字符编码转换

   对于不同表如果使用的字符集不同，即使表的对应字段都是索引也会导致索引搜索失效导致全表扫描。 


### 12. 为什么只查询一行语句也执行很慢

可能发生的情况：

1. 表锁或者行锁
2. MDL锁
3. 等待 flush

怎么定位执行较慢SQL语句的程序：

1. 使用语句 `show processlist`

2. 使用语句 `select blocking_pid from sys.schema_table_lock_waits;`

3. 如果想要查看谁占用的写锁，可以使用以下语句：

   ```sql
   select * from t sys.innodb_lock_waits where locked_table = `db.t`;
   ```

4. 然后使用 `KILL ID` 或者 `KILL QUERY ID` 来杀死对应的查询线程。

其他奇怪的结果：

```sql
select * from t where id = 1;					# 执行时间为 800 ms
select * from t where id = 1 in share mode; 	# 执行时间为 0.2 ms
```

可能出现的原因，此时对于表 t 的 id 为 1 的记录正在进行大量的修改操作，前者是快照读，因此可能需要执行查找很多的 undo log 才能找到对应的记录，而后者是当前读，可以直接读到数据。

![image-20220530214128790](database.assets/image-20220530214128790.png)

### 13. 行锁与间隙锁

假设空表 t 中的 d 字段插入对应数据分别为 0,5,10,15,20,25 六个数据，则在 Innodb 中产生 7 个间隙如下：

![image-20220531115020322](database.assets/image-20220531115020322.png)

在对数据进行加锁扫描的时候，不仅给对应的行加了行锁，还给行两边的空隙也添加了间隙锁。但是间隙锁和平常的锁不一样，间隙锁的作用就是保护这个间隙不允许插入新的值，对于间隙锁上的查询操作是不会阻塞的。

> 间隙锁只有在可重复读(RR)的隔离级别下才有效

间隙锁和行锁合称为 next-key lock，每个 next-key lock 都是前开后闭的区间。如果对于这个表 t 来说如果执行的 `select * from t for update`，就形成了 7 个 next-key 锁：(-∞,0]、(0,5]、(5,10]、(10,15]、(15,20]、(20, 25]、 (25, +∞]。

🔵加锁规则

1. 加锁的基本单位是 next-key lock，前开后闭区间，只有在查询到对应对象时候才会加锁。
2. 对于索引上的等值查询，如果是唯一索引 next-key 锁就会退化为行锁。
3. 对于索引上的等值查询，如果查询到最后一个值不满足条件的时候 next-key 锁会退化为间隙锁。

### 14. MySQL如何实现主备一致(binlog)

主备切换流程：

![image-20220608201330004](database.assets/image-20220608201330004.png)

客户端的读写都是直接访问节点A，节点B是A备库。建议将备库设置为 readonly 模式（只对同步更新有效，用户更新无效），有以下几点考虑：

1. 对于运营类的查询语句可以放到备库上查询，设置为只读模式可以防止误操作。
2. 防止切换逻辑有bug，造成双写数据不一致的情况。
3. 可以查看 readonly 的状态来判断节点的角色。

![image-20220608204347477](database.assets/image-20220608204347477.png)

<p style='text-align:center' id='主备更新'>主备更新流程图</p>

主库和备库之间是维持长连接的，两者之间事务日志同步流程为以下几个步骤：

1. 在备库 B 上通过 change master 命令，设置主库 A 的 IP、端口、用户名、密码以及开始请求 binlog 的位置、文件名和日志偏移量。
2. 在备库 B 上执行 start slave 命令，备库会启动两个线程即 `sql_thread` 以及 `io_thread`，前者负责解析并且执行日志中的命令，后者负责与主库之间建立连接。
3. 主库 A 校验完毕后，按照备库 B 传过来的位置，从本地读取 binlog 发给 B。
4. 备库 B 拿到 binlog 之后写到本地文件（中转日志 relay log）
5. `sql_thread` 读取中转日志解析并且执行。

binlog 中记录语句的方式有三种：

1. statement，只记录SQL语句。但是存在风险，比如一个删除语句 `delete from t limit 1`，假如主库和备库中使用的索引不一致就会导致删除的数据也不一致。
2. row，即记录真实的数据，对于 delete 语句会记录删除 item 的主键 id，不会存在主备不一致的情况。但是会存在占用空间较多的情况，假如一个 delete 语句要删除10万条记录，那么10万条记录都会存储到 binlog 中就会耗费较多的 IO 资源，影响执行速度。
3. mixed，即前两者的结合，MySQL 采取折中的方案，如果可能会引起主备不一致的情况就使用 row 格式，否则就使用 statement 的格式。

查看 binlog 语句：`show binlog events in 'master.000001'`

但是越来越多的场景更偏向于 `row` 格式，对于删错行或者插入错数据可以准确知道数据的具体内容。

🔵如何进行数据恢复

对于 binlog 进行恢复数据的标准做法就是使用 mysqlbinlog 工具进行解析然后使用MySQL执行。

```sql
# 从 binlog 文件中进行恢复数据
mysqlbinlog binlog.000001 binlog.000002 | mysql -u root -p
# 从 binlog 中2738-2973字节位置开始执行
mysqlbinlog master.000001 --start-position=2738 --stop-position=2973
```

### 15. MySQL 是如何保证高可用的

<img src="database.assets/image-20220609104830330.png" alt="image-20220609104830330" style="zoom:67%;" />

<p style='text-align:center'>双 M 结构切换流程</p>

在介绍高可用之前首先要明确主备延迟。

🔵主备延迟

主备切换可能是主动运维的动作(软件升级)也可能是被动操作(停电)，主备切换有三个流程：

1. 主库执行完成事务之后写入 binlog，记为 T1 时刻
2. 备库接收到 binlog 记为 T2 时刻
3. 备库执行完成事务之后即为 T3 时刻

主备延迟就是主库和备库执行完成事务的时间差 T3 - T1.

主备延迟中时间占比最大的就是备库消耗中转日志(relay log)的速度，比主库生产 binlog 的速度要慢很多。

可以使用 `show slave status` 来查看当前备库延迟了多少时间即 `seconds_behind_master`。

🔵主备延迟的原因：

1. 硬件原因，备库的机器性能要比主库差

2. 备库压力大，由于对于主库的使用较为克制，反而忽略了对备库的压力控制，因此在备库上的查询耗费了大量 CPU 资源，从而影响了同步速度造成了主备延迟。

   如何解决：

   * 一主多从，多添加几个备库来分担备库读的压力。
   * 将 binlog 输出到外部系统比如 Hadoop 这种提供统计类查询的能力

3. 大事务，如果在主库上执行很久，那么在备库上也会执行很久。

   如何解决：

   * 使用 delete 语句不要一次性删除太多的数据，分批次删除。

4. 大表的 DDL

   计划内的 DDL，建议使用 [gh-ost](https://github.com/github/gh-ost) 方案。

5. 备库的并行复制能力

🔵主备切换的策略

根据主备延迟有着不同主备切换策略：可靠性优先策略和可用性优先策略。

一般情况下建议使用可靠性优先策略来保证数据的一致性，可用性优先策略用于救急。

### 16. 备库并行复制

对于备库偶发性的查询压力，对备库延迟的影响一般都是分钟级别的，但是如果备库执行日志的速度持续低于主库生成日志的速度，那么延迟就可能达到小时级别，压力较大的情况很可能备库永远跟不上主库的节奏。

![image-20220608204347477](database.assets/image-20220608204347477.png)

并行复制能力和图中 master 中 start->undolog 和 slave 中 sql_thread->Data执行中转日志的两部分有关。假如主库中使用多线程进行读写而备库中只使用单线程进行写入数据就导致严重的主备延迟，因此回来MySQL中引入了多线程复制。

<img src="database.assets/image-20220610145035627.png" alt="image-20220610145035627" style="zoom:67%;" />

MySQL 5.6之后 sql_thread 变成了 coordinator，只负责读取中转日志和分发事务，worker 线程用于真正跟新日志，worker 的数量是由 `slave_parallel_workers` 进行决定。coordinator 进行任务分发有两个基本要求：

1. 不能造成更新覆盖，对于同样更新一行的两个事务，必须分发到同一个 worker 中
2. 同一事务中的语句不能拆开，必须存放到同一个 worker 中。

### 17. 一主多从——主备切换

![image-20220611093439940](database.assets/image-20220611093439940.png)

<p style='text-align:center'>一主多从基本结构</p>

其中 A 和 A' 是互为主备关系，从库 BCD 是 A 的备库。当一主多从结构切换完成后，A' 称为新的主库，BCD 也需要重新指向新的主库，因此主备切换的复杂性增加了。

![image-20220611094104018](database.assets/image-20220611094104018.png)

🔵基于位点的主备切换

当切换 master 的时候需要执行一条 `change master` 的命令：

```sql
change master to MASTER_HOST=$host MASTER_PORT=port MASTER_USER=$user MASTER_PASSWORD=$pass MASTER_LOG_FILE=$log_file MASTER_LOG_POS=$log_pos
```

后两个参数记录的是主库的日志文件和日志偏移量。但是两个参数是怎么确定的？

* 前者由于 A 和 A' 互为主备，日志文件都是相同的，后者偏移量很难精确获取到。

在找位点的过程中要确保不能丢失数据，总要找到一个稍微考前的位点，然后跳过在库B上已经执行过的事务。

一种取得同步位点的方法：

1. 等待新主库 A' 把中转日志(relay log)全部同步完成

2. 在 A' 库上执行 show master status 命令，获取到 A' 上最新的 File 和 position

   ```sql
   mysql> show master status;
   +---------------+----------+--------------+------------------+-------------------+
   | File       | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
   +---------------+----------+--------------+------------------+-------------------+
   | binlog.000039 |   156  |          |             |             |
   +---------------+----------+--------------+------------------+-------------------+
   ```

3. 取原主库 A 故障时刻 T

4. 使用 mysqlbinlog 工具来解析 A' 的日志文件得到 T 时刻的位点。

   ```sql
   mysqlbinlog File --stop-datetime=T --start-datetime=T
   ```

但是这个值并不精确，会存在以下这种情况：

1. 在 T 时刻，主库 A 执行完成一个 insert 语句插入一行数据 R，将 binlog 传给 A' 和 B，然后 A 遭遇断电
2. 在新主库 A' 上 R 这一行已经存在，日志是写在 Position 位置之后的
3. 在备库 B 上执行 change master 命令，指向 A' 的 File 文件 Position 位置，就会把 R 这一行的数据又同步到备库 B 上执行
4. 而这个时候 B 就会发送 `Duplicate entry` 发生主键冲突的错误，然后停止同步。

通常情况下，在切换任务的时候要主动跳过这些错误：

1. 每次碰到错误的时候停下来，执行一次跳过命令，直到不在出现停下来的情况为止。

   ```sql
   set global sql_salve_skip_counter=1;
   start slave;
   ```

2. 设置 `slave_skip_errors` 参数，直接设置跳过指定的错误

   常见的错误：

   * 1062 插入数据时遇到唯一键错误
   * 1032 删除数据时找不到对应行

   ```SQL
   # 在 my.cnt 中配置
   slave_skip_errors=1062,1053
   slave_skip_errors=all
   slave_skip_errors=ddl_exist_errors
   # 作为 mysql 启动参数的写法
   --slave-skip-errors=1062,1053
   --slave-skip-errors=all
   --slave-skip-errors=ddl_exist_errors
   ```

🔵基于 GTID 的主备切换

上述跳过事务和忽略错误的方法操作都很复杂并且很容易出错，MySQL 5.6 之后引入了 GTID。

GTID(Global Transaction Identifier)，即全局事务 ID，是一个事务在提交的时候形成的，是这个事务的唯一标识，组成格式为：`GTID=server_uuid:gno`。

GTID 启动参数：

```sh
# 在启动 MySQL 示例的时候加上参数
mysql -u root -p --gtid_mode=on --enforce_gtid_consistency=on
```

在启动 GTID 后主备切换语句为：

```sql
change master to MASTER_HOST=$host MASTER_PORT=port MASTER_USER=$user MASTER_PASSWORD=$pass master_auto_position=1
```

### 18. 一主多从——读写分离

<img src="database.assets/image-20220611093439940.png" alt="image-20220611093439940" style="zoom:80%;" />

<p style='text-align:center'>一主多从基本结构</p>

一主多从是读写分离的一种结构，读写分离的目标是分摊主库的压力，但是上面这种结构负载均衡是由client进行决定，还有一种架构是在MySQL和客户端之间有一个中间代理层 proxy，客户端只连接 proxy，由 proxy 来根据请求类型和上下文决定请求分发的路由。

![image-20220612164939864](database.assets/image-20220612164939864.png)

<p style='text-align:center'>带proxy的读写分离架构</p>

客户端直连和带 proxy 的读写分离架构特点：

* 客户端直连方案查询性能稍好，整体架构简单；但是需要了解后端部署细节，比如在出现主备切换、库迁移的过程中时候客户端都会感知，一般采用 Zookeeper 来进行管理后端组件。
* 带 proxy 架构对客户端比较友好，不需要关注后端细节。对后端维护团队要求较高需要有高可用架构，整体系统较为复杂。

⭐由于主从之间存在延迟，客户端执行完一个更新事务之后马上发起查询，如果查询操作选择的是从库的话，就很有可能读取到事务更新之前的状态。

> 从库中读取到系统过期的状态，暂定为“**过期读**”

有几种处理过期读的方案：

1. 强制走主库（简单并且使用最多）

   将查询请求就行分类，一类是必须拿到最新结果的请求，则强制发到主库；另一类是可以读到旧数据的请求，可以发送到从库。

2. sleep 方案（有点不靠谱）

   大多数主备延迟都在 1s 左右，大概率会拿到最新的数据。

3. 判断主备无延迟方案

   * 在从库每次执行查询请求之前，先判断 seconds_behind_master 是否为 0，如果为 0 才执行查询请求。
   * 使用 `show slave status` 来对比主从库的位点，Master_Log_File 和 Read_Master_Log_Pos 表示读到主库的最新位点，Relay_Master_Log_File 和 Exec_Master_Log_Pos 表示备库执行的最新位点。
   * 使用 `show slave status` 来对比 GTID 集合，Auto_Position=1 表示主备关系使用了 GTID 协议，Retrieved_Gtid_Set 表示备库收到所有日志的 GTID 集合，Executed_Gtid_Set 表示已经执行完成的 GTID 集合，两个集合如果相同就表示已经同步完成。

   但是还存在一个问题，上述判断主备无延迟是根据“备库收到日志都执行完成”，但是还有一部分日志，主库已经执行确认但是备库未收到日志的情况。

4. 配合 semi-sync 方案

   为了解决 3 中存在的问题，引入了**半同步复制**(semmi-sync replication)，主要设计为：

   1. 在事务提交的时候，主库把 binlog 发送给从库
   2. 从库收到 binlog 时候，返回主库 ack
   3. 主库收到 ack 之后才给 client 返回事务完成的确认

   启用 semi-sync 就表示所有发送给客户端发送过确认的事务都保证备库收到了这个日志。

5. 等主库位点方案

   * 使用 `show master status` 得到主库执行到的 File 和 Position
   * 选定从库执行 `select master_pos_wait(File, Position, 1)`，等待从库同步主库 binlog 到指定位置
   * 如果返回值 >= 0，则在从库执行查询语句；否则到主库执行查询语句。

6. 等 GTID 方案

   同方案 5 类似，执行等待同步语句为：`select wait_for_executed_gtid_set(gtid_set, 1)`

### 19. 如何判断数据库是否出问题了

方法：

1. SELECT 1，这个不行。这个只能判断主库的进程存在，并不能说明主库没有问题。如果出现达到并发线程上限(`innodb_thread_concurrency`)是不能察觉的，这条语句依旧能正常执行。

2. 查表判断，在系统库中创建一个表 `health_check`，`select * from mysql.health_check`，定期执行。但是不能检测空间满了的情况，比如更新事务要写 binlog，一旦 binlog 空间占用率达到 100%，那么所有更新语句和事务提交 commit 语句就会阻塞，但是读语句还是能正常执行。

3. 更新判断：在 `health_check` 表中添加一个 timestamp 字段，用于表示最后一次执行检测的时间：

   ```sql
   update mysql.health_check set t=now();
   ```

   但是在主备都执行相同的更新命令可能会出现行冲突，从而导致主备同步停止。因此还需要添加一个 ID 字段，表示主备库的 serverID 为主键

4. 使用 MySQL 内部发现问题的方法，使用库 `performance_schema`

   ```sql
   select * from performance_schema.file_summary_by_event_name where event_name='wait/io/file/innodb/innodb_log_file';
   ```


### 20. 误删数据怎么办

删除数据分为以下几类：

1. 使用 delete 语句误删数据行

   可以使用 Flashback 工具将数据恢复，使用这个方案的前提是确保 `binlog_format=row` 以及 `binlog_row_image=FULL`。

   较为推荐的做法是恢复一个备份或者找一个从库作为临时库，在临时库上进行操作，确认后再恢复到主库。这种方法防止二次破坏。

   对于事前预防要设置参数 `sql_safe_updates=on`，指定 delete 和 update 语句必须 where 条件语句。

2. 使用 drop table 或者 truncate table 误删数据表

   如果误删数据表就需要使用到**全量备份+增量日志**，这要求线上有定期的全量备份，并且实时备份 binlog。

   假如有人在中午 12 点误删一个库，恢复流程如下：

   * 取出最近一次全量备份，假设库是一天一备，上次备份的时间是 0 点
   * 用全量备份恢复出一个临时库
   * 从日志备份中取出 0 点以后的日志，将处理误删数据的语句全部应用到临时库

3. 使用 drop database 误删数据库

   搭建**延迟复制的备库**。一般主备复制结果存在问题就是主库表被删除，从库的表也会被删除。

   延迟备库是一种特殊的备库，通过 `change master to master_delay=N`，可以指定备库持续保持跟主库有 N 秒的延迟。当在 N 秒之内发现误操作这个命令之后，就可以在备库上执行 `stop slave` 命令，跳过误操作命令，恢复出需要的数据。

4. 使用 rm 命令误删整个 MySQL 实例

   有其他实例的备份，不慌

更多的方法就是预防误操作的风险：使用账号分离和指定操作规范。

### 21. 为什么还有 kill 不掉的语句

MySQL 中有两个 kill 命令：

```sql
kill query thread_id;
kill connection thread_id;
```

> thread_id 可以通过 `show processlist` 命令获取

MySQL 中的 kill 命令和 linux  中的 kill 命令类似，并不是让进程直接停止，而是给进程发送信号让其进入终止流程，比如对表进行CRUD的时候已经获取了MDL锁，如果强行停止会导致MDL锁无法释放。

MySQL 中 kill 命令做了两件事：

* 把对应连接的状态改为 `THE::KILL_QUERY`
* 然后给连接的执行线程发送信号

Kill 不掉的情况：

1. 对应线程正在阻塞，并未执行
2. 超大事务执行期间被 kill 需要进行回滚
3. 大查询回滚，由于查询过程中生成了较大临时文件
4. DDL 命令执行到最后阶段被 kill 也可能收到 IO 资源影响耗时较久。

### 23. 多数据查询会不会OOM

🔵全表扫描对 server 层的影响：

比如执行一条语句，改表大小有 200G：

```shell
mysql -u root -p -e "select * from db.t" > file
```

收发数据流程如下：

1. 获取一行数据，写到 `net_buffer` 中，buffer 大小由 `net_buffer_length` 定义，默认 16k
2. 重复获取行，直到 `net_buffer` 写满然后调用网络接口发出去。
3. 发送成功则清空 `net_buffer`，然后和继续写入发送
4. 如果发送函数返回 `EAGAIN` 或者 `WSAEWOULDBLOCK`，就表示本地网络栈(socket send buffer)写满，进入等待直到网络栈重新可写

![image-20220615133630996](database.assets/image-20220615133630996.png)

从这个流程可以看出 `net_buffer_length` 和 `socket send buffer`(由 /proc/sys/net/core/wmem_dafault 指定) 并不会达到 200G，即 MySQL 是**边读边发**的，如果客户端接收慢，很有可能导致事务执行时间变长。

🔵全表扫描对 innodb 层的影响：

在 innodb 内部，有对大查询的淘汰策略，魔改了 LRU 算法，大查询不会导致内存暴涨。

### 24. Join 的使用

在多表查询的时候，MySQL优化器首先要确定以哪个表来基准，一般情况下哪个表的结果集更小就选择哪个表为驱动表，驱动表的定义为：

1. 满足查询条件记录行数较少的表为驱动表
2. left join / straight_join 最左表为驱动表

```sql
# 字段 a 为索引
select * from t1 join t2 on (t1.a=t2.a)
```

Join 的执行流程：

1. 从表 t1 中读取一行数据 R1
2. 从数据行 R 中取出 a 字段到表 t2 中查找得到 R2（使用索引）
3. 合并 R1，R2 直到查询完毕

这个过程和嵌套查询类似，可以使用被驱动表的索引称为 `Index Nested-Loop Join` (NLJ)

<img src="database.assets/image-20220616164618672.png" alt="image-20220616164618672" style="zoom:80%;" />



Block Nested-Loop Join

对于被驱动表上的字段没有使用索引时候：

```sql
# 字段 a 为索引，b 不是索引
select * from t1 join t2 on (t1.a=t2.a)
```

<img src="database.assets/image-20220616171155915.png" alt="image-20220616171155915" style="zoom: 80%;" />

对于被驱动表上没有可用的索引，算法流程为：

1. 将表 t1 的数据读入到线程内存 join_buffer 中
2. 扫描表 t2，将表 t2 中每一行读取出来，跟 join_buffer 中的数据进行对比，返回满足条件的数据

扫面的行数是 $O(m+n)$，判断的次数时间复杂度为 $O(MN)$

因此使用 join 语句的时候尽量使用 Index NLJ，Block NLJ 尽量不要使用。

🔵join 语句怎么优化

大多数数据都是按照主键递增的顺序插入得到，如果按照主键递增查询，对磁盘的读比较接近顺序读，就能提升读性能。

```sql
# a 为索引
select * from t1 where a>=1 and a<=100;
```

MRR(multi-range read) 优化后的流程为：

1. 根据索引定位记录，将对应的主键 id 存放到结果集(read_rnd_buffer)中
2. 将结果集中的 id 进行递增排序
3. 将排序后的主键 id 回表查找记录，作为结果返回。

如果 read_rnd_buffer 存放满之后就会先执行 2 3 操作，然后清空 read_rnd_buffer 继续循环。

如果想使用 MRR 的话需要设置：`set optimizer_switch="mrr_cost_based=off"`，MRR 之所以能够提升性能是因为做索引上做的查询是范围查询（多值查询），可以得到足够多的主键 id。

Batched Key Access：

需要提前设置，BKA 算法依赖于 MRR 算法：

```sql
set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
```

BKA 算法可以看作对 Block NLJ 的优化

假如 A 表有100万行数据，B 表有 1000 行数据，BNL 算法对应时间复杂度就是 10 亿。但是在 B 表上创建索引又会浪费资源，因此可以使用临时表来完成这个操作：

```sql
create temporary table temp_t(id int primary key, a int, b int, index(b))engine=innodb;
insert into temp_t select * from t2 where b>=1 and b<=2000;
select * from t1 join temp_t on (t1.b=temp_t.b);
```

### 25. 临时表

MySQL中临时表主要分为两类：

* 内部临时表：一种是 information_schema 中的临时表；另一种执行会话中有 `using temporary` 的时候 
* 外部临时表：是通过语句 create temporary table 来创建的临时表，仅在本会话有效，表定义文件为 `frm`

创建外部临时表：

```sql
create temporary table t () engine=innodb;
```

临时表有以下几个特点：

1. 临时表只能被创建它的 session 访问，对其他线程不可见。
2. 临时表可以和普通表重名，当增删改查的时候都是使用同名表
3. show tables 不显示临时表

因此临时表很适合 join 优化的场景，不需要担心表名重复导致创建表失败，不需要担心数据删除问题，在 session 结束的时候临时表会被自动回收。

🔵外部临时表的应用

一般分库分表场景，将逻辑上的大表分散到不同的数据库实例上。比如将一个大表 ht 按照字段 f 拆分为 1024 个分表，然后分布到 32 个数据库实例上：

![image-20220618103128149](database.assets/image-20220618103128149.png)

一般情况下分库分表系统都要 proxy 层，在这个架构中，分区 key 是以选择**减少跨库和跨表查询**为依据的，如果大部分语句都包含 f 的等值条件，那么就要使用 f 做为分区键，就能够确定这条语句路由到哪个分表做查询。

如果没有使用到分区键，就需要到每个分区中查找满足条件的行。这个流程就是将所有分库拿到的数据汇总到一个 MySQL 实例中的临时表中，然后最终在汇总的临时表上进行操作。

![image-20220618110234943](database.assets/image-20220618110234943.png)

🔵外部临时表主备复制

在 session 结束的时候，会对每个临时表进行 `Drop` 操作，当 binlog_format=statement/mixed 的时候，需要写入到 binlog 中。如果当前 binlog_format 格式为 row 的时候，就不会记录临时表的操作。

🔵什么情况下会使用内部临时表

1. union 执行流程

   比如下面这条 sql 语句：

   ```sql
   (select 1000 as f) union (select id from t1 order by id desc limit 2);
   ```

   通过 explain 查看执行计划使用了 `Using temporary` 。

   执行流程如下：

   * 首先创建内存临时表，表中只有一个主键字段 f
   * 执行第一个子查询，得到 1000 的值，并存入临时表中
   * 执行第二个子查询，如果存在相同的行由于违反了主键唯一性约束插入失败；如果不存在相同的行则插入成功

   <img src="database.assets/image-20220619120008275.png" alt="image-20220619120008275" style="zoom:67%;" />

   可以看到内存临时表起到了暂存数据的作用，由于使用了临时主键表 id 的唯一性索引实现 union 的约束

   当然如果使用 union all 的话就不需要使用临时表了。

2. group by 执行流程

   ```sql
   select id%10 as m, count(*) as c from t1 group by m;
   ```

   使用 explain 查看可以看到 `Using index, Using temporary, Using filesort`，这条语句执行流程：

   * 创建内存临时表，有两个字段 m 和 c，主键是 m
   * 扫描表 t1 上的索引一次取出叶子节点上的 id 值，计算 id % 10 的结果，如果临时表中不存在则创建，存在则 +1。
   * 遍历完毕之后根据字段 m 进行排序然后返回给客户端

   <img src="database.assets/image-20220619122254782.png" alt="image-20220619122254782" style="zoom:80%;" />

   如果你不想要排序，可以在末尾添加 `order by null`

3. Group by 优化

   不论使用内存临时表还是磁盘临时表， group by 逻辑都要使用带唯一索引的表，执行代价比较高。如果表的数据量比较大的话，执行 group by 语句执行就会很慢。

   group by 语句为什么要使用临时表？group by 语句就是用来统计不同值出现的次数，但是由于每一行 id % 10 的结果是无序的就需要临时表来记录和统计计算结果。但是如果保证计算结果的数据是有序的就简单了。

   * 索引优化

     在 MySQL 5.7 之后支持了 generated column 机制，用来实现对列数据的关联更新，可以创建新的字段，来关联需要关注的字段：

     ```sql
     alter table t1 add column int generated always as (id%10\0), add index(z);
     # 之后的 group by 语句可以改为：
     select z, count(*) as c from t1 group by z;
     ```

     这个时候就既不需要排序，也不需要临时表了。

   * 直接排序

     但是不是所有的情况都适合加索引，假如一个表数据量河大，那么存放到内存临时表后还需要转成磁盘临时表，这看起来比较傻。可以直接选择走磁盘临时表，在 group by 语句中就是 `SQL_BIG_RESULT`，直接告诉优化器此表数据量大，直接使用磁盘临时表。



## 其他

### 其他链接

[面试官:讲讲mysql表设计要注意啥? - 知乎](https://zhuanlan.zhihu.com/p/73260510)

[MySQL 面试题](https://www.jishuchi.com/read/mysql-interview/2791)

[大众点评订单系统分库分表实践](https://tech.meituan.com/2016/11/18/dianping-order-db-sharding.html)
