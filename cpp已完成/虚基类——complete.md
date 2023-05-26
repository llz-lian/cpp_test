# 虚继承
如果类A中有方法func被B，C继承，D继承B，C。
调用D方法func不明确。
```c++
class B { public: int n{0}; void foo() {} };
class X: public B{};
class Y: public B{};
class Z: public X, Y{};
int main()
{
	Z z;
	//z.foo(); 不明确
	z.X::foo();// OK
    // z.Y::foo(); Y不可访问
}
```
上述Z的内存分布可以是
```c++
/*
	X::B::n <----Z* X* X::B*
	Y::B::n <----Y* Y::B*
*/
int main()
{
	Z z;
	X* x_ptr = &z;
	B* xb_ptr = x_ptr;
	Y* y_ptr = &z;
	B* yb_ptr = y_ptr;
	Z* z_ptr = &z;
	std::cout << 
		std::format("X * addr: {:#0x},Y * addr: {:#0x} Z * addr: {:#0x}\n",
		(size_t)x_ptr, (size_t)y_ptr, (size_t)z_ptr);
	std::cout << std::format("X::B * addr: {:#0x}, Y::B * addr : {:#0x}\n", (size_t)xb_ptr, (size_t)yb_ptr);
}
output:
X * addr: 0x73f9d0,Y * addr: 0x73f9d4 Z * addr: 0x73f9d0
X::B * addr: 0x73f9d0, Y::B * addr : 0x73f9d4
```
使用虚继承使D中只含有一个基类对象。
```c++
import <iostream>;
import <format>;
using namespace std;
class B { public: int n{0}; void foo() {} };//40
class X:virtual public B{};
class Y:virtual public B{};
class Z: public X, public Y{};
int main()
{
	Z z;
	z.foo();//OK
	z.n = 10;//OK
	std::cout << z.X::n << z.X::B::n << z.Y::n << z.Y::n << z.Y::B::n;// 1010101010
}
```
在虚继承下只有最终的类调用虚基类的构造函数，虚基类对象初始化在所有非虚对象之前。
```c++
class B {
public: 
	B(int n = 0) :n(n) { std::cout << std::format("B init n is {}\n", n); } 
	int n{ 0 }; 
	void foo() {} 
};
struct X :virtual public B 
{
	X() :B(10) { std::cout << "X init\n"; };
};
struct Y :virtual public B
{
	Y() :B(20) { std::cout << "Y init\n"; };
};
struct Z :public Y, public X
{ 
	Z() :X(), B(100), Y() {}
};
int main()
{
	Z z;
	X x;
}
output:
B init n is 100
Y init
X init
B init n is 10
X init
```
大部分编译器的虚继承通过虚基类表实现，每个虚继承的类都有自己的虚基类表。
上述Z类内部可以是:
```c++
/*
	Y.vbptr--->B_offset--| <-----Y*, Z*
	X.vbptr--->B_offset--| <-----X*
	B::n<----------------| <-----B*
*/
int main()
{
	Z z;
	B* b_ptr = &z;
	Y* y_ptr = &z;
	Z* z_ptr = &z;
	std::cout << std::format("B * addr: {:#0x},Y * addr: {:#0x} Z * addr: {:#0x}\n",(size_t)b_ptr, (size_t)y_ptr, (size_t)z_ptr);

}
output:
B init n is 100
Y init
X init
B * addr: 0xa0fe20,Y * addr: 0xa0fe18 Z * addr: 0xa0fe18
```
最后一个博客：[RTTI、虚函数和虚基类的实现方式、开销分析及使用指导
](http://baiy.cn/doc/cpp/inside_rtti.htm)