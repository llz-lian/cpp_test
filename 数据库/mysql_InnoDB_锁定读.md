# 锁定读（Locking Reads）
即使InnoDB在一些隔离级别存在MVCC机制，如果想要查询并修改某些行，但是这些行也会被其他事务修改（删除或更新）。
```sql
A:
----------
mysql> begin
    -> ;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test;
+------+
| id   |
+------+
|    0 |
|    1 |
|    2 |
|    3 |
|    4 |
+------+
5 rows in set (0.00 sec)

mysql> delete from test where id = 0;
Query OK, 1 row affected (0.00 sec)

mysql> commit;
Query OK, 0 rows affected (0.01 sec)
----------

B:
----------
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from test;
+------+
| id   |
+------+
|    0 |
|    1 |
|    2 |
|    3 |
|    4 |
+------+
5 rows in set (0.00 sec)

mysql> update test set id = 5 where id = 0;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 0  Changed: 0  Warnings: 0
----------
```

可以使用`select...for share`或者`select...for update`锁定行。


