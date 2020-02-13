---
title: 数据库的事务以及事务隔离级别
excerpt: 数据库事务是数据库管理系统执行过程中的一个逻辑单位，包含了一个有限的数据库操作序列。事务不是天然存在的东西，它是被人为创造出来，目的是简化应用层的编程模型。数据库事务有两个目的：（1）定义一个逻辑单位，以便于当数据库从故障中恢复的时候能正确地恢复数据；（2）提供一种隔离方式，以便于解决多个客户端并发操作的时候出现的奇怪问题。
date: 2020-02-14 00:01:02
tags: Database
---

## 事务是什么
数据库事务是数据库管理系统执行过程中的一个逻辑单位，包含了一个有限的数据库操作序列。事务不是天然存在的东西，它是被人为创造出来，目的是简化应用层的编程模型。数据库事务有两个目的：

* 定义一个逻辑单位，以便于当数据库从故障中恢复的时候能正确地恢复数据
* 提供一种隔离方式，以便于解决多个客户端并发操作的时候出现的奇怪问题

## ACID是什么

数据库系统在实现事务的时候必须满足4个条件：

|简称|中文|说明|
|---|---|---|
|Atomic|原子性|事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行|
|Consistent|一致性|事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束|
|Isolated|隔离性|多个事务并发执行时，一个事务的执行不应影响其他事务的执行|
|Durable|持久性|已被提交的事务对数据库的修改应该永久保存在数据库中|

这4个条件构成了常说的ACID模型。数据库系统在实现事务的时候要遵照ACID模型，也就是说数据库事务必须是：
* 原子的（它必须完整地完成或不起作用）
* 一致的（它必须符合数据库中现有的约束）
* 隔离的（它不能影响其他事务）
* 持久的（必须将其写入持久性存储）

## 事务的隔离级别是什么

在数据库系统中，隔离度决定了其他用户和系统如何看到事务完整性。较低的隔离级别可提高并发能力，但同时也会导致出现更多的并发问题（例如脏读或丢失更新）；较高的隔离级别可以减少并发问题，但需要更多的系统资源，也会降低系统的并发能力。最高级别的隔离会导致性能低下，无法应对业务需求，因此也很少使用。

事务的隔离在实现的时候需要兼顾性能和应用层实际需求。针对四种可能出现的问题，定义了四种隔离级别：

|Isolation Level|隔离级别|脏读|更新丢失|不可重复读|幻读|
|---|---|||||
|Read Uncommitted|读未提交|<span style="color:red">可能发生</span>|<span style="color:red">可能发生</span>|<span style="color:red">可能发生</span>|<span style="color:red">可能发生</span>|
|Read Committed|读已提交|<span style="color:green">不会发生</span>|<span style="color:red">可能发生</span>|<span style="color:red">可能发生</span>|<span style="color:red">可能发生</span>|
|Repeatable Read|可重复读|<span style="color:green">不会发生</span>|<span style="color:green">不会发生</span>|<span style="color:green">不会发生</span>|<span style="color:red">可能发生</span>|
|Serializable|可串行化|<span style="color:green">不会发生</span>|<span style="color:green">不会发生</span>|<span style="color:green">不会发生</span>|<span style="color:green">不会发生</span>|

在四种隔离级别中, Serializable的级别最高, Read Uncommited级别最低。

* 大多数数据库的默认隔离级别为: Read Commited,如Sql Server, Oracle。
* 少数数据库默认的隔离级别为Repeatable Read, 如MySQL InnoDB存储引擎。

### 脏读
 
如下图所示，有 emails 和 mailboxes 两张表，emails 表中 unread_flag 字段表示是否已读，mailboxes 表中 unread 字段则是未读的email数量。

当用户1执行 insert 操作和 update 操作中间，用户2进行了读取，这时会获得一个矛盾的结果，明明有1封未读的 email，但是 unread 计数器却是0。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/51EB2AC5-758D-42CD-947E-6CDE8D1A7654_1_105_c.jpeg)

防止保证：读数据库的时候，只能看到已成功提交的数据。

### 更新丢失

如下图所示，用户1和用户2都分别尝试获取 counter，然后对 counter 进行加1并保存，理论结果应该是44，但是实际上结果为43。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/077E886C-0AE1-4A76-9B4F-3C7F69A5EB72_1_105_c.jpeg)

防止保证：通过原子化写操作、加锁等方式。
### 脏写

Alice 和 Bob 同时购买一辆车，买车同时也会更新发票，下图的场景中会出现， Bob 买到了车，但是 Alice 收到了发票的情况。

注意，这种情况和 **丢失更新** 不一样，丢失更新中第二次写入是在第一个事务提交后才执行的。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/F51A6AD6-9621-44BB-909F-23752F7CC9CD_1_105_c.jpeg)

防止保证：写数据库的时候，只能覆盖已经成功提交的数据。

### 不可重复读（读倾斜）

Alice有两个子账户，分别有500，一共有1000，转账者从账户2转了100到账户1，转账有两个SQL操作，两个操作在同一个事务中。如果 Alice 在同一个事务中读取了多次账户1，最初读取到了账户1是500，而后面会读取到600，两次读取数据不一致。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/2070BC05-6770-4BE7-BAA3-5C88761D03C3_1_105_c.jpeg)

防止保证：读数据库的时候，只能看到已成功提交的数据。

### 幻读（写倾斜）

下图中的表是一个 oncall 排班系统，如果有两个医生 oncall 的情况下，允许一个医生退出 oncall。Alice 和 Bob 分别读取了 oncall 的人数，判断当前 oncall 的人数多于两人，然后分别退出了 oncall，导致这个时间段无人 oncall。

![](https://zh-liang-cn.oss-cn-hangzhou.aliyuncs.com/images/76024979-5358-4E84-AE98-6CEB66AB4ECE_1_105_c.jpeg)

第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时 (此时第一事务还未提交) ，第二个事务向表中插入一行新数据。这时第一个事务再去读取表时,发现表中还有没有修改的数据行，就好象发生了幻觉一样。

防止保证：如果只有在事务完全提交后才能读取数据，则可以避免该问题。

## 事务隔离是如何实现的

不同的隔离级别采用不同的锁类开来实现。

### 可序列化的

有两种方式：

* 使用基于锁的并发控制实现，可串行性要求在事务结束时释放读写锁（在选定数据上获取）。此外，当SELECT查询使用带范围的WHERE子句时，必须获取范围锁，尤其是要避免幻像读取现象。
* 当使用基于非锁的并发控制时，不会获取任何锁。但是，如果系统在多个并发事务中检测到写冲突，则仅允许其中之一提交。这种方式称之为快照隔离。

### 重复读取

基于锁的并发控制实现将保持读写锁（在选定数据上获取），直到事务结束为止。但是，范围锁不受管理，因此可能发生幻像读取。

### 读已提交

基于锁的并发控制实现将写锁（在选定数据上获取）保持到事务结束，但是一旦执行SELECT操作，就会释放读锁（因此出现不可重复的读现象）。与重复读取一样，不管理范围锁。

### 阅读未提交

这是最低的隔离级别。在此级别，允许进行脏读取，因此一个事务可能会看到其他事务尚未提交的更改。

## 基于锁的并发控制是什么

锁分为两类：
1. 悲观锁：先取锁再访问
2. 乐观锁：多版本控制就是一种乐观锁实现

多版本控制详见[【MySQL】MVCC(多版本并发控制)](https://blog.csdn.net/u012512634/article/details/72453511)

## MySQL里面的事务控制

关闭自动提交(session级别), 每次执行单个逻辑工作单元后，手动`commit`提交/`rollback`回滚

```sql
set autocommit=0;
delete from users where id = 100;
delete from blogs where user_id = 100;
commit;
```
使用`begin`(或者`start transaction`)显示开启一个事务，然后使用`commit`提交，或者`rollback`回滚

```sql
begin;
delete from users where id = 100;
delete from blogs where user_id = 100;
commit;
```

## 参考

* [Wikipedia - Database transaction](https://en.wikipedia.org/wiki/Database_transaction)
* [Wikipedia - Isolation (database systems)](https://en.wikipedia.org/wiki/Isolation_(database_systems))
* [MySQL 8.0 Reference Manual / Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
* [MySQL 8.0 Reference Manual / InnoDB and the ACID Model](https://dev.mysql.com/doc/refman/8.0/en/mysql-acid.html)