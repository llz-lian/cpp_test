# vector中push_back与emplace_back区别
## 1.函数声明
### push_back:
#### c++ 11 前:
```
void push_back( const T& value );
```
#### c++ 20 前:
```
void push_back( const T& value );
void push_back(value_type &&__x) 
{
    emplace_back(std::move(__x));
}
```
#### c++ 20 起：
```
constexpr void push_back( const T& value );
constexpr void push_back( T&& value );
```
### emplace_back:
#### c++ 11 起：
```
template< class... Args >
void emplace_back( Args&&... args );
```
#### c++ 17 起：
```
template< class... Args >
reference emplace_back( Args&&... args );
```
#### c++ 20 起：
```
template< class... Args >
constexpr reference emplace_back( Args&&... args );
```
## 2.使用上
效率问题
```
#include <iostream>
#include <cstring>
#include <string>
#include <vector>
using namespace std;

class A {
public:
    A(int i) {
        str = to_string(i);
        cout << "构造函数" << endl;
    }
    ~A() {}
    A(const A& other) : str(other.str) {
        cout << "拷贝构造" << endl;
    }
    A(const A&& other) : str(other.str) {
        cout << "移动构造" << endl;
    }

public:
    string str;
};

int main()
{
    vector<A> vec;
    vec.reserve(10);
    for (int i = 0; i < 10; i++) {
        //vec.push_back(A(i)); //调用了10次构造函数和10次移动构造函数,若移动构造没有实现，则调用拷贝构造
        vec.emplace_back(i);  //调用了10次构造函数一次拷贝构造函数都没有调用过
    }

}
```
initialize_list
```
int main()
{
    vector<vector<int>> vec;
    //push_back
    vec.push_back({ 1,2,3 });//OK
    //emplace_back
    vec.emplace_back({ 1,2,3 });//ERR
    vec.emplace_back(vector<int>{1, 2, 3});//OK
}
```