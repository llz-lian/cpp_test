通过模板、友元和成员指针
```c++
#include <iostream>
#include<format>
class A
{
private:
	int num = 0;
	double f = 1.;
public:
	void showInt()
	{
		std::cout << std::format("num is {}.\n", num);
	}
	void showDouble()
	{
		std::cout << std::format("f is {}.\n", f);
	}
};
//take any
template<class U, int U::*M>
struct Friend_Int
{
	friend int& friendTakeInt(U &u)//全局
	{
		return u.*M;
	}
};
template<class U, double U::* M>
struct Friend_Double
{
	friend double& friendTakeDouble(U& u)//全局
	{
		return u.*M;
	}
};
template struct Friend_Double<A, &A::f>;
template struct Friend_Int<A, &A::num>;//int A int A::*（模板显示实例化不检查）
int& friendTakeInt(A& a);
double& friendTakeDouble(A& a);

//也行
template<int A::*int_ptr>
struct TakeInt
{
	friend int& getAInt(A& a)
	{
		return a.*int_ptr;
	}
};
template TakeInt<&A::num>;
int& getAInt(A& a);

int main() 
{
	A a;
	a.showInt();// 0
	a.showDouble();//1.
	auto& friend_int = friendTakeInt(a);//OK
	auto& friend_double = friendTakeDouble(a);//OK
	friend_int = 200;
	friend_double = -2.0;
	a.showInt();//200
	a.showDouble();//-2.0

	auto& i = getAInt(a);
	i = 900;
	a.showInt();//900
}
output:
num is 0.
f is 1.
num is 200.
f is -2.
num is 900.
```