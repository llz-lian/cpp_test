# ->重载
1. 没有参数
2. 非静态成员函数（运算符）
返回指针当指针用，如果不返回指针，继续调用返回值的operator->，如果一直没有指针，大部分编译器直接保存
```c++
// VS2019
class A 
{
public:
    A(){}
    ~A(){}
    A & operator ->()
    {
        std::cout << "call ->" << std::endl;
        return *this;
    }
    int num = 1;
};
int main()
{
    A a;
    a->num;// error 
}
```

```c++
class A 
{
public:
    A(){}
    ~A(){}
    auto operator ->()
    {
        return this;
    }
    void boo()
    {
        std::cout << "boooooo" << std::endl;
    }
    int num = 1;
};
//A* operator ->(const A& a)//error “operator->”必须是成员函数	
//{
//
//}
int main()
{
    A a;
    std::cout<<a->num<<std::endl;
    a->boo();

    A* tmp = &a;
    std::cout << tmp->num << std::endl;
    tmp->boo();
}
output:
1
boooooo
1
boooooo

```