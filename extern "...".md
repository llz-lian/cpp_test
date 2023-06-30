# 语言链接
cppref:[语言链接](https://zh.cppreference.com/w/cpp/language/language_linkage)。
外部链接的函数和变量均有语言链接的特性。


# extern "C"
C++按C语言进行链接，或者用于C++调用C库

下面代码编译成一个库
```c++
extern "C"
{
    void foo()
    {

    }
}
```
在c++中调用需要extern "C"才能找到符号
```c++
extern "C"
{
    void foo();
}

int main()
{
    foo();
}
```
extern可以用于声明
```c++
extern "C" int a = 0;
extern "C" void foo() {}
```

# extern "C++"
下面代码编译成一个库
```c++
void foo()
{

}
```
下面代码链接加入上面的库
```c++
extern "C++"
{
    void foo();
}

int main()
{
    foo();
}
```