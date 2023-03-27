# 重载(override)
同一访问区中存在的相同函数名，不同参数列表的函数。
* 只有返回值不同无法重载
* const和非const可以构成重载

# 重写(overload)
子类中存在重新定义的函数，其函数名、参数列表、返回值类型与基类中某一方法一致。
子类会调用自己重写的方法。
被重写的方法必须是虚函数。
# 隐藏(hidden)
派生类类型的对象、指针、引用访问基类和派生类都有的同名函数时，访问的是派生类的函数，即隐藏了基类的同名函数。
```
class Base
{
public:
	void foo() 
	{
		std::cout << "Base::foo()" << std::endl;
	}

	virtual void vfunc()
	{
		std::cout << "Base::vfunc()" << std::endl;
	}
	void func(){}
	void func(int i){}// 重载 overload
};
class Son:public Base
{
public:
	void foo() // 隐藏
	{
		std::cout << "Son::foo()" << std::endl;
	}
	virtual void vfunc() {} // 重写 override
};
```
# override关键字
可以显式的在派生类中声明，哪些成员函数需要被重写，如果没被重写，则编译器会报错。