# new 关键字
## operator new 函数
1. 不是重载new
2. 是函数
3. 用于申请内存
## new 做了什么
1. 调用operator new申请内存
2. 在内存上初始化对象
3. 返回指针
# delete 关键字
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