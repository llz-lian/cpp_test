总所周知模板是在编译器处理的，声明实现不能分离。
但是使用extern template可以让模板的声明实现分离。
```c++
// a.hpp
template<class>
void foo();

// a.cpp
#include"a.hpp"

template<class>
void foo() {}
template void foo<int>();

// main.cpp
#include"a.hpp"
extern template void foo<int>();
extern template void foo<char>();// OK just a dec
int main()
{
    foo<int>();// ok
    // foo<char>();// link err
    return 0;
}
```
不负责任猜测，实际上是这样的
```c++
// a.cpp
template<class>
void foo();//声明
template<class>
void foo() {}//函数模板具体实现，但是不会有符号
template void foo<int>();//让编译器生成符号

// main.cpp
template<class>
void foo();//声明
extern template void foo<int>(); // extern了一个函数，但是是模板的实例化
extern template void foo<char>();// 同样extern了一个函数
int main()
{
    foo<int>();// 链接时找到a.cpp中的
    // foo<char>();// 没有一个文件有
    return 0;
}
```
然后由于是模板，所以声明多少次都没事
```c++
// a.cpp
#include"a.hpp"
template<class>
void foo() {}//函数模板具体实现，但是不会有符号
template void foo<int>();//让编译器生成符号
template void foo<int>();//让编译器生成符号
template void foo<int>();//让编译器生成符号
template void foo<int>();//让编译器生成符号
template void foo<int>();//让编译器生成符号
```

