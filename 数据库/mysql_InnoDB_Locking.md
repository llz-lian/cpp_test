# 先放官方文档：[InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)
还有一个加锁分析博客存档[MySQL 加锁处理分析](https://www.cnblogs.com/tutar/p/5878651.html)
# InnoDB的几种锁
## 1. 共享锁和独占锁（shared & exclusive locks）
行级别的两种锁，共享锁（s）允许持有锁的过程对行进行`读取`，独占锁(x)允许对行进行`更新和删除`。

一个并发小例子：

两个同时进行的事务A和B，都想对行R进行操作。
1. 事务A得到了行R的`共享锁`，事务B可以获取行R的`共享锁`，但是无法获得`独占锁`。
2. 事务A得到了行R的`独占锁`，事务B无法获得行R的任何锁，只能等A释放独占锁。
## 2. 意向锁（Intention Locks）
表级别的锁，可以为共享或独占锁。
意向共享锁（intention shared lock, is）指示打算在表中行加共享锁。
意向独占锁（intention exclusive lock,ix）指示打算表中行加独占锁。
```sql
select ... for share  # 加共享锁
select ... for update # 加独占锁
```
在事务获取表中`行的共享锁`之前，必须获得`意向共享锁`或更强的锁。
在事务获取表中`行的互斥锁`之前，必须获得`意向独占锁`。

|想要获得的\已经加上的|x|ix|s|is
|:------------------|:-|:-|:-|:-|
|x|X|X|X|X
|ix|X|√|X|√
|s|X|X|X|√|√
|is|X|√|√|√

IX锁之间可能并不会存在重叠，所以是可以相容的。

如果一共过程获取锁与当前锁冲突，等待当前锁被释放。如果可能发生死锁，产生一个错误。

意向锁主要用于表明有事务需要加锁。

## 3. 记录锁（Record Locks）
满足某一条件的行锁。select c1 from t where c1 = 10 for update;，阻止其他事务修改c1=10的行。

记录锁通常锁定索引。

在读已提交级别上，在检查where字句后没有记录就会释放。

## 4. 间隙锁（Gap Locks）
锁某一个区间的行锁，区间不会改变。select c1 from t where c1 between 10 and 20 for update;，会阻止其他事务插入c1介于10到20之间的值。

unique的字段在单独查询时不会加间隙锁。如selec * from child where id = 100;如果id为unique的没有必要加锁。

允许不同事务持有冲突的间隙锁，如果需要进行修改，必须合并不同事务的间隙锁。

在读已提交级别上可以禁用间隙锁。

## 5. ~~下贱锁~~（Next-Key Locks）
索引序之前（包含当前查的）的加锁。
假设索引包含以下值：10，11，13，20。则有一下几种锁
```
(-∞,10]
(10,11]
(11,13]
(13,20]
(20,+∞)
```
在InnoDB默认隔离级别（可重复读）下，使用此锁进行搜索和索引扫描，防止幻读。

## 6. 插入意向锁（Insert Intention Locks）
insert操作在插入之前设置的间隙锁，但是只要主键不冲突就不互斥。
```sql
create table test
(
    id int primary key
);
A:
----------------
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into test values(0),(1),(3);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0
----------------

B:
----------------
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> insert into test values(2);
Query OK, 1 row affected (0.00 sec)

mysql> insert into test values (0);
# wait lock
----------------

A:
----------------
mysql> commit;
Query OK, 0 rows affected (0.00 sec)
----------------
B:
----------------
ERROR 1062 (23000): Duplicate entry '0' for key 'test.PRIMARY'
----------------
```
# 7. 自增锁（AUTO-INC Locks）
`AUTO_INCREMENT`列的特别锁。向有AUTO_INCREMENT的表插入数据时，会先获取自增锁。

# 8. 空间索引的谓词锁（Predicate Locks for Spatial Indexes）

