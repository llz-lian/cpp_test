# lock和unlock
均为原子操作
# std::mutex
非重入互斥锁。
1. lock或者try_lock获得锁，unlock释放锁。
2. 一般不单独使用，配合lock_guard和unique_lock使用。
3. 是标准布局的。
4. 不可复制与移动
# std::timed_mutex
支持获取锁时限时尝试。
# std::recursive_mutex
可重入，同一个线程获得锁的互斥时，可以继续成功lock，不可以指定最大多少次重复上锁，到达上限时lock抛出**异常**，try_lock返回false。
# std::recursive_timed_mutex
std::timed_mutex的重入版本
# std::shared_mutex
可以共享锁定，也可以排他锁定（共享、独占）。
如果一个线程获取独占锁，其他线程无法获取锁。
没有线程获取独占锁时，共享锁能被多个线程获取。
同一个线程只能获取共享或独占的某**一个**锁。
# std::shared_timed_mutex
std::timed_mutex的共享版本
# std::lock_guard & std::unique_lock
创建时尝试获得互斥锁所有权。

std::unique_lock: mutex的包装器，可移动但不可复制，支持限时尝试。

std::lock_guard: 就是RAII包装器，不可复制
# std::shared_lock
共享所有权包装器。
# std::scoped_lock(c++17)
RAII包装器，可以对多个锁进行所有权获取，通过算法避免死锁。
# std::lock
与std::scoped_lock类似，可以同时获取多个锁但不用担心死锁。
但是调用比std::scoped_lock麻烦。
```c++
mutex m1,m2;
// std::lock:
std::lock(m1,m2);
std::lock_guard<std::mutes> l1(m1,std::adopt_lock);
std::lock_guard<std::mutes> l2(m2,std::adopt_lock);
// c++ 17
// std::scoped_lock lk(e1.m, e2.m);
```
