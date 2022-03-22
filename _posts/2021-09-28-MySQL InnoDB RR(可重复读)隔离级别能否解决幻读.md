---
layout: post
title: MySQL InnoDB RR(可重复读)隔离级别能否解决幻读
date: 2021-09-28 14:00:00 +0800
category: note
thumbnail: /style/image/note.png
icon: note
---

* content
{:toc}
# 关于mysql innodb存储引擎在rr隔离级别下能否解决幻读问题?

这个问题 看国内博客简直是... 百家争鸣

[Innodb中RR隔离级别能否防止幻读?](https://github.com/Yhzhtk/note/issues/42)

这个Issue中基本覆盖了国内博客的大部分观点.

主要讨论的就是 加锁 和 事务是否为read only

其中几个有争议的点

1. 关于select 是否应该加锁

   1. 一个事务select where 查询(不加锁) 之后如果有另一个事务执行了插入并提交, 然后当前事务再次执行相同的select where for share/update(加锁) 出现了不同的结果集

   2. 前面步骤相同, 最后步骤改为不加锁, 出现相同的结果集

2. 使用update 对不存在的数据或范围数据进行修改, 且由于当前读的原因, 修改了其他事务已提交的, 但当前事务不可见的数据, 进而导致update 前后两次read 的结果不一致

其本质上讨论的就是 **`当前读/快照读`** 的问题

当前读: 通过加锁的方式, 保证当前读取的数据是最新的数据(需要对数据行加读锁/写锁), 对于select语句, 通过for share/update 显式加读锁/写锁; 对于insert/update/delete 语句, 它们只有当前读, 且会自动对相应的行加 写锁

快照读: 在RC, RR隔离级别下, 普通的select 会查询并返回当前事务可见的数据(不一定是当前最新的数据), 不加锁

## 先说结论, 不能完全解决

**可以说可以解决幻读, 但不能解决幻写... 其原因归结于规范中对于这部分的不完整定义**

因为事务四种隔离级别, 和三种问题(脏读, 不可重复读, 幻读) 都是在上世纪90年代初定义的, 可以说这个规范对当时已经够用了, 但随着数据库的不断发展, 30年过去了, 一些新技术的演变可能需要更加详细的划分, 早期的规范不适用于现在的情况. 如果硬说概念 `可以解决幻读`或者`不能解决幻读` 也没有意思.

甚至早起的规范中, 没有具体sql例子, 只有简单的文字描述. 而文字描述有时候显然是不严谨的





### SQL 92 规范

ANSI SQL-92 规范是1992年定义的一套数据库规范

[1992 SQL Standard](https://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt) 68页 描述了脏读, 不可重复读, 幻读产生的场景

> 幻读: 事务T1读取满足where条件的数据集N, 然后事务T2执行sql语句生成一条或多条满足T1条件的数据(原文中没有提到提交, 但严谨的讲. 需要等到事务T2提交之后), 然后事务T1再次重复执行相同条件的查询, 将获得不一样的结果集

> P3 ("Phantom"): SQL-transaction T1 reads the set of rows N that satisfy some <search condition>. SQL-transaction T2 then executes SQL-statements that generate one or more rows that satisfy the <search condition> used by SQL-transaction T1. If SQL-transaction T1 then repeats the initial read with the same <search condition>, it obtains a different collection of rows.

规范文档中并没有当前读和快照读的概念, 而mysql在事务第一次读取时创建快照, 并且之后每次读取(不加锁) 都是基于这份快照进行查询的

**规范中并没有提到是否需要对select进行加锁的说明**

### MySQL文档中一致性非阻塞读 & 虚拟行

[MySQL文档一致性非阻塞读](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html) 中描述了 多版本并发控制的情况

> RR隔离级别下, 同一事务中所有一致性读, 都会读取该事务第一次执行select时建立的快照(通过创建read-view 维护活跃事务id数组), 可以通过提交当前事务然后发起新的查询(RR隔离级别) 或者采用read committed 或者 采用当前读(加锁) 来获取更新的快照.  即 争议点1.2

> 但如果使用update修改数据时, 恰好修改了一条刚刚插入并且已经提交的事务的数据, 那么会出现 `修改了虚拟行`  这种情况  在 `MySQL文档一致性非阻塞读` 中的 note 处进行了sql&文字描述

> T1事务开启, 执行了select where 查询, 事务T2插入了符合事务T1 where 条件的数据并提交, 然后T1 通过update 修改where 范围内的数据, 导致原本在read-view 中标记为活跃不可见的数据, 因为对该条数据的更新操作, 使得trx_id 发生变化, 从而在下一次select where时变得可见.. 即 争议点2

[MySQL文档虚拟行](https://dev.mysql.com/doc/refman/8.0/en/innodb-next-key-locking.html) 中描述了 其他db引擎使用 where for update可能会有 insert 插入的情况

Innodb通过提供next-key锁定算法,保证使用 where for update时 会阻塞其他事务的 insert行为, 避免幻影行的情况

### 关于MySQL怎样产生幻读的提问

[怎么产生幻读](https://stackoverflow.com/questions/5444915/how-to-produce-phantom-reads)

stackOverFlow 上关于这个问题的回答挺有意思的, vote较高的两个回答, 恰好就是两个派系的.

![](https://s2.loli.net/2022/03/17/H4oWtyfe7DqnbxI.png)

这个回答就是采用的争议点2 (`MySQL文档一致性非阻塞读`中也有提到) 就不再多解释了. 但这种是否应该视为幻读? 

![](https://s2.loli.net/2022/03/17/kDd5IcPmprLqxvs.png)

而这个回答强调的是, 一个事务内所有的一致性读都是读取的第一次select 时创建的快照, 这一点在上面也说了

它还建议提问者使用其他数据库如MSSQL SERVER 或者Oracle 来演示幻读

### select 是否加锁

针对select 是否加锁的争议, 我的观点是

1. 如果前后两次select 都加锁, 那么它和 `serializable` 隔离级别一样, 因为mysql 会通过gap lock & next key lock 防止其他事务在where 范围内插入/删除

2. 如果前一次不加锁, 后一次加锁, 那么它和 `read committed` 隔离级别一样, 因为后一次通过加锁(当前读), 读取到的一定是最新一条已提交事务的数据

3. 如果前后两次select 都不加锁, 则基于MySQL的快照读机制, 创建read-view, 并保证后一次读和前一次读数据一致. **但是** 如果两次select之间 有update, update 是只有当前读的, 当前读不受read-view限制, 它会去竞争要修改的数据行的锁, 进而可能会对  trx_id 处于read-view中的数据行(快照读不可见)  进行了修改, 数据行的trx_id也被修改为当前事务的trx_id, 然后再使用普通select时 就变得可见了.. 但这种是否应该算作幻读呢?  

   > 这里update 语句  按照隔离级别来说是read committed , 因为它可以读到当前已提交事务的数据, 或者会由于获取锁而被阻塞

### P4 幻写

上面说到的92规范中, 提到了P3 幻读 phantom, (另有p1 dirty read, p2 Non-repeatable read) 这篇文章

[UNDERSTANDING MYSQL ISOLATION LEVELS: REPEATABLE-READ](https://blog.pythian.com/understanding-mysql-isolation-levels-repeatable-read/) 给出了更多 `"幻写"` 的例子, 并给出以下结论

MySQL 实现了可重复读隔离级别：

* 仅使用 select 语句时比 SQL Standard 更严格，因为不会发生幻读。此外，快照用于所有表和所有行

* 当事务修改数据时，行为是可重复读取（未修改的行不可见）和已提交（修改的行可见）的混合。我们不能说这不是标准，因为规范中没有描述这些情况，并且不适合三种并发现象：脏读、不可重复读和幻读。

* 当事务根据现有数据写入新数据时，它使用提交的数据，而不是之前检索到的快照。这对修改的和新的行都有效，模仿读已提交的行为。

### SQL-92缺陷

[A Critique of ANSI SQL Isolation Levels](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf) 中 remark 5 中指出**ANSI SQL 隔离现象不完整**, 有许多异常情况可能会出现, 需要重新定义 `phenomena` 并且P3 phantom 需要重新解释

------

### 补充 trx_id & undo log的一点内容

创建一个简单的表有 id, name, age, address 等字段.

当创建成功后, 实际字段会比创建时要多, 会有以下隐藏列

1. 如果表中没有指定primary key, 则会选择一个not null unique 列作为主键, 如果没有, 选择唯一索引作为主键, 如果还没有, 创建一个隐式主键row_id (用户不可见)
2. trx_id 事务id 隐藏列, 用于记录数据最后一次修改的事务id, 事务快照维护的read-view 中, 记录了当前事务不可见的trx_id
3. roll_pointer 回滚指针 隐藏列, 用于记录一条数据的版本链, 它指向了undo log中该数据行的上一个版本. 
4. 如果使用b+树, 还会有next_record 记录下一条数据位置

事务内执行select时, 查到一条数据, 可能trx_id 在当前事务read_view中, 于是通过回滚指针查找回滚日志中 该条数据的历史版本, 如果这条数据的trx_id 仍然在read_view中, 则继续寻找它的历史版本, 直到找到一条可见版本, 或者空



-------



## 总结

如果前面这些内容都有了了解, 那看到这里, 去争论MySQL InnoDB RR隔离级别是否存在幻读已经不重要了, 毕竟基于一个不完整的规范, 或者说这里需要再细分具体的幻读和幻写;

但如果面试时遇到这个问题, 可能需要详细给他扯一扯. 需要讲清楚 MySQL InnoDB RR 隔离级别 的快照, 快照读 & 当前读

在只读时是可以避免幻读的

在读写时可能会因为update操作使得不可见的行变得可见, 从而出现幻影行









