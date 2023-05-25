# 在可重复读（REPEATABLE READ）时
读用快照。
写时加锁。
加锁时注意索引，如果where中存在索引，则只锁定对应行，如果不存在则用`间隙锁和下🗡锁`。

无索引情况：
```sql
drop table test;
create table test (id int);
insert into test values(0),(1),(2),(3),(4);
select * from test;

A:
-------------------
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update test set id = 0 where id = 3;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
-------------------

B:
-------------------
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update test set id = 100 where id = 4;
# wait lock
-------------------

A:
-------------------
commit;
-------------------

B:
-------------------
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
-------------------
```
有索引情况则不会阻塞。
select ... for share;可以获取新版本数据。
# 在读已经提交（READ COMMITTED）时
在写时通过快照判断是否满足where字句，只锁定对应行。

无阻塞情况：
```sql
drop table test;
create table test (id int);

insert into test values(0),(1),(2),(3),(4);
A:
-------------------
mysql> set transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)

mysql> set transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update test set id = 100 where id = 4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
------------------

B:
------------------
mysql> set transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.00 sec)

mysql> set transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update test set id = 5 where id = 3;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
------------------
```

存在阻塞情况，等待id=2处释放：
```sql
drop table test;
create table test (id int);
insert into test values(0),(1),(2),(3),(4);

A:
--------------------
mysql> set transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update test set id = 100 where id <= 2;
Query OK, 3 rows affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0
--------------------

B:
--------------------
mysql> set transaction isolation level READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> update test set id = 100 where id >=2;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
--------------------
```
# 读未提交（READ UNCOMMITTED）
读完全不用快照，只看最新版本。
写与读提交一样。

# 串行化（SERIALIZABLE）
写与可重复读类似。
禁用autocommit时，读全部隐式上共享锁。
