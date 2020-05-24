## 1、Innodb支持的锁种类

### 1.1、[Shared and Exclusive Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-shared-exclusive-locks) 

```mysql
1、share locks也叫共享锁（s锁、读锁）
2、Excusive locks也叫排他锁（x锁，写锁）
```

### 1.2、[Intention Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-intention-locks)(意向锁)

```mysql
意向锁分为：
1、意向读锁：IS锁
2、意向写锁：IX锁
意向锁的引入，是因为Innodb同时支持表级锁和行级锁。
假设一个请求A执行"delete from table where ...",这个时候会加行级x锁，
这个时候另一个请求B执行"lock table for write"会申请加表级X锁，然后mysql需要去检查表中的每一行是否有加锁，没有加则申请成功，有就阻塞加锁请求。由于需要遍历每一行，效率会很低。所以引入了意向锁。
重新看下之前的例子,A请求对记录行加X锁时，先对表加IX锁，这个时候B请求过来请求加表级X锁，只需要检查该表是否有IS锁或者IX锁就可以了，不需要遍历每一行的锁情况。
具体数据的锁情况可以查看：preformance_schema.data_locks
```
在mysql 的锁情况中输出为：

```mysql
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

锁的兼容矩阵

|      | X    |  IX  | S    | IS   |
| -- | -- | -- | -- | -- |
| X    | 冲突 | 冲突 | 冲突 | 冲突 |
| IX   | 冲突 | 兼容 | 冲突 | 兼容 |
| S    | 冲突 | 冲突 | 兼容 | 兼容 |
| IS   | 冲突 | 兼容 | 兼容 | 兼容 |

### 1.3、[Record Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-record-locks)

```mysql
Record locks是对索引进行加锁，并不会对记录进行加锁。即使在表中没有任何索引，mysql也会默认生成一个聚簇索引（主键索引），然后对该索引加锁。
锁是锁索引，而不是锁记录的原因个人猜测是（不一定正确，欢迎指正）：
   除了唯一索引外，其他类型的索引跟记录的对应关系是1：N，比如"delete from table where col1="" and col2=""" 就是可以删除多条数据，如果是锁记录就需要多个锁，如果是锁索引就只需要一个锁。
```

在mysql 的锁情况中输出为：

```mysql
RECORD LOCKS space id 58 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` 
trx id 10078 lock_mode X locks rec but not gap
Record lock, heap no 2 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
 0: len 4; hex 8000000a; asc     ;;
 1: len 6; hex 00000000274f; asc     'O;;
 2: len 7; hex b60000019d0110; asc        ;;
```

### 1.4、[Gap Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-gap-locks)

```mysql
在数据库层看到的结果是这样的：
    RECORD LOCKS space id 281 page no 5 n bits 72 index idx_c of table `lc_3`.`a` trx id 133588125 lock_mode X locks gap before rec 
```

### 1.5、[Next-Key Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-next-key-locks)

```mysql
在数据库层看到的结果是这样的：
    RECORD LOCKS space id 281 page no 5 n bits 72 index idx_c of table `lc_3`.`a` trx id 133588125 lock_mode X
```

### 1.6、[Insert Intention Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-insert-intention-locks)

### 1.7、[AUTO-INC Locks](https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html#innodb-auto-inc-locks)

### 1.8、MVCC

## 2、备注

1、查看mysql事务级别

```
level: {
     REPEATABLE READ
   | READ COMMITTED
   | READ UNCOMMITTED
   | SERIALIZABLE
}
```
```mysql
1、查看当前会话隔离级别
select @@transaction_isolation;
2、查看全局隔离级别
select @@global.transaction_isolation;
```
2、修改mysql事务级别
```mysql
1、修改全局隔离级别
SET GLOBAL transaction_isolation='REPEATABLE-READ';
2、修改当前会话隔离级别
SET SESSION transaction_isolation='SERIALIZABLE';
```

- BEGIN 或 START TRANSACTION 显式地开启一个事务；
- COMMIT 也可以使用 COMMIT WORK，不过二者是等价的。COMMIT 会提交事务，并使已对数据库进行的所有修改成为永久性的；
- ROLLBACK 也可以使用 ROLLBACK WORK，不过二者是等价的。回滚会结束用户的事务，并撤销正在进行的所有未提交的修改；


2、查看一条SQL的加锁情况
```mysql
1、在使用之前需要打开innodb lock monitor，这样在查看 engine innodb status 的时候可以更加清晰的查到到锁的情况
2、set autocommit=0; 禁止自动提交，更好的查看事务持有锁的情况
3、show engine innodb status; 查看事务锁情况
```


## 参考链接：

[线上Mysql Delete 和 Insert 操作导致死锁问题分析](https://ketao1989.github.io/2014/10/09/Mysql-Delete-Insert-Deadlock-Analyse/)

[MySQL DELETE 删除语句加锁分析](http://www.fordba.com/lock-analyse-of-delete.html)

[[MySQL锁系列（一）之锁的种类和概念](https://keithlan.github.io/2017/06/05/innodb_locks_1/)]

