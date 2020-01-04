---
layout: post
title: MySQL删除表的时候忽略外键约束
excerpt: |+
  删除表不是特别常用，特别是对于存在外键关联的表，删除更得小心。但是在开发过程中，发现Schema设计的有问题而且要删除现有的数据库中所有的表来重新创建也是常有的事情；另外在测试的时候，也有需要重新创建数据库的所有表。当然很多自动化工具也可以做这样的事情。
date: '2013-12-18 08:10:11 +0800'
categories:
- MySQL
tags:
- MySQL
- Database
---

删除表不是特别常用，特别是对于存在外键关联的表，删除更得小心。但是在开发过程中，发现Schema设计的有问题而且要删除现有的数据库中所有的表来重新创建也是常有的事情；另外在测试的时候，也有需要重新创建数据库的所有表。当然很多自动化工具也可以做这样的事情。

删除表的时候有时会遇到这样的错误消息：

    ERROR 1217 (23000): Cannot delete or update a parent row: a foreign key constraint fails

这是因为你尝试删除的表中的字段被用作了其他表的外键，因此在删除这个表（父表）之前必须先删除具有外键的表（子表）。也就是说，删除表的过程需要和创建表的过程一致。

但是这往往不可接受，一方面如果表太多了，手动排序有点不可接受；另一方面，现在还没有自动的工具对进行排序（其实也不是不能实现）。因此，MySQL中提供了一个变量`FOREIGN_KEY_CHECKS`来设置是否在必要的时候检查外键约束。一般比较推荐这样做：

首先，自动生成所有的DROP语句，将其中的`MyDatabaseName`替换成你的数据库名称：

```sql
SELECT concat('DROP TABLE IF EXISTS ', table_name, ';')
FROM information_schema.tables
WHERE table_schema = 'MyDatabaseName';
{% endhighlight%}

然后，在生成的代码前后添加下面设置`FOREIGN_KEY_CHECKS`变量的语句：

```sql
SET FOREIGN_KEY_CHECKS = 0
-- DROP语句
SET FOREIGN_KEY_CHECKS = 1;
{% endhighlight%}

不过，要是忘记了最后一句也没太大关系，这个变量是基于Session的，也就是说，当你关闭了客户端，重新建立连接的时候，这个变量会恢复默认值。如果需要在全局范围内不检查外键约束（这种情况会比较少吧），可以这样做：

```sql
SET GLOBAL FOREIGN_KEY_CHECKS = 0;
{% endhighlight%}

或者

```sql
set @@global.FOREIGN_KEY_CHECKS = 0;
{% endhighlight%}

## 参考：

* <a href="http://stackoverflow.com/questions/8538636/mysql-set-foreign-key-checks">mysql SET FOREIGN_KEY_CHECKS</a>
* <a href="http://stackoverflow.com/questions/3476765/mysql-drop-all-tables-ignoring-foreign-keys">MySQL DROP all tables, ignoring foreign keys</a>

