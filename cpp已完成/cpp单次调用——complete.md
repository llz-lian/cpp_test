# std::call_once & std::once_flag
执行一次调用，即使是多线程情况下。
std::once_flag能指示call_once调用的函数是否被调用过。
调用出现异常时，异常给调用call_once的调用方，同时不算一次调用。

正常情况：
```c++
#include<thread>
#include<mutex>
std::once_flag flag;
void foo()
{
	// call once
	std::cout << "call once" << std::endl;
}
void work()
{
	std::call_once(flag, foo);
}
int main() 
{
	work();
	work();
	std::jthread thread([]() {work(); });
	thread.join();
}
output:
call once
```
抛异常情况:
```c++
#include<thread>
#include<mutex>
std::once_flag flag;
void foo()
{
	static int call_cnt = 0;
	call_cnt++;
	if (call_cnt < 3)
	{
		throw std::exception();
	}
	// call once
	std::cout << "call once" << std::endl;
}
void work()
{
	try
	{
		std::call_once(flag, foo);
	}
	catch (const std::exception&)
	{
		std::cout << "got an exception." << std::endl;
	}
}
int main() 
{
	work();//1 try catch
	work();//2 try catch
	std::jthread thread([]() {work(); });
	thread.join();//3 call
    work();//4 call once
}
output:
got an exception.
got an exception.
call once
```




