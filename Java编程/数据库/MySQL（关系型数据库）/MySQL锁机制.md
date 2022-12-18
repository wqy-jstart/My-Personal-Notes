# MySQL锁机制

## 什么是锁？

##### 锁是计算机协调多个进程或线程并发访问某一资源的机制（避免抢夺）。

在数据库中，除传统的计算资源（如CPU、RAM、IO等）的争用以外，数据也是一种供许多用户使用的资源。

如何保证数据库并发访问的一致性、有效性将是数据库必须解决的问题，锁冲突也是影响数据库并发访问性能的一个重要因素。这使得锁对数据库来说显得尤为重要和复杂。

#### 1.从数据操作的粒度划分

1）表锁：操作时，会锁定整个表

2）行锁：操作时，会锁定当前操作的行

#### 2.从对数据操作的类型划分

1）读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响

2）写锁（排它锁）：当前操作没有完成之前，会阻断其他写锁和读锁

> 各存储引擎对锁的支持

| 存储引擎 | 表级锁 | 行级锁 |
| -------- | ------ | ------ |
| MyISAM   | 支持   | 不支持 |
| InnoDB   | 支持   | 支持   |
| MeMory   | 支持   | 不支持 |
| BDB      | 支持   | 不支持 |

## 锁的特性

MySQL锁的特性大致归纳如下：

**表级锁**：偏向MyISAM存储引擎，开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度低。

**行级锁**：偏向InnoDB存储引擎，开销大，加锁慢，会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

> 可见，不能判断哪一种锁更好，只能就具体应用特点来判断哪一个锁更加合适！
>
> 表级锁更适合与查询为主，少量按索引更新数据
>
> 行级锁更适合有大量索引条件并发更新少量不同数据

## MyISAM表锁

MyISAM存储引擎只支持表锁

#### 如何添加表锁？

MyISAM存储引擎会在查询语句SELECT前，自动给涉及的所有表加读锁，在执行更新操作UPDATE、DELETE、INSERT前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此，用户一般不需直接用Lock Table 命令给MyISAM表显式加锁。

```mysql
加读锁： lock table table_name read；

加写锁： lock table table_name write；
```

#### 表锁特点

1）对MyISAM表的读操作，不会阻塞其他用户对同一表的读取请求，但会阻塞对同一表的写请求；

2）对MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作

> 简而言之，就是读锁会阻塞写，但是不会阻塞读。而写锁，则既会阻塞读，又会阻塞写

此外，MyISAM 的[读写锁](https://so.csdn.net/so/search?q=读写锁&spm=1001.2101.3001.7020)调度是写优先，这也是MyISAM不适合做写为主的表的存储引擎的原因。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。

```mysql
 
-- MySQL的锁机制
drop database if exists  mydb14_lock;
create database mydb14_lock ;
 
use mydb14_lock;
  
create table `tb_book` (
  `id` int(11) auto_increment,
  `name` varchar(50) default null,
  `publish_time` date default null,
  `status` char(1) default null,
  primary key (`id`)
) engine=myisam default charset=utf8 ;
 
insert into tb_book (id, name, publish_time, status) values(null,'java编程思想','2088-08-01','1');
insert into tb_book (id, name, publish_time, status) values(null,'solr编程思想','2088-08-08','0');
```

## InnoDB行锁

#### 行锁特点

偏向InnoDB 存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。

#### 行锁模式

InnoDB 实现了以下两种类型的行锁。

**共享锁（S）**：又称为读锁，简称S锁，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
**排他锁（X）**：又称为写锁，简称X锁，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排他锁，其他事务就不能再获取该行的其他锁，包括共享锁和排他锁，但是获取排他锁的事务是可以对数据就行读取和修改。

对于UPDATE、DELETE和INSERT语句，InnoDB会自动给涉及数据集加排它锁（X）；

对于普通的SELECT语句，InnoDB不会任何锁；

**可以通过以下语句显示给记录集加共享锁或排他锁 。**

```mysql
共享锁（S）：SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE 
排他锁（X) ：SELECT * FROM table_name WHERE ... FOR UPDATE
```

```mysql
-- 行锁 
drop table if exists test_innodb_lock;
create table test_innodb_lock(
    id int(11),
    name varchar(16),
    sex varchar(1)
)engine = innodb ;
 
insert into test_innodb_lock values(1,'100','1');
insert into test_innodb_lock values(3,'3','1');
insert into test_innodb_lock values(4,'400','0');
insert into test_innodb_lock values(5,'500','1');
insert into test_innodb_lock values(6,'600','0');
insert into test_innodb_lock values(7,'700','0');
insert into test_innodb_lock values(8,'800','1');
insert into test_innodb_lock values(9,'900','1');
insert into test_innodb_lock values(1,'200','0');
 
create index idx_test_innodb_lock_id on test_innodb_lock(id);
create index idx_test_innodb_lock_name on test_innodb_lock(name);
```

