# MySQL实战45讲

# Mysql操作

```mysql
#连接
mysql -hlocalhost -uroot -p

```



# 第3讲 事务隔离-1

## 1. 隔离性与隔离级别

当数据库上有多个事务同时执行的时候，就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，为了解决这些问题，就有了“隔离级别”的概念。隔离得越严实，效率就会越低。因此很多时候，我们都要在二者之间寻找一个平衡点。SQL 标准的事务隔离级别包括：读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable ）

读未提交：一个事务还没提交时，它做的变更就能被别的事务看到。

读提交：一个事务提交之后，它做的变更才会被其他事务看到。

可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。

串行化：顾名思义是对于同一行记录，“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。

<img src="https://static001.geekbang.org/resource/image/7d/f8/7dea45932a6b722eb069d2264d0066f8.png" alt="img" style="zoom: 50%;" />

不同隔离级别下的 V1、V2、V3的值为：

在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。在“可重复读”隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。在“读提交”隔离级别下，这个视图是在每个 SQL 语句开始执行的时候创建的。这里需要注意的是，“读未提交”隔离级别下直接返回记录上的最新值，没有视图概念；而“串行化”隔离级别下直接用加锁的方式来避免并行访问。

可重复读的业务场景：

假设你在管理一个个人银行账户表。一个表存了账户余额，一个表存了账单明细。到了月底你要做数据校对，也就是判断上个月的余额和当前余额的差额，是否与本月的账单明细一致。你一定希望在校对过程中，即使有用户发生了一笔新的交易，也不影响你的校对结果。

## 2. 事务隔离的实现

在mysql中。每条记录在更新时都会同时记录一条回滚操作在(undo log)回滚日志中，通过回滚操作就可以得到前一个状态的值。同一条记录在系统中可以存在多个版本，就是数据库的多版本并发控制（MVCC）

![img](https://static001.geekbang.org/resource/image/d9/ee/d9c313809e5ac148fc39feff532f0fee.png)

**什么时候删除回滚日志？**

当系统判断，没有事务再需要用到这些回滚日志时就会删除，即当系统里没有比这个回滚日志更早的read-view时。所以一般不建议使用长事务，因为长事务就意味着系统中会存在很多很老的事务视图，导致回滚日志占据大量的内存空间

## 3. 事务的启动方式

1. 显式启动事务语句， begin 或 start transaction。配套的提交语句是 commit，回滚语句是 rollback。
2. set autocommit=1, 通过显式语句的方式来启动事务。

建议方式：**set autocommit=1, 通过显式语句的方式来启动事务。**

autocommit只涉及数据库是否帮你提交事务，如果你没有显示的开启一个事务，那么数据库会帮你自动开启一个事务的。autocommit = 1 为开启自动提交。

begin/start transaction 命令并不是一个事务的起点，在执行到它们之后的第一个操作 InnoDB 表的语句，事务才真正启动。如果你想要马上启动一个事务，可以使用 start transaction with consistent snapshot 这个命令。

# 第8讲 事务隔离-2

**考虑可重复读隔离级别和行锁情况下的数据更新问题**

### Mysql中的视图

在 MySQL 里，有两个“视图”的概念：

一个是 view。它是一个用查询语句定义的虚拟表，在调用的时候执行查询语句并生成结果。创建视图的语法是 create view … ，而它的查询方法与表一样。

另一个是 InnoDB 在实现 MVCC 时用到的一致性读视图，即 consistent read view，用于支持 RC（Read Committed，读提交）和 RR（Repeatable Read，可重复读）隔离级别的实现。

![img](https://static001.geekbang.org/resource/image/82/d6/823acf76e53c0bdba7beab45e72e90d6.png)

解释：

事务 C 没有显式地使用 begin/commit，表示这个 update 语句本身就是一个事务，语句完成的时候会自动提交。

## 1. 快照在MVCC里的工作

在可重复读隔离级别下，事务在启动的时候就“拍了个快照”。注意，这个快照是基于整库的。

InnoDB 里面每个事务有一个唯一的事务 ID，叫作 transaction id。它是在事务开始的时候向 InnoDB 的事务系统申请的，是按申请顺序严格递增的。

![img](https://static001.geekbang.org/resource/image/68/ed/68d08d277a6f7926a41cc5541d3dfced.png)

实际上，图 2 中的三个虚线箭头，就是 undo log；而 V1、V2、V3 并不是物理上真实存在的，而是每次需要的时候根据当前版本和 undo log 计算出来的。比如，需要 V2 的时候，就是通过 V4 依次执行 U3、U2 算出来。

因此，一个事务只需要在启动的时候声明说，“以我启动的时刻为准，如果一个数据版本是在我启动之前生成的，就认；如果是我启动以后才生成的，我就不认，我必须要找到它的上一个版本”。当然，如果“上一个版本”也不可见，那就得继续往前找。还有，如果是这个事务自己更新的数据，它自己还是要认的。

在实现上， InnoDB 为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃”的所有事务 ID。“活跃”指的就是，启动了但还没提交。

### 1.1 当前读

更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”（current read）。

除了 update 语句外，select 语句如果加锁，也是当前读。

```mysql
mysql> select k from t where id=1 lock in share mode;	//加 读(S锁，共享锁) 
mysql> select k from t where id=1 for update;			//加 写锁(X 锁，排他锁)
```

可重复读的核心就是一致性读（consistent read）；而事务更新数据的时候，只能用当前读。如果当前的记录的行锁被其他事务占用的话，就需要进入锁等待。

### 1. 2读提交隔离状态下的事务状态

而读提交的逻辑和可重复读的逻辑类似，它们最主要的区别是：

在可重复读隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图；

在读提交隔离级别下，每一个语句执行前都会重新算出一个新的视图。

![img](https://static001.geekbang.org/resource/image/18/be/18fd5179b38c8c3804b313c3582cd1be.jpg)

这时，事务 A 的查询语句的视图数组是在执行这个语句的时候创建的，时序上 (1,2)、(1,3) 的生成时间都在创建这个视图数组的时刻之前。但是，在这个时刻：(1,3) 还没提交，属于情况 1，不可见；(1,2) 提交了，属于情况 3，可见。所以，这时候事务 A 查询语句返回的是 k=2。显然地，事务 B 查询结果 k=3。

# 第6-7讲 MySQL锁

​		数据库锁设计的初衷是处理并发问题。作为多用户共享的资源，当出现并发访问的时候，数据库需要合理地控制资源的访问规则。而锁就是用来实现这些访问规则的重要数据结构。

## 1. 全局锁

**语法：Flush tables with read lock (FTWRL)**

1.MySQL 提供了一个加全局读锁的方法，命令是 Flush tables with read lock (FTWRL)。使用这个命令之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句。

典型使用场景：全库逻辑备份

缺点：

1. 如果你在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆；

2.  如果你在从库上备份，那么备份期间从库不能执行主库同步过来的 binlog，会导致主从延迟。

但是如果不加全局锁的话会出现的问题：

不加锁的话，备份系统备份的得到的库不是一个逻辑时间点，这个视图是逻辑不一致的。（在数据库引擎支持事务的前提下，也可以通过MVCC的办法保证拿到一致性视图，且开销比全局锁要小）

官方自带的逻辑备份工具是 mysqldump。当 mysqldump 使用参数–single-transaction 的时候，导数据之前就会启动一个事务，来确保拿到一致性视图。而由于 MVCC 的支持，这个过程中数据是可以正常更新的。

## 2. 表级锁
### 2.1 表锁
MySQL里面的表级别锁有两种：表锁和元数据锁(meta data lock MDL)

语法：lock tables … read/write（加锁） unlock table （解锁）
表锁：分为共享读锁（共享锁）和表独占写锁（排它锁）

**表锁有什么好处？**

（1）表锁占用内存少很多，行锁的数量与行记录数相关，非常耗内存；

（2）如果业务经常读写表中很大一部分数据时，表锁会更快，因为此时只涉及一个锁，而不是同时管理N多个锁；

（3）如果业务经常使用group by，表锁会更快，原因同（2）；

**表锁是怎么运作的？**

和其他临界资源的读写锁类似。

1. 写时，要加写锁：

（1）如果表没有锁，对表加写锁；

（2）否则，入写锁队列；

2. 读时，要加读锁：

（1）如果表没有写锁，对表加读锁；

（2）否则，入读锁队列；

**表锁释放时**：

如果写锁队列和读锁队列里都有锁，写有更高的优先级，即写锁队列先出列。这么做的原因是，如果有“大查询”，可能会导致写锁被批量“饿死”，而写锁往往释放很快。

如果有大量并发update请求，select会等所有update请求执行完才执行。
### 2.2 MDL

**MDL作用是防止DDL和DML并发的冲突**

不需要显示的使用，会在访问一个表的时候自动加上，保证读写的正确性。

例如：如果一个查询正在遍历一个表中的数据，而执行期间另一个线程对这个表结构做变更，删了一列，那么查询线程拿到的结果跟表结构对不上，肯定是不行的。

因此，在 MySQL 5.5 版本中引入了 MDL：

1. 当对一个表做增删改查操作的时候，加 MDL 读锁；

2. 当要对表做结构变更操作的时候，加 MDL 写锁。

**特点：**

1. 读锁之间不互斥，因此你可以有多个线程同时对一张表增删改查。
2. 读写锁之间、写锁之间是互斥的，用来保证变更表结构操作的安全性。因此，如果有两个线程要同时给一个表加字段，其中一个要等另一个执行完才能开始执行。
3. 事务中的 MDL 锁，在语句执行开始时申请，但是语句结束后并不会马上释放，而会等到整个事务提交后再释放。

## Question

Question1：如何安全的给小表加字段

1. 首先我们要解决长事务，事务不提交，就会一直占着 MDL 锁。在 MySQL 的 information_schema 库的 innodb_trx 表中，你可以查到当前执行中的事务。如果你要做 DDL 变更的表刚好有长事务在执行，要考虑先暂停 DDL，或者 kill 掉这个长事务。

Question2：如果你要变更的表是一个热点表，虽然数据量不大，但是上面的请求很频繁，而你不得不加个字段，你该怎么做呢？

1. 这时候 kill 可能未必管用，因为新的请求马上就来了。比较理想的机制是，在 alter table 语句里面设定等待时间，如果在这个指定的等待时间里面能够拿到 MDL 写锁最好，拿不到也不要阻塞后面的业务语句，先放弃。之后开发人员或者 DBA 再通过重试命令重复这个过程。

Question3：备份一般都会在备库上执行，你在用–single-transaction 方法做逻辑备份的过程中，如果主库上的一个小表做了一个 DDL，比如给一个表上加了一列。这时候，从备库上会看到什么现象呢？

## 3. 行锁

两阶段协议 死锁和死锁检测

### 3.1 两阶段协议：

​	在 InnoDB 事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放。这个就是两阶段锁协议。

Tips：如果你的事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放，最大程度的减少事务之间的锁等待，提升并发度。

### 3.2 死锁和死锁检测

死锁：当并发系统中不同线程出现循环资源依赖，涉及的线程都在等待别的线程释放资源时，就会导致这几个线程都进入无限等待的状态，称为死锁。

死锁的解决办法：

1. 直接进入等待，直到超时。这个超时时间可以通过参数 innodb_lock_wait_timeout 来设置。
2. 发起死锁检测，发现死锁后，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数 innodb_deadlock_detect 设置为 on，表示开启这个逻辑。

方法一的实时性很差，对于在线业务而言是无法接受的，但是方法二的死锁检测会消耗大量的CPU资源，对于所有的事务都更新同一行时，每个新来的线程都需要判断会不会由于自己的加入导致死锁，是一个时间复杂度O(N^2)的。

方法二的优化方法：

1. 控制并发度，降低同时操作同一行的线程数量，可以通过中间件实现，或者修改MySQL的源码，使之在进入引擎前排队。

# 第34讲 join

## Mysql添加删除索引

```mysql
#1.添加PRIMARY KEY（主键索引）：

ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) 

#2.添加UNIQUE(唯一索引) ：

ALTER TABLE `table_name` ADD UNIQUE ( `column` ) 
 
#3.添加INDEX(普通索引) ：
ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
 
#4.添加FULLTEXT(全文索引) ：
ALTER TABLE `table_name` ADD FULLTEXT ( `column`) 
 
#5.添加多列索引：
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )

#删除索引
ALTER TABLE `table_name` DROP INDEX index_name;
```

这两个表都有一个主键索引 id 和一个索引 a，字段 b 上无索引。存储过程 idata() 往表 t2 里插入了 1000 行数据，在表 t1 里插入的是 100 行数据。

```mysql
CREATE TABLE `t2` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`)
) ENGINE=InnoDB;

drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=1000)do
    insert into t2 values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();

create table t1 like t2;
insert into t1 (select * from t2 where id<=100)
```

## 1. Index Nested-Loop Join(NLJ)

### STRAIGHT_JOIN

STRAIGHT_JOIN只适用于内连接，因为left join、right join已经知道了哪个表作为驱动表，哪个表作为被驱动表，比如left join就是以左表为驱动表，right join反之，而STRAIGHT_JOIN就是在内连接中使用，而强制使用左表来当驱动表，所以这个特性可以用于一些调优，强制改变mysql的优化器选择的执行计划

```mysql
select * from t1 straight_join t2 on (t1.a=t2.a);
```

在这条语句里，被驱动表 t2 的字段 a 上有索引，join 过程用上了这个索引，因此这个语句的执行流程是这样的：

### NLJ流程分析

“Index Nested-Loop Join”，简称 NLJ

1. 从表 t1 中读入一行数据 R；
2. 从数据行 R 中，取出 a 字段到表 t2 里去查找；
3. 取出表 t2 中满足条件的行，跟 R 组成一行，作为结果集的一部分；
4. 重复执行步骤 1 到 3，直到表 t1 的末尾循环结束。

这个过程是先遍历表 t1，然后根据从表 t1 中取出的每行数据中的 a 值，去表 t2 中查找满足条件的记录。在形式上，这个过程就跟我们写程序时的嵌套查询类似，并且可以用上被驱动表的索引，所以我们称之为“Index Nested-Loop Join”，简称 NLJ。

### NLJ空间复杂度

1. 对驱动表 t1 做了全表扫描，这个过程需要扫描 100 行；
2. 而对于每一行 R，根据 a 字段去表 t2 查找，走的是树搜索过程。由于我们构造的数据都是一一对应的，因此每次的搜索过程都只扫描一行，也是总共扫描 100 行；
3. 所以，整个执行流程，总扫描行数是 200。

### NLJ时间复杂度

在这个 join 语句执行过程中，驱动表是走全表扫描，而被驱动表是走树搜索。

假设被驱动表的行数是 M。每次在被驱动表查一行数据，要先搜索索引 a，再搜索主键索引。每次搜索一棵树近似复杂度是以 2 为底的 M 的对数，记为 log2M，所以在被驱动表上查一行的时间复杂度是 2*log2M。*

假设驱动表的行数是 N，执行过程就要扫描驱动表 N 行，然后对于每一行，到被驱动表上匹配一次。

因此整个执行过程，近似复杂度是 N + N*2*log2M。

显然，N 对扫描行数的影响更大，因此应该让小表来做驱动表。

## 2. Simple Nested-Loop Join

如果修改上述的SQL语句为：

```mysql
select * from t1 straight_join t2 on (t1.a=t2.b);
```

由于表 t2 的字段 b 上没有索引，因此再用图 2 的执行流程时，每次到 t2 去匹配的时候，就要做一次全表扫描。这样算来，这个 SQL 请求就要扫描表 t2 多达 100 次，总共扫描 100*1000=10 万行，这个算法也有一个名字，叫做“Simple Nested-Loop Join”，当然，MySQL 也没有使用这个 Simple Nested-Loop Join 算法，而是使用了另一个叫作“Block Nested-Loop Join”的算法，简称 BNL。

## 3. Block Nested-Loop Join(BNL)

不使用索引字段 join 的 explain 结果：

![img](https://static001.geekbang.org/resource/image/67/e1/676921fa0883e9463dd34fb2bc5e87e1.png)

### BNL流程分析

1. 把表 t1 的数据读入线程内存 join_buffer 中，由于我们这个语句中写的是 select *，因此是把整个表 t1 放入了内存；
2. 扫描表 t2，把表 t2 中的每一行取出来，跟 join_buffer 中的数据做对比，满足 join 条件的，作为结果集的一部分返回。

### BNL时间复杂度和空间复杂度分析

在这个过程中，对表 t1 和 t2 都做了一次全表扫描，因此总的扫描行数是 1100。由于 join_buffer 是以无序数组的方式组织的，因此对表 t2 中的每一行，都要做 100 次判断，总共需要在内存中做的判断次数是：100*1000=10 万次。

前面我们说过，如果使用 Simple Nested-Loop Join 算法进行查询，扫描行数也是 10 万行。因此，从时间复杂度上来说，这两个算法是一样的。但是，Block Nested-Loop Join 算法的这 10 万次判断是内存操作，速度上会快很多，性能也更好。

### 分段join_buffer

当join_buffer放不下驱动表的数据时，会进行分段存放。

join_buffer 的大小是由参数 join_buffer_size 设定的，默认值是 256k。

分段BNL的执行流程

1. 扫描表 t1，顺序读取数据行放入 join_buffer 中，放完第 88 行 join_buffer 满了，继续第 2 步；
2. 扫描表 t2，把 t2 中的每一行取出来，跟 join_buffer 中的数据做对比，满足 join 条件的，作为结果集的一部分返回；
3. 清空 join_buffer；
4. 继续扫描表 t1，顺序读取最后的 12 行数据放入 join_buffer 中，继续执行第 2 步。

![img](https://static001.geekbang.org/resource/image/69/c4/695adf810fcdb07e393467bcfd2f6ac4.jpg)



## 4. 是否使用 join 的问题

**第一个问题：能不能使用 join 语句？**

1. 如果可以使用 Index Nested-Loop Join 算法，也就是说可以用上被驱动表上的索引，其实是没问题的；
2. 如果使用 Block Nested-Loop Join 算法，扫描行数就会过多。尤其是在大表上的 join 操作，这样可能要扫描被驱动表很多次，会占用大量的系统资源。所以这种 join 尽量不要用。

所以你在判断要不要使用 join 语句时，就是看 explain 结果里面，Extra 字段里面有没有出现“Block Nested Loop”字样。

**第二个问题是：如果要使用 join，应该选择大表做驱动表还是选择小表做驱动表？**

1. 如果是 Index Nested-Loop Join 算法，应该选择小表做驱动表；

2. 如果是 Block Nested-Loop Join 算法

   1. 在 join_buffer_size 足够大的时候，是一样的；

   2. 在 join_buffer_size 不够大的时候（这种情况更常见），应该选择小表做驱动表。

所以，这个问题的结论就是，总是应该使用小表做驱动表。

所以，更准确地说，在决定哪个表做驱动表的时候，应该是两个表按照各自的条件过滤，过滤完成之后，计算参与 join 的各个字段的总数据量，数据量小的那个表，就是“小表”，应该作为驱动表。

# 第35讲 join优化

表 t1 里，插入了 1000 行数据，每一行的 a=1001-id 的值。也就是说，表 t1 中字段 a 是逆序的。

表 t2 中插入了 100 万行数据。

主键索引是 id , 普通索引 a

```mysql
create table t1(id int primary key, a int, b int, index(a));
create table t2 like t1;
drop procedure idata;
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=1000)do
    insert into t1 values(i, 1001-i, i);
    set i=i+1;
  end while;
  
  set i=1;
  while(i<=1000000)do
    insert into t2 values(i, i, i);
    set i=i+1;
  end while;

end;;
delimiter ;
call idata();
```

## Multi-Range Read 优化

Multi-Range Read 多范围读(MRR)，针对基于辅助/第二索引的查询，减少随机IO，并且将随机IO转化为顺序IO，提高查询效率。

设置MRR优化开启的方式：（官方文档的说法，是现在的优化器策略，判断消耗的时候，会更倾向于不使用 MRR，把 mrr_cost_based 设置为 off，就是固定使用 MRR 了。）

```mysql
select * from t1 where a>=1 and a<=100;
```

![img](https://static001.geekbang.org/resource/image/97/05/97ae269061192f6d7a632df56fa03605.png)

如果随着 a 的值递增顺序查询的话，id 的值就变成随机的，那么就会出现随机访问，性能相对较差。

因为大多数的数据都是按照主键递增顺序插入得到的，所以我们可以认为，如果按照主键的递增顺序查询的话，对磁盘的读比较接近顺序读，能够提升读性能。

MRR流程：

1. 根据索引 a，定位到满足条件的记录，将 id 值放入 read_rnd_buffer 中 ;
2. 将 read_rnd_buffer 中的 id 进行递增排序；
3. 排序后的 id 数组，依次到主键 id 索引中查记录，并作为结果返回。

这里，read_rnd_buffer 的大小是由 read_rnd_buffer_size 参数控制的。如果步骤 1 中，read_rnd_buffer 放满了，就会先执行完步骤 2 和 3，然后清空 read_rnd_buffer。之后继续找索引 a 的下个记录，并继续循环。

![img](https://static001.geekbang.org/resource/image/d5/c7/d502fbaea7cac6f815c626b078da86c7.jpg)

MRR 能够提升性能的核心在于，这条查询语句在索引 a 上做的是一个范围查询（也就是说，这是一个多值查询），可以得到足够多的主键 id。这样通过排序以后，再去主键索引查数据，才能体现出“顺序性”的优势。

## Batched Key Access(BAK)

MySQL 在 5.6 版本后开始引入的 Batched Key Access(BKA) 算法了。这个 BKA 算法，其实就是对 NLJ 算法的优化。

BKA优化算法设置：

```mysql
set optimizer_switch='mrr=on,mrr_cost_based=off,batched_key_access=on';
#其中，前两个参数的作用是要启用 MRR。这么做的原因是，BKA 算法的优化要依赖于 MRR。
```

### NLJ的BAK优化

NLJ 算法执行的逻辑是：从驱动表 t1，一行行地取出 a 的值，再到被驱动表 t2 去做 join。也就是说，对于表 t2 来说，每次都是匹配一个值。这时，MRR 的优势就用不上了。把表 t1 的数据取出来一部分，先放到 join_buffer 中。

![img](https://static001.geekbang.org/resource/image/68/88/682370c5640244fa3474d26cc3bc0388.png)

### BNL的BAK优化

1. 直接在被驱动表上建索引，就可以直接转成BKA算法
2. 当确实不适合在被驱动表建立索引时： 使用临时表

在表 t2 中插入了 100 万行数据，但是经过 where 条件过滤后，需要参与 join 的只有 2000 行数据。如果这条语句同时是一个低频的 SQL 语句，那么再为这个语句在表 t2 的字段 b 上创建一个索引就很浪费了。

在表 t2 的字段 b 上创建索引会浪费资源，但是不创建索引的话这个语句的等值条件要判断 10 亿次，想想也是浪费。那么，有没有两全其美的办法呢？

使用临时表的思路：

1. 把表 t2 中满足条件的数据放在临时表 tmp_t 中；
2. 为了让 join 使用 BKA 算法，给临时表 tmp_t 的字段 b 加上索引；
3. 让表 t1 和 tmp_t 做 join 操作。

```mysql
create temporary table temp_t(id int primary key, a int, b int, index(b))engine=innodb;
insert into temp_t select * from t2 where b>=1 and b<=2000;
select * from t1 join temp_t on (t1.b=temp_t.b);
```

加入临时表后的优化流程：

1. 执行 insert 语句构造 temp_t 表并插入数据的过程中，对表 t2 做了全表扫描，这里扫描行数是 100 万。
2. 之后的 join 语句，扫描表 t1，这里的扫描行数是 1000；join 比较过程中，做了 1000 次带索引的查询。相比于优化前的 join 语句需要做 10 亿次条件判断来说，这个优化效果还是很明显的。

总体来看，不论是在原表上加索引，还是用有索引的临时表，我们的思路都是让 join 语句能够用上被驱动表上的索引，来触发 BKA 算法，提升查询性能。

## hash-join

hash-join的好处是做匹配时的时间复杂度是O(1)；

其实上面计算 10 亿次那个操作，看上去有点儿傻。如果 join_buffer 里面维护的不是一个无序数组，而是一个哈希表的话，那么就不是 10 亿次判断，而是 100 万次 hash 查找。

可以通过业务端的逻辑实现

1. select * from t1;取得表 t1 的全部 1000 行数据，在业务端存入一个 hash 结构，比如 C++ 里的 set、PHP 的数组这样的数据结构。
2. select * from t2 where b>=1 and b<=2000; 获取表 t2 中满足条件的 2000 行数据。
3. 把这 2000 行数据，一行一行地取到业务端，到 hash 结构的数据表中寻找匹配的数据。满足匹配的条件的这行数据，就作为结果集的一行。
