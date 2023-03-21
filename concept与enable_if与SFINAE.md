# 1.Substitution failure is not an error(SFINAE)
参阅cppreference: [link](https://zh.cppreference.com/w/cpp/language/sfinae) 。

例1:
```
template<class T>
int f(typename T::B*);//failure
 
template<class T>
int f(T);
 
int i = f<int>(0); // 使用第二个重载
```
例2：
```
struct X {
  typedef int type;
};

struct Y {
  typedef int type2;
};

template <typename T> void foo(typename T::type);    // Foo0
template <typename T> void foo(typename T::type2);   // Foo1
template <typename T> void foo(T);                   // Foo2

void callFoo() {
   foo<X>(5);    // Foo0: Succeed, Foo1: Failed,  Foo2: Failed
   foo<Y>(10);   // Foo0: Failed,  Foo1: Succeed, Foo2: Failed
   foo<int>(15); // Foo0: Failed,  Foo1: Failed,  Foo2: Succeed
}
```
# 2. enable_if
使用enable_if禁止某些模板实例化：
```
using namespace std;

struct CanAdd{};

class A {
public:
    A(int i):i(i) {
    }
    A& operator+(const A& a)
    {
        i += a.i;
        return *this;
    }
    A& operator+(int a)
    {
        i += a;
        return *this;
    }
    operator int()
    {
        return i;
    }
public:
    int i;
    using ADD = CanAdd;
};
template<class T,
    class C = typename std::enable_if<std::is_same<CanAdd, typename T::ADD>::value>::type
>
T add(T && a, T && b) // #1
{
    return a + b;
}

template<class T,
    class C = typename std::enable_if<std::is_same<CanAdd, typename T::ADD>::value>::type
>
T add(T& a, T& b)  // #2
{
    return a + b;
}
template<class T,
    class C = typename std::enable_if<std::is_integral<T>::value>::type>
T add(T && a, T && b, C * = nullptr)    // #3
{
    return a + b;
}

template<class T>
T sum(T t)
{
    return t;
}
template<class T, class ... Args, 
    class C = typename std::enable_if<std::is_same<CanAdd, typename T::ADD>::value>::type>
T sum(T && first, Args && ...args)         
{
    return first + sum<T>(std::forward<Args>(args)...);     //#4
}
int main()
{
    A obj1(1);
    A obj2(2);
    // operator +
    obj1 + obj2; //OK
    obj1 + 1;    //OK
    1 + obj2;    //OK function obj => int


    add(1, 2);      //1+2 OK use #3
    add(A(1), A(2)); add(obj1, obj2); //obj+obj OK use #1 & #2
    auto ret = sum(A(1), A(2), A(3)); //ret.i == 6 use #4

    //add(1, A(1));   //1 + obj ERR
    //add(A(1), 1);   //obj + 1 ERR
    //auto ret_ = sum(1, 2, 3, 4); //1 + 2 + 3 + 4 ERR
}
```

# 3. concept (c++ 20)
没有人怀念enable_if

更新VS或编译器

知乎专栏介绍 [link](https://zhuanlan.zhihu.com/p/266086040) 。

上述例子改为如下形式
```
template <typename T>
concept class_add = std::is_same<CanAdd, typename T::ADD>;

template<class_add T>
T add(T && a, T && b) // #1
{
    return a + b;
}

// ↓也可以
template<class T>
requires class_add<T>
T add(T && a, T && b) // #1
{
    return a + b;
}

template<class_add T>
T add(T& a, T& b)  // #2
{
    return a + b;
}

template <typename T>
concept integral = std::is_integral_v<T>;

template<integral T>
T add(T a, T b)    // #3
{
    return a + b;
}

template<class_add T>
T sum(T t)
{
    return t;
}
template<class_add T, class_add ... Args>
T sum(T && first, Args && ...args)         
{
    return first + sum<T>(std::forward<Args>(args)...);     //#4
}

```