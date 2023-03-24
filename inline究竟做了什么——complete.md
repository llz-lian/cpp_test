# c++ inline
* 只能在头文件中使用
* class内部自动inline
* 允许inline修饰的函数、变量在多个编译单元存在（a.cpp b.cpp head.hpp）
```
//head.hpp
#pragma once
inline int foo(int x) {
	return x + 1;
};
//a.cpp
int fooc();
int foob();
int main()
{
	fooc() + foob();
}

//b.cpp
#include<iostream>
#include"head.h"
int foob()
{
	std::cout << "foob\n";
	return foo(2);
}
//c.cpp
#include<iostream>
#include"head.h"
int fooc()
{
	std::cout << "fooc\n";
	return foo(3);
}
```
上述代码编译时生成b.cpp、c.cpp、a.cpp。
a.cpp需要b.cpp与c.cpp中foob,fooa
foob与fooa需要foo函数
同时在b.cpp与c.cpp生成代码中存在符号foo
在最后生成a时加入inline不存在多定义符号
# inline变量
```
//head.cpp

```