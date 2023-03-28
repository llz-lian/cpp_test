# 虚函数与纯虚函数定义
类中方法声明带有`virtual`的方法为虚函数
```
class A 
{
	virtual void foo();
}
```
虚函数如下声明为纯虚函数
```
class A 
{
	virtual void foo() = 0;
}
```
# 区别
* 纯虚函数只有声明。
* 含有虚函数的类可以实例化，含有纯虚函数的类为抽象类，不能实例化。
* 纯虚函数的子类（派生类）必须重写纯虚函数。

# 虚函数表
知乎链接 [→link←](https://zhuanlan.zhihu.com/p/75172640) 。

