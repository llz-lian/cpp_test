```
#include<iostream>
#include<vector>
#include<functional>
#include<queue>
#include<thread>
#include<atomic>
#include<mutex>
#include<future>
#include<memory>
class ThreadPool
{
private:
	using Task = std::function<void()>;
	std::queue<Task> __task_queue;//非线程安全。
	std::vector<std::thread> __threads;
	std::atomic_bool is_running = false;
	std::condition_variable wait_task_lock;
	std::mutex task_lock;
public:
	ThreadPool(int thread_nums)
	{
		is_running = true;
		__threads.push_back(
			std::thread(
				[this]() {
                    //线程任务，几乎是个死循环
					while (is_running)
					{
						{
							std::unique_lock<std::mutex> wait_lock(task_lock);
							wait_task_lock.wait(wait_lock);//阻塞
						}
						if (!__task_queue.empty())//取任务
						{
							Task task = __task_queue.front();
							__task_queue.pop();
							task();
						}
					}
				}
			)
		);
	}
	template<class F,class ...Args>
	auto submit(F && func,Args && ...args)
	{
        //返回值类型
		using func_return_type = decltype(std::forward<F>(func)(std::forward<Args>(args)...));
        //packaged_task异步获得结果，注意使用了make_shared，不要频繁submit
		auto task_ptr = std::make_shared<
			std::packaged_task<func_return_type()>>(std::bind(std::forward<F>(func), std::forward<Args>(args)...));
        //只执行任务
		__task_queue.push(
			[task_ptr]() {(*task_ptr)();}
		);
		wait_task_lock.notify_one();
        //返回结果
		return task_ptr->get_future();
	}
};
int main()
{
	ThreadPool pool(1);
	int i = 0;
	while (true)
	{
		pool.submit([&]() {
			std::this_thread::sleep_for(std::chrono::seconds(2)); 
			std::cout << "run task." << std::endl; 
			i++;
			});
		std::this_thread::sleep_for(std::chrono::seconds(3));
		std::cout << i << std::endl;
	}
}
output:
0
run task.
1
run task.
2
run task.
3
run task.
4
run task.
5
run task.
6
run task.
7
run task.
8
....
```