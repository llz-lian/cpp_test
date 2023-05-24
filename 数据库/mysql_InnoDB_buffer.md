首先官方文档：[MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/innodb-in-memory-structures.html)

# A. Buffer Pool
使用一种LRU的变体实现，数据在内存中组织成链表结构，靠近头部的为最近被访问过的页(new sublist)，靠近尾部的为最近最少使用的(old sublist)，每次新缓存加入时插入到链表中间部位，淘汰链表尾部。
```
-----------------------
        head
        |
        |
        new sublist
        |
        |
        new sublist tail
插入==》mid
        old sublist head
        |
        |
        old sublist
        tail
------------------------
```

在默认情况下，new sublist占长度的八分之五，old sublist占剩下的八分之三。

可以通过配置更改buffer pool的大小，也可以将数据固定在buffer pool中,在重启时可以读取之前旧的buffer pool。

存在预读。

# B. Change Buffer
主要用于二级索引的缓存，同时二级索引的内容未缓存。当二级索引因为DML（insert,update,delete）操作需要修改时，change buffer可以通过合并操作来减少修改磁盘内容时的io，当其他操作需要修改的页时，可以读到内存中通过change buffer进行修改来得到正确结果。change buffer占用buffer pool的内存。

合并操作的时机：1. 一个页被读到buffer里；2. 超过上限；3. 崩溃恢复；4. 崩溃重启；5. slow server shutdown通过参数强制进行。

在系统空闲或者关机时，将内容写到磁盘。当change buffer中缓存了很多修改操作时，会拖慢磁盘查询。

含有主键或者降序列的二级索引不支持change buffer。

# C. Adaptive Hash Index
同样使用buffer pool的内存，通过在内存中建立hash索引，加快=和IN的查找速度，通常基于已有的B-tree索引构建，可以在索引键的任意长度上建立hash索引(LIKE %可以使用hash)。hash索引是根据查询自动建立的。

在某些负载情况下，hash索引会拖慢查询速度。

hash与B-tree的比较：[官方文档](https://dev.mysql.com/doc/refman/8.0/en/index-btree-hash.html)

# D. Log Buffer
存log数据，定期写回磁盘，默认大小16MB，存在大量插入删除行的过程时，增大log buffer大小减少磁盘i/o。

