---
layout: post
title: 学习数据库（一）
date: 2017-10-29 16:00:00 +0800
description: mysql事务隔离级别。悲观锁，乐观锁。
img: db.jpg
tags: [mysql]
---
随着工作年龄的增大，大家对于你技术的要求也在逐步提升，刚开始工作的时候，可能你知道个赠删该查。别人就会觉得你不错。过些年，你可能要知道所有的查询必须走索引，索引的最左原则。又过了些年大家都你的要求就是，你要知道mysql的架构，你要知道那些组件是能更换的，更换这些组件能带来什么优势。

目前我最想了解的是mysql的innodb的存储引擎。因为他支持ACID事务，支持行级锁定，这些特性在高并发的实时交易系统中很重要。

今天我们先来捋一捋事务隔离级别。
#四个特性
##原子性（Atomicity）
事务作为一个整体被执行，包含在其中的对数据库的操作，要么全部执行，要么全部不执行。这一点很重要，因为程序在执行过程中会产生各种各样的问题，如果数据库操作被执行了一半产生的脏数据对系统的影响不可预估的。
##一致性（Consistency）
事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束。我对一致性的理解为，数据库的数据的执行一定是按照程序的规则去完成，每个语句的执行结果是可预期的。比如a账户转账个b账户 100元，结果一定是账户减少100元，b账户增加100元。不会有第二个状态的可能。
##隔离性（Isolation）
多个事务并发执行时，一个事务的执行不应该影响其他事务的执行。是今天要讨论的主要话题。
##持久性（Durability）
已经提交的事务对数据库的修改应该永久保存在数据库中。

#隔离级别
##串行化（Serializable）
最高的隔离级别。锁表的方式。所有操作必须一个一个执行。可避免脏读，不可重复读，幻读。
##可重复读（Repeatable Read）
mysql的innodb存储引擎默认的隔离级别。使用mvcc（Multi-Version Concurrency Control ）多版本并发控制来提供高性能的可重复读操作。可避免脏读。不可重复读。
如果配合Next-Key Lock可在一定程度上避免幻读。
##读以提交（Read Committed）
可避免脏读的发生。是oracle和SQL Server的默认隔离级别。即同一个事务里两次查询的结果可能不一样。因为读取的数据可能被另一个事务以提交。这个在宏观上来对的，因为已经提交的数据是应该被查询出来的。但是对于一些特殊的情况不太适用，比如在一个事务里生成多个统计报表时。
begn work
insert into statistics1 select * from basics where ....
insert into statistics2 select * from basics where ....
commit
理论上生成的报表数据是基于同一个时间点的，也就是说多个报表数据是要完全对应的上的。而在读以提交的这个隔离级别，在两个语句之间有数据的变更，会导致 两个统计数据结果不一致。
##读未提交(Read Uncommitted)
这是最低级别的隔离级别，在一个事务中能读取到另一个未提交的数据，我没有想到什么场景能应用此隔离级别。他产生的现象就是脏读。

| 隔离界别 | 脏读 | 不可重复读 | 幻读 |
| - | - | - |
| Read Uncommitted | YES | YES | YES | 
| Read Committed | NO | YES | NO | 
| Repeatable Read | NO | NO | YES |
| Serializable | NO | NO | NO |

#实际操作
因为mysql innodb的默认隔离级别是Read Committed，所以接下来的实际操作都是在该隔离界别上操作，理解了该隔离级别的各种操作，其他的隔离级别自然就理解了。
{% highlight sql %}
create table order_info (
  id int PRIMARY KEY AUTO_INCREMENT,
  user_id int NOT NULL ,
  serial_number VARCHAR(32) NOT NULL ,
  status int NOT NULL,
  UNIQUE  INDEX  uniq_serial_number ( serial_number  ASC),
  INDEX idx_user_id (user_id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 ;

CREATE table pay_info (
   id int PRIMARY KEY AUTO_INCREMENT,
   order_id int  NOT NULL ,
   price int NOT NULL,
   status int NOT NULL,
   INDEX idx_order_id (order_id)
)ENGINE=InnoDB DEFAULT CHARSET=utf8 ;
{% endhighlight %}