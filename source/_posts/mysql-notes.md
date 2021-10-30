---
title: MySQL必知必会
date: 2021-10-31 00:49:38
tags: MySQL
excerpt: 这是累积的需要在日常工作中用到的MySQL语句和知识，包括MySQL用户和权限设置，表的创建、修改等操作，数据备份和还原等。
---

> 这是累积的需要在日常工作中用到的MySQL语句

## 查看数据库和表

查看所有的数据库、当前数据库中所有表，显示表的schema的语句
```sql
SHOW DATABASES;
USE <database>
SHOW TABLES;
SHOW COLUMNS FROM <tablename>;
```

为了和 Oracle 兼容，MySQL 提供了 `describe` 和 `desc` 语句，和 `show columns` 语句一样

```sql
DESCRIBE <tablename>;
DESC <tablename>;
```

显示创建表的语句

```sql
SHOW CREATE TABLE <tablename>;
```

## 创建表

指定存储引擎，虽然不指定ENGINE会使用默认的InnoDB，但是数据库高手都是指定ENGINE的，因为不同用途的表的性能会因使用引擎的不同而不同。

```sql
CREATE TABLE my_tbl() ENGIN=MyISAM;
CREATE TABLE my_tbl() ENGIN=InnoDB;
```

CREATE TABLE ... SELECT 语法

```sql
CREATE TABLE new_tbl AS SELECT * FROM orig_tbl;
```

创建约束
```sql
CREATE TABLE product (
     category INT NOT NULL, 
     id INT NOT NULL,
     price DECIMAL,
     PRIMARY KEY (category, id)
) ENGINE = INNODB;

CREATE TABLE customer (
     id INT NOT NULL,
     PRIMARY KEY (id)
) ENGINE = INNODB;

CREATE TABLE product_order (
     no INT NOT NULL AUTO_INCREMENT,
     product_category INT NOT NULL,
     product_id INT NOT NULL,
     customer_id INT NOT NULL,

     PRIMARY KEY (no),
     INDEX (product_category, product_id),
     INDEX (customer_id),

     FOREIGN KEY (product_category, product_id)
          REFERENCES product(category, id)
          ON UPDATE CASCADE ON DELETE RESTRICT,

     FOREIGN KEY (customer_id)
          REFERENCES customer(id)
) ENGINE = INNODB;
```

也可以这样写：

```sql
CREATE TABLE product_order (
     no INT NOT NULL AUTO_INCREMENT,
     product_category INT NOT NULL,
     product_id INT NOT NULL,
     customer_id INT NOT NULL,

     CONSTRAINT pk_product_order PRIMARY KEY (no),
     INDEX (product_category, product_id),
     INDEX (customer_id),

     CONSTRAINT fk_product_order_product FOREIGN KEY (product_category, product_id)
          REFERENCES product(category, id)
          ON UPDATE CASCADE ON DELETE RESTRICT,

     CONSTRAINT  fk_product_order_customer FOREIGN KEY (customer_id)
          REFERENCES customer(id)
) ENGINE = INNODB; 
```

注意下面两点：
1. 这里的`constraint`是可选的，后面的`constraint name`也是可选的。
2. 可以用于`primary key`, `foreign key`, `unique key`，即：

```sql
constraint pk_name primary key(id)
constraint fk_name foreign key (fk_id) references fk_table(id)
constraint uk_name unique [index|key] index_name index_type...
```

其他有用的语句:
```sql
TRUNCATE [TABLE] tbl_name;
```
这里TABLE是可选的。

```sql
LOAD DATA INFILE 'file_name' 
     [REPLACE | IGNORE]
     INTO TABLE tbl_name
```

## 修改表

修改表名:

```sql
alter table <old table name> rename <new table name>;
```

添加列:

```sql
alter table <table name> add column <new column name> varchar(10);
```

删除列:

```sql
alter table <table name> drop column <column name>;
```

修改列类型:


```sql
alter table <table name> modify <column name> char(10);
--或者
alter table <table name> change <column name> <column name> char(40);
```

修改列名称：

```sql
alter table <table name> change column <old column name> <new column name> varchar(30);
```

## 备份和还原数据库

```shell
mysqldump -h localhost -u root -p123456 www > data-backup.sql
```
注意：-p后面没有空格，www是数据库名字，该命令会备份整个数据库www到d盘

```shell
mysqldump -h localhost -u root -p123456 <database> <table 1> <table 2> > data-backup.sql
```
注意：备份<database>数据库的<table 1>和<table 2>表

```shell
mysqldump -h localhost -u root -p123456 –databases <database 1> <database 2> <database 3> > data-backup.sql
```

注意：备份三个数据库<database 1>，<database 2>，<database 3>中的所有表

```shell
mysqldump -h localhost -u root -p123456 <database> <table> --where="UPDATED_AT>'2013-01-01'" > data-backup.sql
```

注意：这里的`--where`可以对原来表中的数据进行筛选

其他参数：
* `--no-data` 只导出schema，不导出数据

还原数据库备份文件，常用`source`命令。进入mysql数据库控制台，然后使用`source`命令，后面参数为脚本文件（如这里用到的d:\wcnc_db.sql）

```
$ mysql -u root -p
mysql>use 数据库
mysql>source d:\wcnc_db.sql
```

也可以这样将details.sql文件中的数据导入到database数据库中:

```
$ mysql -h localhost -u root <database> < details.sql
```

## 修改MySQL密码

用这个命令：

```shell
mysqladmin -u root -p password
```
输入命令后会提示你输入当前root的密码，然后会让你输入新密码

注意：mysqladmin的命令格式是

mysqladmin [options] [command]

因此这个命令中的password是一个command，前面的"-u root -p"是options

如果当前没有root密码，需要给root账户添加密码，则：

```
mysqladmin -u root password
```

输入命令后，会提示你输入新的密码。

## MySQL用户和权限设置

创建一个数据库、一个用户，并且赋予用户控制这个数据库的权限

```sql
CREATE DATABASE <database name>;
CREATE user '<username>'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL privileges ON <database name>.* TO '<username>'@'localhost';
```

删除用户

```sql
drop user '<username>'@'localhost'
```

也可以直接删除mysql.user表中的数据：

```sql
delete from user where user='<username>' and host='<from host>';
```

不过两者是有区别的。

查看权限：

```sql
show grants for 'root'@'localhost'
show grants for 'root'@'%'
```

注意：'root'@'localhost'和'root'@'%'对于MySQL而言是两个不同的用户，前者只能从本机登录，后者可以从任何一台机器登录

赋予权限：

```sql
grant all privileges on *.* to '<username>'@'%' identified by '<password>';
grant select on <db_name>.<table_name> to '<username>'@'<from host>' identified by '<password>';

flush privileges;
```

执行了最后一句flush privileges后，权限才会生效。

不常见的权限：
1. GRANT_OPTION 执行grant语句的权限
2. REPLICATION_CLIENT 执行show master status、show slave status以及show binary logs的权限
3. REPLICATION_SLAVE 连接到master上的replication user必须具有的权限
4. SHUTDOWN 执行mysqladmin shutdown的权限
5. SUPER 拥有change master to, kill的权限，同时即使当前连接数超过了max_connections的限制，也可以连接到MySQL服务器

如果是只读帐号，最好设成：

```sql
grant select, show view on just_test.* to 'ro'@'localhost';
```

其中show view权限是让用户执行show create view(对于view来讲，也可以用show create table)，用一些客户端的时候，会有自动提示，这类软件会用show create view来读取视图结构，因此最好把这个权限也赋给只读用户。

其他修改密码的方法：

1. 通过修改MYSQL数据库中MYSQL库的USER表，就用普通的UPDATE、INSERT语句就可以

```sql
use mysql
update user set Password=password('newpassword') where User='root'; 
```

2. 使用SET PASSWORD语句，

```sql
SET PASSWORD FOR myuser@localhost = PASSWORD('mypasswd');
```

3. 使用GRANT ... IDENTIFIED BY语句

```sql
GRANT USAGE ON *.* TO myuser@localhost IDENTIFIED BY 'mypassword'; 
```

完了之后要执行：`flush privileges;`


## 对表进行分析

命令语法：

```sql
analyze table table_name;
```

analyze命令会更新表的统计信息

```sql
show table status [from <database name>]
```

输出为一个表格，因此我们可以用where语句来筛选，显示表名以INV开头的表的状态

```sql
show table status where name like 'INV_%'
```

注意：analyze table 命令会使用索引的信息来更新table的统计信息，对myisam表执行这个命令的时候会扫描整个索引，也就是说会锁住table一段时间。

参考：http://www.mysqlperformanceblog.com/2008/09/02/beware-of-running-analyze-in-production/

因此，看起来analyze table应该是一个非常快的命令，并且是无侵入性（non intrusive）的命令。

实际上不是，因为`analyze table`会等待之前所有的table的cache都flush到磁盘后才能进行统计，同时，`analyze table`本身也持有一个排他锁，详见：http://bugs.mysql.com/bug.php?id=33278

也就是说，如果当前正在在innodb table上执行一个很长的query，你的analyze table会阻塞在“waiting for table”锁上，直到这个query执行完成。

执行完analyze table后，可以查看索引的状态。

```sql
show index from <table name>
```

参考：http://www.penglixun.com/tech/database/mysql_show_index_cardinality.html

会显示如下字段：
```
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment |
```

Cardinality列的含义，查看官方文档的解释：

An estimate of the number of unique values in the index. This is updated by running ANALYZE TABLE or myisamchk -a. Cardinality is counted based on statistics stored as integers, so the value is not necessarily exact even for small tables. The higher the cardinality, the greater the chance that MySQL uses the index when doing joins.

所以这个Cardinality会有如下的含义：
1. 列值代表的是此列中存储的唯一值的个数（如果此列为primary key 则值为记录的行数）
2. 列值只是个估计值，并不准确。ANALYZE: MyISAM vs Innodb - MySQL Performance Blog 中可以看出来这个值有多么不靠谱
3. 列值不会自动更新，需要通过Analyze table来更新一张表或者mysqlcheck -Aa来进行更新整个数据库。
4. 列值的大小影响Join时是否选用这个Index的判断。
5. 创建Index时，MyISAM的表Cardinality的值为null，InnoDB的表Cardinality的值大概为行数。
6. MyISAM与InnoDB对于Cardinality的计算方式不同。 

## MySQL主从复制配置

详见：https://zh-liang-cn.github.io/2014/01/19/mysql-master-slave-replication/

## MySQL分区

先看示例：

```sql
-- create table
CREATE TABLE user_details(
     id INT NOT NULL AUTO_INCREMENT,
     user_id INT NOT NULL,
     first_name CHAR(50) NOT NULL,
     last_name CHAR(50) NOT NULL,
     birth_date DATE NOT NULL,
     age INT NOT NULL DEFAULT 0,
     PRIMARY KEY (id, birth_date),
     KEY full_name (`user_id`, `first_name`, `last_name`)
) 
PARTITION BY HASH(YEAR(birth_date))
PARTITIONS 4;
```

这里创建了一个表，按照birth_date的year进行分区，分成4个区，编号分别为p1, p2, p3, p4。

可以通过`explain`来查看数据在哪个分区上：

```sql
-- explain query
EXPLAIN PARTITIONS 
SELECT * FROM user_details
WHERE birth_date = '2014-09-11'
     AND user_id = 2
     AND first_name = 'test';
```

因为我们查询的YEAR(birth_date)为2014，除4后余2，因此所在数据所在分区为p2。

可以看出这个SQL用了索引，当分区字段做为条件传递的时候就会只在相应的分区内做查询了，可以有效减少扫描的索引结点以及数据行。

### 分区函数

Partition function有4种: 
1. RANGE
2. LIST
3. \[LINEAR\] HASH
4. \[LINEAR\] KEY

其返回值为整数，代表分区的编号，而且Partition function不能返回常数或者随机值

Partition function要求partition expression中可以包含一个或者多个列名，这种分区方式称为horizontal partitioning，也就是将不同的行分配到不同的物理分区上。

还有一种分区方式称为“垂直分区”，指将不同的列分配到不同的分区上，但是在MySQL 5.5中还没有实现

Mysql一个表的所有分区的storage engine必须相同，但是同一个database的不同表可以用不同的storage engine。

Mysql的merge, csv, federated引擎不支持分区。

特别注意：如果一个表没有主键，那么分区字段可以为任何字段；如果一个表有主键，那么分区字段必须是主键的一部分。也就是说下面这样是不行的，会报错：

```sql
CREATE TABLE table_with_primary_key (
     c1 INT PRIMARY KEY,
     c2 VARCHAR(50)
)
PARTITION BY RANGE COLUMNS (c2) (
PARTITION p0 VALUES LESS THAN ('b'),
PARTITION p1 VALUES LESS THAN ('c'),
PARTITION p2 VALUES LESS THAN MAXVALUE
); 
```

### RANGE分区类型

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN MAXVALUE
); 
```

VALUES LESS THAN 后面也可以接表达式，只需要能MySQL能计算出这个表达式的值，并且这个值和这个字段能比较就可以了

按年分区：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY RANGE ( YEAR(separated) ) (
    PARTITION p0 VALUES LESS THAN (1991),
    PARTITION p1 VALUES LESS THAN (1996),
    PARTITION p2 VALUES LESS THAN (2001),
    PARTITION p3 VALUES LESS THAN MAXVALUE
); 
```

按UNIX_TIMESTAMP分区：

```sql
CREATE TABLE quarterly_report_status (
    report_id INT NOT NULL,
    report_status VARCHAR(20) NOT NULL,
    report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)
PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
    PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
    PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
    PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
    PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
    PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
    PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
    PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
    PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
    PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
    PARTITION p9 VALUES LESS THAN (MAXVALUE)
); 
```

注意TimeStamp字段是不能用于分区的，Bug #42849

RANGE分区的好处：
1. 如果你想删除旧的数据，可以直接删除这个分区：alter table employees drop partition p0;，这比delete from要高效很多
2. 你需要一列包含日期或者时间数值，或者包含从其他字段计算出来的值
3. 你经常执行query中包含用于分区的列，比如EXPLAIN PARTITIONS SELECT COUNT(*) FROM employees WHERE separated BETWEEN '2000-01-01' AND '2000-12-31' GROUP BY store_id;

### LIST分区类型

和RANGE很像，RANGE里面的每个值必须是连续的，LIST则可以是离散的，当然，同样都只能是整数

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY LIST(store_id) (
    PARTITION pNorth VALUES IN (3,5,6,9,17),
    PARTITION pEast VALUES IN (1,2,10,11,19,20),
    PARTITION pWest VALUES IN (4,12,13,14,18),
    PARTITION pCentral VALUES IN (7,8,15,16)
); 
```

好处：删除旧数据ALTER TABLE employees TRUNCATE PARTITION pWest 

注意：ALTER TABLE XXX TRUNCATE PARTITION是清空分区中的数据，不删除分区，ALTER TABLE XXX DROP PARTITION会删除分区，当然可以通过ALTER TABLE XXX ADD PARTITION再加回来

和RANGE的一个区别：LIST没有那种catch all的分区，如果你添加一个MySQL无法分区的值，就会报错，可以用insert ignore

### COLUMNS分区类型

有两种：RANGE COLUMNS和LIST COLUMNS, MySQL 5.5.0才引入。

使用COLUMNS分区方式可以使用多个列进行分区，而且列的类型不仅仅是整数：
1. 整数类型：TINYINT, SMALLINT, MEDIUMINT, INT(INTGER), BIGINT，特别注意：FLOAT和DECIMAL不支持分区
2. 时间类型：DATE, DATETIME
3. 字符类型：CHAR, VARCHAR, BINARY, VARBINARY，特别注意：TEXT和BLOB不支持分区

### RANGE COLUMNS分区类型

不支持表达式，只支持列名；可以输入多个列，MySQL会执行基于元组（Tuple）的比较；不仅仅局限于使用整数型列

```sql
CREATE TABLE rcx (
    a INT,
    b INT,
    c CHAR(3),
    d INT
)
PARTITION BY RANGE COLUMNS(a,d,c) (
     PARTITION p0 VALUES LESS THAN (5,10,'ggg'),
     PARTITION p1 VALUES LESS THAN (10,20,'mmmm'),
     PARTITION p2 VALUES LESS THAN (15,30,'sss'),
     PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
); 
```

由于COLUMNS分区可以用来分区非整数型的字段，所以像下面这种做法也是可以的：

```sql
CREATE TABLE employees_by_lname (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE COLUMNS (lname)  (
    PARTITION p0 VALUES LESS THAN ('g'),
    PARTITION p1 VALUES LESS THAN ('m'),
    PARTITION p2 VALUES LESS THAN ('t'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
); 
```

### LIST COLUMNS

结合两个看就明白了

### HASH分区类型

之前的几种分区方式，都需要显示的指定行存放的分区的位置，但是HASH就不用了：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY HASH(store_id)
PARTITIONS 4; 
```

如果没指定PARTITIONS的值，默认为1，就是没分区

HASH分区函数接受任何可以返回整数值的表达式作为参数，但是因为每次插入都会执行这个语句，太复杂的就不行了；同时，如果修改了行导致计算表达式返回的值变化了，也会导致行从一个分区中移动到另一个分区，因此，最好是选择稳定的列或者稳定的表达式，比如YEAR(xxx)就比TO_DAYS(xxx)强

### LINEAR HASH分区类型

用了很复杂的算法，正态分布，好处在于添加、删除，合并、拆分分区很快，名字很容易误导人，LINEAR总是让然想起平均分

### KEY分区类型

Key分区和Hash分区是有区别的，Hash的分区函数可以看出来是用的整除，但是Key分区的分区函数是PASSWORD(),应该是散列，感觉Key和Hash的名字取反了吧，见http://blog.chinaunix.net/uid-20785090-id-4015547.html 。

用于KEY分区函数的字段：
1. 可以是0个或者多个，如果是0个，则使用主键作为分区，如果没有主键但是有UNIQUE KEY，则会使用UNIQUE KEY
2. 必须是表的主键的一部分
3. 不局限于整数类型

```sql
CREATE TABLE tm1 (
    s1 CHAR(32) PRIMARY KEY
)
PARTITION BY KEY(s1)
PARTITIONS 10; 
```

### LINEAR KEY分区类型

```sql
CREATE TABLE tk (
    col1 INT NOT NULL,
    col2 CHAR(5),
    col3 DATE
)
PARTITION BY LINEAR KEY (col1)
PARTITIONS 3; 
```

### 子分区

```sql
CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) )
    SUBPARTITIONS 2 (
        PARTITION p0 VALUES LESS THAN (1990),
        PARTITION p1 VALUES LESS THAN (2000),
        PARTITION p2 VALUES LESS THAN MAXVALUE
    ); 
```

对于NULL的处理，在LIST和RANGE分区函数中，将NULL的值当作比non-NULL值小的值，因此，对于下面的分区：

```sql
create table t1(
     c1 int,
     c2 varchar(20)
)
partition by range(c1) (
     partition p0 values less than (0),
     partition p1 values less than (10),
     partition p2 values less than MAXVALUE
);
```

插入insert into t1 values(null, 'liang'); 会到第一个分区。

对于LIST一般这么分：

```sql
create table t1(
     c1 int,
     c2 varchar(20)
)
partition by list(c1) (
     partition p0 values in (1,3,5,7),
     partition p1 values in (2,4,6,8),
     partition p2 values in (null)
);
```

如果后面没有这个null语句，插入null会报错：`ERROR 1504 (HY000): Table has no partition for value NULL`

对于HASH和KEY分区函数，如果字段的值为NULL或者表达式的值为NULL，则会当作0进行处理。

值得注意的是下面两语句都返回为NULL：

```sql
SELECT YEAR(NULL);
SELECT NULL MOD 4;
```

对于表：
```sql
create table th(
     c1 int,
     c2 varchar(20)
)
partition by hash(c1)
partitions 2;
```
如果插入：
```sql
insert into th(null, 'liang')
```
将会放到p0分区

### 限制

partition expression中：
1. 不能有存储过程、函数、UDF以及插件
2. 不能声明变量以及使用自定义变量
3. 可以有+ - *但是不能有 / 或者DIV，除了[LINEAR] KEY以外，只能返回整数值以及NULL
4. 不能有位运算符，比如| & ^ << >> ~
5. 不能有HEANLDER语句

分区创建好之后最好不改变SQL_MODE，可能出现数据丢失的情况

### Table locks

分区操作会获取写锁，不影响select，但是影响insert和update

最大的分区数（包括子分区）为1024，NDB引擎除外

对于已经分区的表，query cache不起作用了，MySQL 5.5对已经分区的表自动禁用query cache，Bug #53775

已经分区的InnoDB表，外键不再支持

详见：http://dev.mysql.com/doc/refman/5.7/en/partitioning.html

## 查看MySQL系统变量

```sql
select @@hostname;
select @@version;
select @@server_id;  
```

注意：一个master-slave集群中的每台机器的server_id必须不同 

查看MySQL的data目录

```sql
show variables like 'datadir' 
```

状态信息：
```sql
show global status;
show global status like 'Created_tmp_%'; 
show session status;
```

配置变量：
```
show global variables;
show global status like '%_table_size'; 
show session variables;
```

## MySQL TimeStamp类型

创建一个插入数据时自动更新的字段：

```sql
create table posts(
  id int auto_increment,
  title char(80) not null default '',
  post text not null default '',
  created_at timestamp default current_timestamp,
  updated_at datetime,
  primary key (id)
);
```

或者将`created_at`那行换成:

```
created_at timestamp default now(),
```

注意，MySQL5.6以前一个表是不能有两个自动更新时间的字段的，也就是说下面这个是会报错的：

```sql
create table posts(
  id int auto_increment,
  title char(80) not null default '',
  post text not null default '', 
  created_at timestamp default current_timestamp,
  updated_at datetime default now() on update now(), 
  primary key (id)
); 
```

不过MySQL5.6之后已经支持两个以上自动更新时间的字段了。

另外注意`updated_at`字段的：`on update now()`，当这一行被update后，这个时间戳会自动更新


## java.sql.SQLException: Value'0000-00-00'异常解决办法 

在使用MySql时, 数据库中的字段类型是timestamp的,默认为0000-00-00, 会发生异常:
```
java.sql.SQLException:   Value '0000-00-00' can not be represented as java.sql.Timestamp
```

解决办法:给jdbc url加上zeroDateTimeBehavior参数：
```
datasource.url=jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull&transformedBitIsBoolean=true
```

`zeroDateTimeBehavior=round`是为了指定MySql中的`DateTime`字段默认值查询时的处理方式；默认是抛出异常，对于值为0000-00-00 00:00:00（默认值）的纪录，如下两种配置，会返回不同的结果：
1. zeroDateTimeBehavior=round   0001-01-01   00:00:00.0
2. zeroDateTimeBehavior=convertToNull   null   

## MySQL临时表

对于某些query，MySQL会创建临时表来进行处理，临时表有两种：
1. 基于MEMORY存储引擎的临时内存表
2. 基于MyISAM存储引擎的临时磁盘表

当临时内存表的大小达到一定限制的时候，MySQL就会将临时内存表写入到磁盘，变为临时磁盘表。

这个限制由`tmp_table_size`和`max_heap_table_size`这两个变量中的最小值确定。

注意，用于使用`create table`创建的内存表和这里提到的临时内存表不一样，前者的大小仅仅只受`max_heap_table_size`来限制，而且永远不会出现超过大小限制会写入到磁盘。

有这么几种情况会出现临时内存表：
1. 包含UNION的query；
2. 使用了UNION或者聚合运算的视图，有个算法专门用于判定一个视图是不是会使用临时表：http://dev.mysql.com/doc/refman/5.0/en/view-algorithms.html
3. 如果query包含order by和group by子句，并且两个子句中的字段不一样；或者order by或者group by中包含除了第一张表中的字段；
4. query中同时包含distinct和order by
如果一个query的explain结果中的extra字段包含using temporary，说明这个query使用了临时表。

当MySQL处理query是创建了临时表（不论是内存临时表还是磁盘临时表），`created_tmp_tables`变量都会增加。如果创建了磁盘临时表，`created_tmp_disk_tables`会增加。

可以通过`show session status like 'Created_tmp_%';`来查看，如果要查看全局的临时表使用次数，将其中的session改为global即可。

一般磁盘临时表会比内存临时表慢，因此要尽可能避免出现磁盘临时表。下面几种情况下，MySQL会直接使用磁盘内存表，要尽可能避免：
1. 表中包含了BLOB和TEXT字段（MEMORY引擎不支持这两种字段）；
2. group by和distinct子句中的有超过512字节的字段；
3. UNION以及UNION ALL语句中，如果SELECT子句中包含了超过512（对于binary string是512字节，对于character是512个字符）的字段。


参考：
* http://www.thegeekstuff.com/2008/09/backup-and-restore-mysql-database-using-mysqldump/
* http://www.kbedell.com/2009/03/07/how-to-create-a-timestampdatetime-column-in-mysql-to-automatically-be-set-with-the-current-time/ 
* http://www.blogjava.net/hilor/articles/164814.html
