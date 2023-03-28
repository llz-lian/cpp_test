# const
1. 修饰变量表示只读,修饰指针时有指针常量与常量指针。
```
const int a = 1;
void foo (const int a);

const int * p = a;
int const * p = a;
const int * const p = a;
```
2. 修饰方法
```
class A
{
    const int foo()const
    {
        return 1;
    }
    void bar()
    {
        return;
    }
}
const A a;
a.foo(); //OK
a.bar(); //ERR
```
# constexpr
表示常量
知乎链接[link](https://www.zhihu.com/question/35614219)。
```
template<int N> class C{};

constexpr int FivePlus(int x) { return 5 + x; }

void f(const int x) {
    C<x> c1; // Error: x is not compile-time evaluable.
    C<FivePlus(6)> c2; // OK
}
```