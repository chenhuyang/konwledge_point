# MySQL——存储引擎
## 一、MySQL存储引擎MyISAM与InnoDB区别总结整理
### 1、MySQL默认存储引擎的变迁
* 在MySQL 5.5之前的版本中，默认的搜索引擎是MyISAM，从MySQL 5.5之后的版本中，默认的搜索引擎变更为InnoDB。
### 2、MyISAM与InnoDB存储引擎的主要特点
* MyISAM存储引擎的特点是：表级锁、不支持事务和支持全文索引，适合一些CMS内容管理系统作为后台数据库使用，但是使用大并发、重负荷生产系统上，表锁结构的特性就显得力不从心
* InnoDB存储引擎的特点是：行级锁、事务安全（ACID兼容）、支持外键、不支持FULLTEXT类型的索引(5.6.4以后版本开始支持FULLTEXT类型的索引)。InnoDB存储引擎提供了具有提交、回滚和崩溃恢复能力的事务安全存储引擎。InnoDB是为处理巨大量时拥有最大性能而设计的。它的CPU效率可能是任何其他基于磁盘的关系数据库引擎所不能匹敌的。
* 注意：
InnoDB表的行锁也不是绝对的，假如在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表(锁完以后，判断不符合条件的会逐步解锁)
例如update table set num=1 where name like “a%”。
两种类型最主要的差别就是InnoDB支持事务处理与外键和行级锁。而MyISAM不支持。所以MyISAM往往就容易被人认为只适合在小项目中使用。
### 3、MyISAM与InnoDB性能测试
* 随着CPU核数的增加，InnoDB的吞吐量反而越好，而MyISAM，其吞吐量几乎没有什么变化，显然，MyISAM的表锁定机制降低了读和写的吞吐量。
### 4、事务支持与否
#### 4.1 事务区别
* MyISAM是一种非事务性的引擎，使得MyISAM引擎的MySQL可以提供高速存储和检索，以及全文搜索能力，适合数据仓库等查询频繁的应用；
* InnoDB是事务安全的；事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而MyISAM就不可以了。 
#### 4.2 事务知识拓展(InnoDB)
##### 4.2.1 MySQL事务简介
* 主要用于处理操作量大，复杂度高的数据。比如说，在人员管理系统中，你删除一个人员，你即需要删除人员的基本资料，也要删除和该人员相关的信息，如信箱，文章等等，这样，这些数据库操作语句就构成一个事务！
* 在 MySQL 中只有使用了 Innodb 数据库引擎的数据库或表才支持事务。
* 事务处理可以用来维护数据库的完整性，保证成批的 SQL 语句要么全部执行，要么全部不执行。
* 事务用来管理 insert,update,delete 语句
##### 4.2.2 事务是必须满足4个条件（ACID）
* 原子性（Atomicity）：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
* 一致性(Consistency)：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。
* 隔离性(Isolation)：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
* 持久性(Durability)：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
##### 4.2.3 事务控制语句：
* BEGIN或START TRANSACTION；显式地开启一个事务；
* COMMIT；也可以使用COMMIT WORK，不过二者是等价的。COMMIT会提交事务，并使已对数据库进行的所有修改成为永久性的；
* ROLLBACK；有可以使用ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；
* SAVEPOINT identifier；SAVEPOINT允许在事务中创建一个保存点，一个事务中可以有多个SAVEPOINT；
* RELEASE SAVEPOINT identifier；删除一个事务的保存点，当没有指定的保存点时，执行该语句会抛出一个异常；
* ROLLBACK TO identifier；把事务回滚到标记点；
* SET TRANSACTION；用来设置事务的隔离级别。InnoDB存储引擎提供事务的隔离级别有READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ和SERIALIZABLE。
##### 4.2.4 MYSQL事务处理主要有两种方法：
###### 1.用 BEGIN, ROLLBACK, COMMIT来实现
* BEGIN 开始一个事务
* ROLLBACK 事务回滚
* COMMIT 事务确认
###### 2.直接用 SET 来改变 MySQL 的自动提交模式:
* SET AUTOCOMMIT=0 禁止自动提交
* SET AUTOCOMMIT=1 开启自动提交
##### 4.2.5 事务提交的操作

`mysql -u root -p` 

`mysql> show variables like "AUTOCOMMIT"` 显示 autocommit=ON 表示每次进行写数据都是自动提交

`mysql> set AUTOCOMMIT=1` 关闭自动提交事务的设置

`mysql> create database mydb;`

`mysql> use mydb;`

`mysql> create table t1(id int(4),name char(20),age int(4)) engine=innodb;`

`mysql> begin;` 开始一个事务

`mysql> insert into t1 values(01,'Apple',19);`  

`mysql> insert into t1 values(02,'Banana',20);`

`mysql> commit;` 事务确认

`msyql> select * from mydb.t1;` 查看数据,正常情况下是有两组数据

`mysql> begin;` 开始一个事务

`mysql> insert into t1 values(03,'Center',21);`  

`mysql> insert into t1 values(04,'Dog',22);`

`mysql> rollback;` 回滚事务

`mysql> select * from mydb.t1;` 仅有刚刚插入的两组数据

### 5、MyISAM与InnoDB构成上的区别
#### 5.1 每个MyISAM在磁盘上存储成三个文件：
* 第一个文件的名字以表的名字开始，扩展名指出文件类型，.frm文件存储表定义。 
* 第二个文件是数据文件，其扩展名为.MYD (MYData)。 
* 第三个文件是索引文件，其扩展名是.MYI (MYIndex)。
#### 5.2 基于磁盘的资源是InnoDB表空间数据文件和它的日志文件，InnoDB表的大小只受限于操作系统文件的大小，一般为 2GB。
### 6、MyISAM与InnoDB表锁和行锁的解释
#### 6.1 MySQL表级锁有两种模式：表共享读锁（Table Read Lock）和表独占写锁（Table Write Lock）。
* 对MyISAM表进行读操作时，它不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写操作。
* 对MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作。
* InnoDB行锁是通过给索引项加锁来实现的，即只有通过索引条件检索数据，InnoDB才使用行级锁，否则将使用表锁！行级锁在每次获取锁和释放锁的操作需要消耗比表锁更多的资源。在InnoDB两个事务发生死锁的时候，会计算出每个事务影响的行数，然后回滚行数少的那个事务。当锁定的场景中不涉及Innodb的时候，InnoDB是检测不到的。只能依靠锁定超时来解决。
### 7、是否保存数据库表中表的具体行数
* InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table 时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。
* 注意：当count(*)语句包含where条件时，两种表的操作是一样的。也就是 上述“6”中介绍到的InnoDB使用表锁的一种情况。
### 8、如何选择：
#### 8.1 MyISAM适合：
* 做很多count 的计算；
* 插入不频繁，查询非常频繁，如果执行大量的SELECT，MyISAM是更好的选择；
* 没有事务。
#### 8.2 InnoDB适合：
* 可靠性要求比较高，或者要求事务；
* 表更新和查询都相当的频繁，并且表锁定的机会比较大的情况指定数据引擎的创建；
* 如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表；
* DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的 删除；
* LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性（例如外键）的表不适用。
### 9、其他区别
* 对于AUTO_INCREMENT类型的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。
* DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。
* LOAD TABLE FROMMASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性(例如外键)的表不适用。
* 清空整个表时，InnoDB是一行一行的删除，效率非常慢。MyISAM则会重建表。
* 对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中可以和其他字段一起建立联合索引。
* InnoDB存储引擎被完全与MySQL服务器整合，InnoDB存储引擎为在主内存中缓存数据和索引而维持它自己的缓冲池。
* 要注意，创建每个表格的代码是相同的，除了最后的 TYPE参数，这一参数用来指定数据引擎。
