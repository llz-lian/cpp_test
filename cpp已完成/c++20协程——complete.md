# 协程
流程图最好背下来。

cppref：[协程](https://zh.cppreference.com/w/cpp/language/coroutines)

文章1： [Coroutine, 异步，同步，async, await](https://zhuanlan.zhihu.com/p/237067072)

文章2： [使用C++20协程（Coroutine）实现线程池](https://zhuanlan.zhihu.com/p/375279181)

文章3、4、5： [C++20 新特性 协程 Coroutines](https://zhuanlan.zhihu.com/p/349210290)


交替执行两个循环
```c++
#include<coroutine>
#include<iostream>
int iter = 0;
int max_iter = 10;
class Ping {
public:
	struct promise_type
	{
		auto initial_suspend() {return std::suspend_always{};	}
		auto final_suspend() noexcept { return std::suspend_always{}; }
		auto yield_value(int & i) 
		{
			std::cout << i << ". ping";
			return std::suspend_always{};
		};
		void return_void() {};
		Ping get_return_object() { return Ping{ handle::from_promise(*this)}; }
		void unhandled_exception(){}
	};
	using handle = std::coroutine_handle<promise_type>;
	bool resume()
	{
		if (h.done()) return false;
		h.resume();
		return true;
	}
	void destory() {h.destroy();}
private:
	handle h;
	Ping(handle h) :h(h) {}
};
class Pong {
public:
	struct promise_type
	{
		auto initial_suspend() { return std::suspend_always{}; }
		auto final_suspend() noexcept { return std::suspend_always{}; }
		auto yield_value(int & i) 
		{
			std::cout << "---pong." << std::endl;
			i++;
			return std::suspend_always{};
		};
		void return_void() {};
		Pong get_return_object() { return Pong{ handle::from_promise(*this) }; }
		void unhandled_exception() {}
	};
	using handle = std::coroutine_handle<promise_type>;
	bool resume()
	{
		if (h.done()) return false;
		h.resume();
		return true;
	}
	void destory() { h.destroy(); }
private:
	handle h;
	Pong(handle h) :h(h) {}
};
Ping ping()
{
	while(iter< max_iter)
		co_yield iter;
}
Pong pong()
{
	while (iter < max_iter)
		co_yield iter;
}
void ping_pong()
{
	auto p = ping();
	auto po = pong();
	while (p.resume()) po.resume();
	p.destory(); po.destory();
}
int main()
{
	ping_pong();
}
```
generator
```c++
import <format>;
import <iostream>;
import <coroutine>;
class Task
{
public:
    struct promise_type
    {
        std::suspend_always initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() {}
        Task get_return_object() { return { std::coroutine_handle<promise_type>::from_promise(*this)}; }
        std::suspend_always yield_value(int i) 
        {
            std::cout << std::format("{}\n", i);
            return {};
        }
    };
    void resume(){handle();}
    using Handle = std::coroutine_handle<promise_type>;
    Task(Handle h):handle(h){}
private:
    Handle handle;
};

Task count(int i)
{
    while (true)
    {
        co_yield i;
        i++;
    }
}
class Counter
{
private:
    Task task;
public:
    void next()
    {
        task.resume();
    }
    Counter(int n):task(count(n)){}
};

int main() {
    Counter count_from_24(24);
    for (int i = 0; i < 3; ++i) {
        count_from_24.next();
    }
    return 0;
}
// output:
// 24
// 25
// 26    
```
