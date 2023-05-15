# void * malloc(unsigned int)
只申请空间，参数为空间大小（字节）。

## malloc分配空间过程
[malloc 底层实现及原理](https://www.cnblogs.com/ssezhangpeng/p/10808969.html#_labelTop)

malloc在虚拟空间中分配。通过brk和mmap两个系统调用实现。

brk是将数据段(.data)的最高地址的指针_edata向更高地址移动。
mmap为内存映射。

当分配空间小于M_MMAP_THRESHOLD时使用brk移动_edata指针。释放时只能从高地址开始释放。

大于M_MMAP_THRESHOLD时使用mmap，在堆栈中分配一段虚拟内存，可以直接释放。

[深入理解 glibc malloc：内存分配器实现原理](https://zhuanlan.zhihu.com/p/443235305)

# void * calloc(size_t n,size_t size)
申请n个大小为size（字节）的空间，并置零。

# void * realloc(void *,size_t)
扩容