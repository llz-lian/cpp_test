# new 关键字
## operator new 函数
1. 不是重载new
2. 是函数
3. 用于申请内存
## new 做了什么
1. 调用operator new申请内存
2. 在内存上初始化对象
3. 返回指针
## operator new 细节
在GNU libc++中默认用malloc进行内存分配。
在编译时使用{`--enable-libstdcxx-static-eh-pool`}可以让异常的分配使用一个静态的缓冲区。


茴的n种写法

c++ 98里
1. void* operator new(std::size_t);
2. void* operator new(std::size_t, std::nothrow_t) noexcept;无异常版本，出问题返回空指针。new (std::nothrow) 
3. void* operator new[] (std::size_t);
4. void* operator new[](std::size_t, std::nothrow_t) noexcept;
5. void* operator new(std::size_t, void*) noexcept; placement new(布置 new)
6. void* operator new[](std::size_t, void*) noexcept;
类内还可以继续重载每个版本
7. void * CLASS::operator new(xxx)xxx;
调用时根据重载决定使用哪个版本
调用nothrow版本:
```
int * i = new(std::nothrow) int[100000000000];// void* operator new(std::size_t, std::nothrow_t) noexcept;
// 你最好检查i==nullptr
```



普通new内部做如下工作:

首先尝试分配内存，如果分配失败，尝试调用注册的new-handler，再不行就抛bad_alloc异常。
```c++
while (true)
{
  if (void* p = /* try to allocate memory */)
    return p;
  else if (std::new_handler h = std::get_new_handler ())
    h ();
  else
    throw bad_alloc{};
}
```
注册方法如下
```c++
typedef void (*PFV)();
static char*  safety;
static PFV    old_handler;
void my_new_handler ()
{
    delete[] safety;
    safety = nullptr;
    popup_window ("Dude, you are running low on heapmemory.  You"	     " should, like, close some windows, or something."	     " The next time you run out, we're gonna burn!");
    set_new_handler (old_handler);
    return;
}
int main ()
{
    safety = new char[500000];
    old_handler = set_new_handler (&my_new_handler);
    ...
}
```


# delete 关键字
完全可以delete空指针，标准说的。
## operator delete函数
1. 不是重载delete
2. 是函数
3. 释放内存
## delete 做了什么
1. 调用指针指向对象的析构函数
2. 调用operator delete释放内存

# new[]和delete[]
通常new[]与delete[]配套使用
```
string * a = new string[4];
delete[]a;
```
上述代码中new[]分配足够空间并调用4次string构造函数，delete[]调用4次析构函数并释放申请的空间。
## delete[]如何知道释放空间大小？
在一般情况下，new[]在申请空间时会在空间头部多申请4个字节用于存储数组长度，delete[]就可以通过这4个字节知道数组大小。
## 如果不配对使用会如何？
情况1：对象A new时使用new,释放时使用delete[]
```
class A
{
public:
    A() 
    {
        std::cout << "construct" << std::endl;
    };
    ~A() 
    {
        std::cout << "deconstruct" << std::endl;
    };
};
int main()
{
    A* a = new A();
    delete[]a;
    return 0;
}

output:
construct
deconstruct
deconstruct
deconstruct
deconstruct
...
```
由于a-4地址未分配，调用析构函数次数位置，释放位置为a-4

情况2：对象A new时使用new[],释放时使用delete
```
int main()
{
    A* a = new A[2];
    delete a;
    return 0;
}
output:
construct
construct
deconstruct
//error
```
只调用一次析构函数，未从a-4处开始释放，段错误

有些情况即使不配套也会正常工作，内置类型，无析构函数结构，可以不配套使用

情况3:
```
int main()
{
    int* a = new int;
    delete[]a;
    return 0;
}
output:
无事发生
```
情况4：
```
int main()
{
    int* a = new int[4];
    delete a;
    return 0;
}
output:
无事发生
```
情况5：
```
struct B
{
    int a = 4;
    int b = 2;
    float c = 0.1;
    double d = 1000;
    bool e = true;
};
int main()
{
    B* b = new B;
    delete[]b;
    return 0;
}
output:
无事发生
```