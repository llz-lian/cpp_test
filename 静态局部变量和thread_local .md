# 静态局部变量 与 thread_local
经过声明时才会初始化（零初始化和常量初始化为例外），之后跳过声明。抛出异常不认为初始化完成。程序退出时进行析构。

多个线程同时初始化这一变量，会保证只初始化一次。
```c++
#include<iostream>
#include<format>
struct A
{
    inline static int i = 0;
    A(){i++;std::cout<<std::format("i is {}.",i)<<std::endl;}
};
void foo()
{
    static A a;// or thread_local A a
}

int main()
{
    foo();
    foo();
    foo();
}
output:
i is 1.
```
# 初始化严格一次实现——双检查锁定模式
```c++
void foo()
{
    static A * a = nullptr;
    if(a == nullptr)//第一次检查
    {
        std::unique_lock<std::mutex> lock(m);
        if(a == nullptr)//第二次检查
        {
            a = new A;
        }
    }
}
```
其中第一次检查是为了性能考虑，如果没有第一检查会在每次调用时加锁，但完全没有必要（因为就初始化一次）。

第二次检查是正常流程，防止多次初始化（两个线程，同时通过第一次检查如果没有第二次检查会发生什么？）。


