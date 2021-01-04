# MySQL事务和隔离级别

## 事务及其特性
事务有四种重要特性：**原子性**（Atomicity）、**一致性**（Consistency）、**隔离性**（Isolation）、**持久性**（Durability）。
我们习惯称之为**ACID**特性。

### 原子性（Atomicity）
事务开始之后，所有的操作要么全部做完，要么全部不做。如果事务执行过程中发生异常，会回滚到事务开始前的状态，就像什么也没发生过一样。

例如：在事务中新增100条记录，但是在新增30条记录之后就出现异常了，那么数据库将对这30条新增的记录进行回滚。

### 一致性（Consistency）
事务执行前后数据都要是合法的，不会违背任何的数据完整性。

### 隔离性（Isolation）
每个事务都有自己的数据空间，所做的修改不能影响其他事务，但是隔离也是分级别的。

### 持久性（Durability）
事务一旦提交，其结果必须是永久性的，即使发生宕机，数据库也能恢复。

注：MySQL使用redo log来保证事务的持久性。

## 事务的隔离级别
事务的隔离级别从并发程度由高到低分为：**READ UNCOMMITTED**（读未提交）、**READ COMMITTED**（读已提交）、**REPEATABLE READ**（可重复读）、**SERIALIZABLE**（序列化）。

### READ UNCOMMITTED（读未提交）
此隔离级别下的事务会读取到其他事务中未提交的数据，也叫做**脏读**。

时序图如下：

![](https://wx3.sinaimg.cn/large/0081fa71gy1gmboahkn3qj30iw0bygm2.jpg)

具体例子：

1. 准备数据
```mysql
create database demo;
use demo;
create table user(id int primary key, name varchar(32));
insert into user(id, name) values(1, '小爱');
```

2. 终端1
```mysql
use demo;
SET @@session.transaction_isolation = 'READ-UNCOMMITTED';
begin;
update user set name = '豆豆' where id = 1;
select name from user where id = 1; -- 输出name是豆豆
```

3. 终端2
```mysql
use demo;
SET @@session.transaction_isolation = 'READ-UNCOMMITTED'; -- MySQL默认为可重复读，没有脏读。 
select name from user where id = 1; -- 输出name是豆豆（脏读，读了别人未提交的数据）
```

### READ COMMITTED（读已提交）
一个事务可以读取其他已提交事务的数据。

时序图如下：

![](https://wx3.sinaimg.cn/large/0081fa71ly1gmbppmnyjxj30jt0c2aaq.jpg)

具体例子：

1. 准备数据
```mysql
create database demo;
use demo;
create table user(id int primary key, name varchar(32));
insert into user(id, name) values(1, '小爱');
```

2. 终端1
```mysql
use demo;
SET @@session.transaction_isolation = 'READ-COMMITTED';
begin;
update user set name = '豆豆' where id = 1;
select name from user where id = 1; -- 输出name是豆豆
```

3. 终端2
```mysql
use demo;
SET @@session.transaction_isolation = 'READ-COMMITTED'; -- MySQL默认为可重复读。 
select name from user where id = 1; -- 输出name是小爱
```

4. 终端1
```mysql
commit;
```

5. 终端2
```mysql
select name from user where id = 1; -- 输出name是豆豆
```

### REPEATABLE READ（可重复读）
此隔离级别是MySQL默认的隔离级别。在同一个事务里，相同的select查询的结果是一样的，但是会有幻读。

时序图如下：

![](https://wx3.sinaimg.cn/large/0081fa71ly1gmbqdn6x64j30h60ewt9j.jpg)

具体例子：

1. 准备数据
```mysql
create database demo;
use demo;
create table user(id int primary key, name varchar(32));
insert into user(id, name) values(1, '小爱');
```

2. 终端1
```mysql
use demo;
SET @@session.transaction_isolation = 'REPEATABLE-READ';
begin;
select * from user where id = 2; -- 没有记录
```

3. 终端2
```mysql
use demo;
SET @@session.transaction_isolation = 'REPEATABLE-READ';
insert into user (id, name) value (2, '小花');
commit;
```

4. 终端1
```mysql
select * from user where id = 2; -- 没有记录
insert into user (id, name) value (2, '小花');-- 报错：主键重复（幻读导致）
```

### SERIALIZABLE（序列化/串行化）
此隔离级别是最高的，但并发程度却是最底的，所有的事务都是串行的。

时序图如下：

![](https://wx3.sinaimg.cn/large/0081fa71gy1gmbqxvaeqxj30il0dtwf0.jpg)

具体例子：

1. 准备数据
```mysql
create database demo;
use demo;
create table user(id int primary key, name varchar(32));
insert into user(id, name) values(1, '小爱');
```

2. 终端1
```mysql
use demo;
SET @@session.transaction_isolation = 'SERIALIZABLE';
begin;
insert into user(id, name) values(1, '小爱');
```

3. 终端2
```mysql
use demo;
SET @@session.transaction_isolation = 'SERIALIZABLE';
SELECT * FROM user; -- 会一直卡主，直到其他事务提交。
```

4. 终端1
```mysql
commit;
```

5. 终端2
```mysql
-- 出查询结果，或者超时
```

### 各种隔离级别总结
|  隔离级别   | 脏读  | 不可重复读 | 幻读
|  ----  | ----  | ----  | ----  |
| 读未提交 | ✔ | ✔ | ✔
| 读提交	| ✘ | ✔ | ✔
| 可重复读 | ✘ | ✘ | ✔
| 序列化	| ✘ | ✘ | ✘
