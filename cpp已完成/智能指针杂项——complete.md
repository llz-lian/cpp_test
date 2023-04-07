# 为什么有智能指针
便于资源管理，不需要显式调用new或delete，自动申请释放资源。
# 3中类型智能指针
auto_ptr c++ 11就被废弃 c++ 17就彻底没了，所有权乱转移，传参都转移
## 1. unique_ptr
转属所有权，指针的内存只能被一个对象持有，无法复制和移动。
```
int main()
{
    unique_ptr<int> a = make_unique<int>(1);
    int i = *a;// 1
    // auto b = a;  //error
    // auto b(a);
    auto c = std::move(a);// ok
    auto d = a.get();// nullptr
    i = *c;// 1
}
```
## 2. shared_ptr
共享同一对象的指针。存在引用计数，引用计数为0时释放持有资源。可能出现循环引用（感觉不常见）。比较常用。
循环引用实例：
```
class B;
class A 
{
public:
    A() 
    {
        std::cout << "A construct" << std::endl;
    }
    ~A()
    {
        std::cout << "A deconstruct" << std::endl;
    }
    shared_ptr<B> b_ptr;
};
class B
{
public:
    B()
    {
        std::cout << "B construct" << std::endl;
    }
    ~B()
    {
        std::cout << "B deconstruct" << std::endl;
    }
    shared_ptr<A> a_ptr;
};
int main()
{
    {
        shared_ptr<A> a = make_shared<A>();
        shared_ptr<B> b = make_shared<B>();
        a->b_ptr = b;
        b->a_ptr = a;
    }
    // leak

}
output:
A construct
B construct
```
# 3. weak_ptr
不持有对象，但是能感知对象是否存在。除了避免循环引用，主要用于观察者模式。

# 线程安全
引用计数是线程安全的，atomic的
智能指针其他均可以认为不安全
如果需要安全智能指针，可以试试C++20的原子智能指针std::atomic<std::shared_ptr>，std::atomic<std::weak_ptr>

# 简易shared实现
```
struct Mycount
{
    int weak_cnt = 0;//atomic
    int strong_cnt = 0;
};
template<class T>
class Mysmart
{
public:
    Mysmart(T* ptr) 
    {
        std::cout << "create table" << std::endl;
        cnt = new Mycount;
        cnt->strong_cnt++;
        this->ptr = ptr;
    };
    ~Mysmart()
    {
        __minusCnt();
    }
    Mysmart(const Mysmart& other)
    {
        cnt = other.cnt;
        cnt->strong_cnt++;
        ptr = other.ptr;
    };
    Mysmart(const Mysmart&& other) 
    {
        cnt = other->cnt;
        ptr = other->ptr;
    };
    const T* get()
    {
        return ptr;
    }
    const Mysmart& operator =(const Mysmart& eq)
    {
        if (eq.ptr == ptr)
            return *this;
        __minusCnt();
        cnt = eq.cnt;
        cnt->strong_cnt++;
        ptr = eq.ptr;
    }
private:
    Mycount* cnt;
    const T* ptr;
    void __minusCnt()
    {
        cnt->strong_cnt--;
        if (cnt->strong_cnt == 0)
        {
            std::cout << "delete data: "<<*ptr<<std::endl;
            delete ptr;
            if (cnt->weak_cnt == 0)
            {
                std::cout << "delete table\n";
                delete cnt;
            }
        }
    }
};

int main()
{
    Mysmart a(new int(5));//create table
    {   auto b = a;
    b = new int(10);//create table
    }//delete data: 10  delete table
    Mysmart c = a;
    a = new int(7);//create table
    Mysmart d(Mysmart(new int(9)));//create table
    return 0;
}//delete data 5
/*
stack
top
↓       d //delete data 9 delete table
↓       c //delete data 5 delete table
Bottom  a //delete data 7 delete table
*/
output:
create table
create table
delete data: 10
delete table
create table
create table
delete data: 9
delete table
delete data: 5
delete table
delete data: 7
delete table
```
# weak简单实现
```

template<class T>
class Myweak
{
public:
    Myweak() {};
    ~Myweak() {
        __minusCnt();
    };
    Myweak(const Myweak& cp)
    {
        ptr = cp.ptr;
        cnt = cp.cnt;
        cnt->weak_cnt++;
    }
    Myweak(const Myweak&& cp)
    {
        ptr = cp.ptr;
        cnt = cp.cnt;
    }
    Myweak(const Mysmart& smart)
    {
        ptr = smart.ptr;
        cnt = smart.cnt;
        cnt->weak_cnt++;
    }
private:
    Mycount* cnt = nullptr;
    const T* ptr = nullptr;
    void __minusCnt()
    {
        cnt->weak_cnt--;
        if (cnt->strong_cnt == 0 && cnt->weak_cnt == 0&& cnt!=nullptr)
        {
            std::cout << "delete table\n";
            delete cnt;
        }
    }
};
```