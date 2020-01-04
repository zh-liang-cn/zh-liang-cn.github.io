---
layout: post
title: 用MySQL Procedure同时像级联表插入数据
excerpt: 一个月以来都没写博客，这篇博客的目的是提醒自己继续写下去，顺便记下笔记，看官若无兴趣请直接飘过。级联表是关系数据库存储领域模型（Domain Model）中一对多关系的不二法门，比如“学生”和“班级”，实在是常用得很。创建表时建立外键关联，查询时使用inner join或者多表联合查询非常便捷。不过插入数据则相对麻烦，因为关键关联的缘故，需要先插入主表，然后再插入从表，如果使用auto_increment主键，在插入从表之前必须获取刚刚插入主表时生成的ID。
date: '2012-04-17 23:46:43 +0800'
categories:
- MySQL
tags:
- Database
- MySQL
---

> 一个月以来都没写博客，这篇博客的目的是提醒自己继续写下去，顺便记下笔记，看官若无兴趣请直接飘过。

级联表是关系数据库存储领域模型（Domain Model）中一对多关系的不二法门，比如“学生”和“班级”，实在是常用得很。创建表时建立外键关联，查询时使用inner join或者多表联合查询非常便捷。不过插入数据则相对麻烦，因为关键关联的缘故，需要先插入主表，然后再插入从表，如果使用auto_increment主键，在插入从表之前必须获取刚刚插入主表时生成的ID。

举例来说，下面classes和students表通过外键class_id建立一对多关联：

``` sql
drop table if exists students;
drop table if exists classes;
drop view if exists student_in_class;
create table classes(
  id serial,
  name char(55) not null,
  unique key cls_name (name),
  primary key(id)
);
create table students(
  number char(11) not null,
  name varchar(55) not null,
  class_id bigint unsigned not null,
  primary key(number),
  foreign key (class_id) references classes(id)
);
```

为了查询数据方便，创建一个视图，只是简单的执行级联查询：

```sql
create view student_in_class as
select number, students.name stu_name, classes.name cls_name
from students
inner join classes
on classes.id=students.class_id;
```

对于之前提出的两个表同时插入数据的问题，熟悉MySQL的朋友都知道，用下面的方法就行了，先插入主表classes，然后通过_last_insert_id()_获取刚刚插入的id，最后向从表students插入数据。

```sql
start transaction;
insert into classes(name) value('Class 1');
insert into students(number, name, class_id)
  values('001', 'Jim', last_insert_id());
commit;
```

不过这样还是有个问题，如果待插入的数据和主表中已有的数据有重复怎么办呢？因此笔者对以上语句进行简单的封装，使用MySQL存储过程实现整个过程，首先对主表进行查询，如果不存在待插入的数据再插入：

```sql
drop procedure if exists insert_stu;
create procedure insert_stu(
  cls_name char(55),
  stu_num char(11),
  stu_name varchar(55))
begin
  declare cls_id bigint unsigned;
  declare cls_cnt int;
  select count(*) into cls_cnt from classes where name=cls_name;
  if cls_cnt = 0 then
    insert into classes(name) value(cls_name);
    set cls_id = last_insert_id();
  else
    select id into cls_id from classes where name=cls_name;
  end if;
  insert into students(number, name, class_id)
    values(stu_num, stu_name, cls_id);
end;
```

调用和检验该存储过程的方法如下：

```sql
call insert_stu('Class 1', '001', 'Bob');
call insert_stu('Class 2', '002', 'Jim');
call insert_stu('Class 1', '003', 'Li Lei');
select * from student_in_class;
```

最后一句是使用之前创建的视图查看输出结果：

    number  stu_name  cls_name
    001        Bob          Class 1
    002        Jim           Class 2
    003        Li Lei        Class 1

## 参考

* <a href="http://stackoverflow.com/questions/175066/sql-server-is-it-possible-to-insert-into-two-tables-at-the-same-time">Stackoverflow: Is it possible to insert into two tables at the same time?</a>
* <a href="http://dev.mysql.com/doc/refman/5.0/en/create-procedure.html">http://dev.mysql.com/doc/refman/5.0/en/create-procedure.html</a>
* <a href="http://dev.mysql.com/doc/refman/5.5/en/commit.html">MySQL transaction commit</a>
* <a href="http://dev.mysql.com/doc/refman/5.0/en/information-functions.html#function_last-insert-id">MySQL last_insert_id() function</a>
* <a href="http://dev.mysql.com/doc/refman/5.5/en/if-statement.html">MySQL If statement</a>

