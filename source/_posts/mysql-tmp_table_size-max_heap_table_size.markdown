---
layout: post
title: MySQL查询优化：tmp_table_size与max_heap_table_size
excerpt: |+
  对于某些query，MySQL会创建临时表来进行处理，临时表有两种：基于MEMORY存储引擎的临时内存表以及基于MyISAM存储引擎的临时磁盘表。当临时内存表的大小达到一定限制的时候，MySQL就会将临时内存表写入到磁盘，变为临时磁盘表。这个限制由tmp_table_size和max_heap_table_size这两个变量中的最小值确定。
date: '2014-04-18 10:02:33 +0800'
categories:
- MySQL
tags:
- MySQL
- Database
---
对于某些query，MySQL会创建临时表来进行处理，临时表有两种：

* 基于**MEMORY**存储引擎的临时**内存表**
* 基于**MyISAM**存储引擎的临时**磁盘表**

当临时**内存表**的大小达到一定限制的时候，MySQL就会将临时内存表写入到磁盘，变为临时**磁盘表**。

这个限制由`tmp_table_size`和`max_heap_table_size`这两个变量中的最小值确定。

> 注意，用于使用`create table`创建的内存表和这里提到的临时内存表不一样，前者的大小仅仅只受`max_heap_table_size`来限制，而且永远不会出现超过大小限制会写入到磁盘。

有这么几种情况会出现临时内存表：

* 包含`UNION`的query；
* 使用了`UNION`或者聚合运算的视图，有个算法专门用于判定一个视图是不是会使用临时表：[view-algorithms](http://dev.mysql.com/doc/refman/5.0/en/view-algorithms.html)；
* 如果query包含`order by`和`group by`子句，并且两个子句中的字段不一样；或者`order by`或者`group by`中包含除了第一张表中的字段；
* query中同时包含`distinct`和`order by`。

如果一个query的explain结果中的extra字段包含`using temporary`，说明这个query使用了临时表。

> 使用explain的时候extra里面如果输出`using temporary;`说明需要使用临时表，但是这里无论如何都看不出来是不是使用了磁盘临时表（因为是不是使用磁盘临时表要根据query结果的大小来判定，在SQL分析阶段是无法判定的），`using filesort`是说没办法使用索引进行排序（`order by`和`group by`都需要排序），只能对输出结果进行quicksort，参考[what-does-using-filesort-mean-in-mysql](http://www.mysqlperformanceblog.com/2009/03/05/what-does-using-filesort-mean-in-mysql/)

当MySQL处理query是创建了一个临时表（不论是内存临时表还是磁盘临时表），`created_tmp_tables`变量都会增加1。如果创建了一个磁盘临时表，`created_tmp_disk_tables`会增加1。
可以通过`show session status like 'Created_tmp_%';` 来查看，如果要查看全局的临时表使用次数，将其中的_session_改为_global_即可。
一般磁盘临时表会比内存临时表慢，因此要尽可能避免出现磁盘临时表。下面几种情况下，MySQL会直接使用磁盘内存表，要尽可能避免：

* 表中包含了`BLOB`和`TEXT`字段（MEMORY引擎不支持这两种字段）；
* `group by`和`distinct`子句中的有超过512字节的字段；
* `UNION`以及`UNION ALL`语句中，如果`SELECT`子句中包含了超过512（对于binary string是512字节，对于character是512个字符）的字段。

## 参考

* [http://www.mysqlperformanceblog.com/2007/01/19/tmp_table_size-and-max_heap_table_size/](http://www.mysqlperformanceblog.com/2007/01/19/tmp_table_size-and-max_heap_table_size/)
* [http://dev.mysql.com/doc/refman/5.0/en/view-algorithms.html](http://dev.mysql.com/doc/refman/5.0/en/view-algorithms.html)
* [http://www.mysqlperformanceblog.com/2009/03/05/what-does-using-filesort-mean-in-mysql/](http://www.mysqlperformanceblog.com/2009/03/05/what-does-using-filesort-mean-in-mysql/)
