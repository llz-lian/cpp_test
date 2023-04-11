# explict
禁用隐式类型转换，该类型的构造函数必须显式调用。
```
class A
{
public:
    int i = 0;
    int j = 0;
    A(int i) :i(i) {};
    A(int i, int j) :i(i), j(j) {};
};
class A_explict
{
public:
    int i = 0;
    int j = 0;
    explicit A_explict(int i) :i(i) {};
    A_explict(int i, int j) :i(i), j(j) {};
    explicit A_explict(int i, int j,int k) :i(i), j(j) {};
};

int main() {
    
    A a = 1;// ok call A(int)
    A_explict ae = 1; //error must use A_explict(1)
    A_explict b = A_explict(1);// OK
    A_explict c = { 1,2 };// OK
    A_explict d = { 1,2,3 };//error must use A_explict(1,2,3)
    return 0;
}
```
# dynamic_cast 
只能用在完整类型的指针和引用上，或将一个完整类型转换为void*，允许派生类向基类转换，也允许基类向派生类转换。
```
class Base{
    virtual void foo() {};
};
class BaseOnBase :public Base{};

class A {};

int main() {
    A* a = new A();
    A* b = dynamic_cast<A*>(a);// A==>A ok
    void* a_void = dynamic_cast<void*>(a);// A==>void error

    Base* publicbase = new BaseOnBase;
    Base* base = new Base;
    BaseOnBase * p_bb;

    p_bb = dynamic_cast<BaseOnBase*>(publicbase); // BaseOnBase==>BaseOnBase
    p_bb = dynamic_cast<BaseOnBase*>(base);// Base==>BaseOnBase  nullptr!

    Base* p_b;
    p_b = base;
    p_b = dynamic_cast<Base*>(base);
    p_b = publicbase;
    p_b = dynamic_cast<Base*>(publicbase);

    void * void_ptr = dynamic_cast<void*>(publicbase);// BaseOnBase ==>void ok
    void_ptr = dynamic_cast<void*>(base); // Base ==>void ok
    return 0;
}
```
# static_cast
static_cast允许相关类之间转换（基类派生类之间转换），但是不保证安全性，同时也允许隐式转换。
```
int main()
{
    Base* b = new Base();
    BaseOnBase* bb = new BaseOnBase();

    BaseOnBase* pbb = static_cast<BaseOnBase*>(b);// ok pbb == b

    void* a = new int(5);
    //int* n = a; error
    int* ptr = static_cast<int*>(a);
    void* b = static_cast<void*>(ptr);
    if (a == b)
        std::cout << "a is b" << std::endl;
    return 0;
}
output:
a is b
```
# const_cast
const变量去const,注意直接修改const 变量属于未定义行为，但是只能管一次。
```
void add(int& i)
{
    i++;
}
void changeVec(std::vector<int>& vector)
{
    vector[0] = 1;
}
void doVec(const std::vector<int>& vec)
{
    changeVec(const_cast<std::vector<int>&>(vec));
    std::cout << "dovec(): ";
    for (int i : vec)
    {
        std::cout << i << " " ;
    }
    std::cout << std::endl;
}
void eq11(const int& i)
{
    if (i == 11)
        std::cout << "eq11():i == 11" << std::endl;
}
int main() {
    
    const int i = 10;
    const int& i_f = i;
    add(const_cast<int&>(i_f));
    std::cout << i_f << std::endl; // 11 OK
    std::cout << i << std::endl; // 10 UB i is const
    eq11(i);// i == 11
    eq11(i_f);// i == 11
    if (&i == &i_f)
    {
        std::cout << "i == i_f" << std::endl;
    }
    const std::vector<int> vec{0,0,0};
    doVec(vec);
    return 0;
}
output :
11
10
eq11():i == 11
eq11():i == 11
i == i_f
dovec(): 1 0 0
```
# reinterpret_cast
任意指针向任意指针之间的转换，任意引用与引用之间的转换。
```
int main() {
    Base* b = new Base();
    BaseOnBase* bb = new BaseOnBase();

    int* i = reinterpret_cast<int*>(b);// ok
    float& f = reinterpret_cast<float&>(bb);// ok

    Base& rb = reinterpret_cast<Base&>(f);// ok
    Base* pb = reinterpret_cast<Base*>(f);// error
    return 0;
}
```

