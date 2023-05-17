# void * malloc(unsigned int)
只申请空间，参数为空间大小（字节）。

## malloc分配空间过程
[malloc 底层实现及原理](https://www.cnblogs.com/ssezhangpeng/p/10808969.html#_labelTop)

malloc在虚拟空间中分配。通过brk和mmap两个系统调用实现。

brk是将数据段(.data)的最高地址的指针_edata向更高地址移动。
mmap为内存映射。

当分配空间小于M_MMAP_THRESHOLD时使用brk移动_edata指针。释放时只能从高地址开始释放。

大于M_MMAP_THRESHOLD时使用mmap，在堆栈中分配一段虚拟内存，可以直接释放。

大概返回低地址。

glibc实现的线程安全，每个线程都尽可能有自己的空间，同时会加锁和使用原子变量。

[深入理解 glibc malloc：内存分配器实现原理](https://zhuanlan.zhihu.com/p/443235305)

[glibc文档](https://sourceware.org/glibc/wiki/MallocInternals)

测试 4 * sizeof(void*)，记得在GNU/linux
```c++
#include<stdlib.h>
#include<iostream>
int main()
{
    std::cout<<sizeof(void*);//8
    std::cout<<sizeof(size_t);//8
    size_t * c = static_cast<size_t*>(malloc(sizeof(char)));
    c = c - 1;//b2b8
    size_t mask = ~0b111;

    int real_size = *c & mask;//32 == 4*8 == 4*sizeof(void*)
    bool a = *c & 0x04;//false
    bool m = *c & 0x02;//false
    bool p = *c & 0x01;//true fastbin一直为true

    size_t * d = static_cast<size_t*>(malloc(7*sizeof(void*)));
    d = d - 1;//b2d8 - b2b8 = 32
    real_size = *d & mask;//48 = 8*8 = 8 * sizeof(void*)
    a = *d & 0x04;//false
    m = *d & 0x02;//false
    p = *d & 0x01;//true 
    free(c+1);
    p = *c & 0x01;//true
    p = *d & 0x01;//true fastbin一直为true
    /*
        c-1  ---------> b2b8 size and AMP
        c malloc ret--> b2b9
                        chunk payload
                        prev_size
        d-1 ----------> b2d8 size and AMP
        d malloc ret--> b2d9
                        chunk payload
    */
    size_t * large_mmap = static_cast<size_t*>(malloc(1000000*sizeof(void*)));
    large_mmap = large_mmap-1;
    real_size = *large_mmap & mask;
    a = *large_mmap & 0x04;//false
    m = *large_mmap & 0x02;//true use mmap
    p = *large_mmap & 0x01;//false
    free(large_mmap+1);
    // p = *large_mmap & 0x01; segft mmap
}
```
# free(void*)
返还void*所指内存。

在glibc中：

1. 如果mmap分配的，mnumap
2. 根据情况放到chunks list中

# void * calloc(size_t n,size_t size)
申请n个大小为size（字节）的空间，并置零。

# void * realloc(void *,size_t)
扩容，realloc(nullptr)和realloc(0)分别处理（glibc的）。

在glibc中：

mmap情况：
1. 如果是mmap得到的内存，则mremap进行重新分配。
2. 如果系统不支持munmap同时大小要小与之前的大小，什么都不做。

brk情况：
1. 大小减少时，大小减小的够多，能分成两个块，返回前半部分，后半部分空闲。
2. 大小增加时，检查下一个块是否空闲，空闲就看下一个块大小够不够分，够分就合并，返回旧指针。
3. 大小增加时，下一个块不够或者无法使用，用空闲list中的块，并拷贝内容。


