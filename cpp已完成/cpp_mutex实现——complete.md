# c++标准不管这些
# GNU libstdc
## std::mutex，std::timed_mutex
使用pthread_mutex_t实现。
### pthread_mutex_t实现
1. CAS指令
```
    if(value == target)
    {
        target = value 
        return 1;
    }
    return 0;
```
2. 一个结构体表示mutex状态（是否占用（__lock），可重入次数，持有锁的线程ID，mutex类型，等待链表）。
3. CAS操作判断__lock是否等于0，如果是0则置1。
4. 没有获取到锁，根据情况挂起。

# 软件实现——皮特森算法
注意乱序执行，加屏障。
