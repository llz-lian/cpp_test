# 两种单例实现
1. namespace全局变量也可以当单例用
2. local static可以线程安全
```
namespace Single{
    // var a,b,c
    int var = 0;
    const int a = 10;
    const int b = 10;
    const vector<int> sss = {};
    static const int i = 10;
    // mutex lock
}

class SingleC {
private:
    SingleC(const SingleC &) = delete;
    SingleC(const SingleC &&) = delete;
    SingleC& operator=(const SingleC&) = delete;
    SingleC() {};
public:
    static SingleC & get() 
    {
        static SingleC cls;
        return cls;
    }
    static int s_num;
};
int SingleC::s_num = 0;
void addNum()
{
    for(int i = 0;i<1000;i++)
        SingleC::get().s_num++;
}
int main()
{
    
    vector<thread> threads;
    for (int i = 0; i < 10; i++)
    {
        threads.push_back(thread(addNum));
    }
    for (int i = 0; i < 10; i++)
    {
        threads[i].join();
    }
    std::cout << SingleC::get().s_num;
    return 0;
}
output:
10000
```