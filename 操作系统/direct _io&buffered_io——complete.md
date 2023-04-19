# buffered i/o
`read()`和`write()`读写通过内核维护的缓存（page cache）。

# direct i/o
## 为什么要有direct i/o？
用户层面可能自己维护一个文件cache，不需要操作系统再有一个page cache。
buffered i/o需要复制几次才能完成写操作，同时写的内容不一定持久化保存在设备上（需要等内核写入或者自己调用fsync同步）。direct i/o可以直接与块设备进行交互，但是文件元数据（文件大小，修改时间等）不会。
buffered i/o可能会预读文件，对于随意读写可能会频繁缺页，同时大文件也会频繁缺页。
## direct i/o优点
- 直接交互，减少内存拷贝
- 用户自己缓存文件数据比buffered i/o少一个缓存

## direct i/o缺点
- 需要buffer位置，buffer大小和文件offset对齐逻辑块大小，任何一个不对齐其出EINVAL错误。
- 还是要用户调用fsync来刷新数据。
- 没有预读

知乎链接：[Linux 文件 I/O 进化史](https://zhuanlan.zhihu.com/p/374626979)