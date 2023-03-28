# c++中空类
1. 没有任何成员函数和成员变量
2. 编译器只会生成一个字节的占位符
3. 如果空类中含有虚函数，类大小加一个指针大小
```
#include<iostream>
using namespace std;

class Virtual1{
	//void* _v_ptr; 4 or 8 bytes
	virtual void foo() {}
};
class Virtual2 {virtual void foo() {} virtual void poo() {} };
class Empty1{};
class Empty2 :public Virtual1{ virtual void foo() {} };
class Empty3 :public Virtual1,Virtual2{};
int main()
{
	std::cout << "Empty1 sizeof:" << sizeof(Empty1) << std::endl;
	std::cout << "Empty2 sizeof:" << sizeof(Empty2) << std::endl;
	std::cout << "Empty3 sizeof:" << sizeof(Empty2) << std::endl;

	std::cout << "Virtual1 sizeof:" << sizeof(Virtual1) << std::endl;
	std::cout << "Virtual2 sizeof:" << sizeof(Virtual2) << std::endl;

	/*
		x86:
			Empty1 sizeof:1
			Empty2 sizeof:4
			Empty3 sizeof:4
			Virtual1 sizeof:4
			Virtual2 sizeof:4
		x64:
			Empty1 sizeof:1
			Empty2 sizeof:8
			Empty3 sizeof:8
			Virtual1 sizeof:8
			Virtual2 sizeof:8
	*/
}
```