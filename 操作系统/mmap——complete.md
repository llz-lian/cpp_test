# 传统读写
传统读写需要通过`read()`和`write()`两个系统调用实现。
首先需要调用`read()`从数据从设备读取到内核的缓冲区，再把内核缓冲区数据复制到用户缓冲区，再通过`write()`把数据传到内核缓冲区，最后写回设备。
实际上用户并没有直接与文件进行交互，而是与一个缓存进行交互来完成读写操作。
很明显上述过程需要四次拷贝过程，而且需要两次系统调用，也就是4次状态切换。
同时用户还要自己申请空间。
# mmap读写
很明显，如果用户可以直接读写缓存，那么就可以省下拷贝的消耗。
而mmap可以将用户空间的虚拟内存地址映射至缓冲区处。而缓冲区对应到文件上。
## 在mmap后何时将缓冲区同步至文件处
- 调用msync进行同步
- 使用munmap解除映射
- 系统关机
- 进程结束
## mmap缺点
- 使用时必须指定大小
- 读写小文件并不是优势，mmap调用开销大于read。
- 写文件时照样有io，大量写时不一定比传统方法快

## mmap读写例子
```
int main()
{
    // mmap.txt: 123456
    int fd = open("mmap.txt",O_RDWR);
    if(fd<0) exit(1);
    void * addr = mmap(nullptr,4096,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    if(addr==MAP_FAILED) exit(1);
    char * start = (char *)(addr);
    // output: 123456
    std::cout<<start<<" "<<std::endl;
    // change 123456 ==> abcdef
    const char * buffer= "abcdefghhhhhhh";
    memcpy(addr,buffer,strlen(buffer));
    // output: abcdefghhhhhhh
    std::cout<<start<<" "<<std::endl;
    // file: abcdef
}
```