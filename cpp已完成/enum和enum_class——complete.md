# 枚举类型
将一个变量限定在某几个值
# enum与enum class区别
enum内的声明会污染整个作用域，如下
```
namespace A{
    enum NUMS:int{NUM0,NUM1,NUM2};
    enum class GOOD_NUMS:int{NUM3= 3,NUM4 = 4};
}

int main()
{
    using namespace A;
    auto num0 = NUM0; // OK
    // auto num3 = NUM3;//error
    auto num4 = GOOD_NUMS::NUM4;
}
```
