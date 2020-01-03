---
layout: post
title: MySQL Master-Slave Replication / 主从复制
excerpt: |+
  通过MySQL主从复制（Master-Slave）来实现读写分离是一种常用的扩展方法：所有的写操作通过Master完成，所有的读操作通过Slave来完成，也可以设置一主多从，或者双主的方式来实现负载均衡，极大的增强了MySQL的可扩展性。我所在的Team也是通过MySQL的这种主从复制的结构来扩展MySQL的性能的，不过，我并没有参与搭建这个平台，本文是我在业余进行的自学结果（不管你有没有读出来，这里的言外之意就是按照我这里搞如果搞出问题来了，我是不负责的）。
date: '2014-01-19 18:11:31 +0800'
categories:
- MySQL
tags:
- MySQL
- Database
---

通过MySQL主从复制（Master-Slave）来实现读写分离是一种常用的扩展方法：所有的写操作通过Master完成，所有的读操作通过Slave来完成，也可以设置一主多从，或者双主的方式来实现负载均衡，极大的增强了MySQL的可扩展性。我所在的Team也是通过MySQL的这种主从复制的结构来扩展MySQL的性能的，不过，我并没有参与搭建这个平台，本文是我在业余进行的自学结果（不管你有没有读出来，这里的言外之意就是按照我这里搞如果搞出问题来了，我是不负责的）。

## MySQL主从复制的实现原理

先上张图，非常经典的图：

<a href="https://www.flickr.com/photos/zhlwish/14209516325/" title="Flickr 上 zhlwish 的 mysql-master-slave-replication"><img src="https://farm6.staticflickr.com/5274/14209516325_b71b337f96_o.png" width="507" height="320" alt="mysql-master-slave-replication"></a>

总结起来有这么4个步骤：

* Slave上面的_I/O thread_连接上Master，并请求从指定日志文件的指定位置(或者从最开始的日志)之后的日志内容;
* Master接收到来自Slave的_I/O thread_的请求后，通过负责复制的_I/O thread_根据请求信息读取指定日志指定位置之后的日志信息，返回给 Slave端的_I/O thread_。返回信息中除了日志所包含的信息之外，还包括本次返回的信息在Master端的Binary Log文件的名称以及在Binary Log（一般是mysql-bin.xxxxxx）中的位置;
* Slave的_I/O thread_接收到信息后，将接收到的日志内容依次写入到Slave端的Relay Log文件（一般是mysql-relay-bin.xxxxxx）的最末端，并将读取到的Master端的Binary Log的文件名和位置记录到`master.info`文件中，以便在下一次读取的时候能够清楚的告诉Master“我需要从某个Binary Log的哪个位置开始往后的日志内容，请发给我”；
* Slave 的_SQL thread_检测到Relay Log中新增加了内容后，会马上解析该Log文件中的内容成为在Master端真实执行时候的可执行的Query语句，并在自身执行这些Query。这样，实际上就是在Master端和Slave端执行了同样的Query，所以两端的数据是完全一样的。

## MySQL主从复制的搭建

下面的步骤参考了MySQL官方的<a href="http://dev.mysql.com/doc/refman/5.5/en/replication.html">MySQL 5.5 Reference Manual :: 16 Replication</a>，顺便吐槽一下,这个手册写的组织结构实在有点乱。

### 1. 配置MySQL Master

首先，打开Master的Binary Log功能，修改Master服务器的**my.cnf**文件:

    [mysqld]
    log-bin=mysql-bin
    server-id=1

有三点需要注意的是：

* `server-id`在整个Replication Group（即所有的Master和Slave服务器）里面必须是唯一的;
* 如果你使用了InnoDB，可以启用`innodb_flush_log_at_trx_commit=1`和`sync_binlog=1`，第一个参数指定每次事务提交后都需要将内存缓冲中的临时Log写入到硬盘上，第二个参数指定在事务提交后立即将内存缓冲中的临时Binary Log写入到硬盘上。通过指定这两个参数可以提高_Durability_和_Consistency_，不过却牺牲了性能，详见 <a href="http://wzmtony.blog.163.com/blog/static/20318015620130171834386/">MySQL的innodb_flush_log_at_trx_commit配置值的设定</a> 和 <a href="http://www.cnblogs.com/Cherie/p/3309503.html">Mysql配置参数sync_binlog说明</a>；
* `skip-networking`这一行必须注释掉，不然无法通过网络连接MySQL服务器。

本文前面提到，Slave的_I/O thread_会连接到Master上读取Binary Log，因此需要设定一个用户名和密码，原则上这里可以使用任何一个MySQL用户名。但是考虑到Slave会将这个用户名和密码用明文的方式保存在`data/master.info`文件中，不太安全。因此一般创建一个独立的账户，并且使这个账户只有`replication slave`的权限。

用MySQL客户端连接Master上，执行下面语句，在Master上创建一个账户，并赋予`replication slave`的权限：

```sql
create user 'replication_user'@'%.xxx.com' identified by 'replication_password';
grant replication slave on *.* to 'replication_user'@'%.xxx.com';
```

如果执行上面的语句的时候出现这样的错误：

    ERROR 1045 (28000): Access denied for user 'root'@'%' (using password: YES)

可能是你所使用的账户没有`grant option`权限。出于安全的考虑，有些MySQL服务器可能禁用了不是从_localhost_进行连接的`grant option`的权限，到MySQL服务器上，用`root`建立连接，并执行下面的语句

```sql
grant GRANT OPTION on *.* to 'root'@'%';
flush privileges;
```

然后重新在执行之前的`grant`语句。所有配置完成后，重启MySQL Master服务器。

### 2. 配置MySQL Slave

修改Slave服务器的**my.cnf**文件，只需要将`server-id`配置得和Master不一样即可:

    [mysqld]
    server-id=2

配置完成后重启MySQL Slave服务器。

注意，在Salve上可以不启用Binary Log，不过启用了有两方面用处：

* 作为备份，可以从Slave定期备份Binary Log，不用增加Master的负担；
* 可以将Slave配置成其他MySQL实例的Master，实现更复杂的拓扑结构（Topology）。

### 3. 复制数据

如果不能允许数据库宕机，或者数据量不算太大，可以用`mysqldump`完整备份整个数据库：

    mysqldump --all-databases --master-data > dbdump.sql

注意，这里如果指定了`--master-data`，在备份文件中会自动生成这样的语句:

```sql
CHANGE MASTER TO MASTER_LOG_FILE='mysqld-bin.xxxxxx', MASTER_LOG_POS=xxx;
```

如果不指定这个参数，就必须先执行下面语句来对整个数据库加读锁。对于InnoDB，这会阻塞所有的写操作。

```sql
flush table with read lock;
```

然后执行

```sql
show master status;
```

来查看并记录当前的Binary Log文件的名称和位置，然后在Slave上导入数据的时候（即本文的下一步）手动执行上面的`change master to ...`语句。

当然最后需要执行下面的语句来释放之前加在数据库上的读锁：

```sql
unlock tables;
```

### 4. 在Slave导入数据，启动Slave

连接到Slave上，执行下面的命令：

```sql
stop slave;
reset slave;
change master to
   master_host = 'your master host',
   master_user = 'replication_user',
   master_password = 'replication_password';
```

在控制台中执行下面命令向Slave中导入Master的数据：

    msyql -u root -p < dbdump.sql

完成后，用MySQL客户端连接到Slave上，执行下面命令启动Slave的I/O thread和SQL thread：

```sql
start slave;
```

完成后，可以通过下面这个命令来检查Slave的状态：

```sql
show slave status;
```

如果`Slave_IO_Running`和`Slave_SQL_Running`列的值都是`Yes`，就看起来正常了，这时你可以尝试在Master上创建一张表，然后在Slave上检查，你会发现这个表在Slave上也出现了。
我在配置的时候发现`Slave_SQL_Running = No`，而且`Last_Error`列出现了一个错误。后来发现是我在配置的时候没有在Slave上执行`stop slave; reset slave;`这两个语句导致的。当然也可以在启动Salve的时候指定`--skip-slave-start`参数。

> 2014-02-20更新：如果`Last_Error`列出现的错误是`Could not execute Delete_rows event on table` 或者`Could not execute Update_rows event on table`等，这表示Master上删除了一条在Slave上并不存在的行，这种情况一般不会发生，除非无关紧要的数据不一致。解决这种问题的方法是在Slave上执行：`STOP SLAVE; SET SQL_SLAVE_SKIP_COUNTER=1; START SLAVE;`，直接忽略这个错误。但是这种方法显然不好，请参考<a href="http://www.mysqlperformanceblog.com/2013/07/23/another-reason-why-sql_slave_skip_counter-is-bad-in-mysql/">Another reason why SQL_SLAVE_SKIP_COUNTER is bad in MySQL</a>。

## 离线复制数据

除了使用`mysqldump`命令来实现从Master拷贝数据到Slave上以外，还可以直接打包整个Master的`data`目录并复制Slave上来实现，详见 <a href="http://dev.mysql.com/doc/refman/5.5/en/replication-howto-rawdata.html">Creating a Data Snapshot Using Raw Data Files</a>。

## 参考

* <a href="http://dev.mysql.com/doc/refman/5.5/en/replication.html">MySQL 5.5 Reference Manual :: 16 Replication</a>
* <a href="http://tech.it168.com/a2009/0526/577/000000577294.shtml">MySQL Replication的实现原理</a>
* <a href="http://www.cnblogs.com/hustcat/archive/2009/12/19/1627525.html">理解MySQL--复制(Replication)</a>
* <a href="http://blog.csdn.net/wlzjsj/article/details/8032493">mysql主从复制搭建中几种log和pos详解</a>
* <a href="http://blog.longwin.com.tw/2008/03/mysql_replication_master_slave_set_2008/">MySQL 設定 Replication (Master - Slave)</a>