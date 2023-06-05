# sizeof是运算符同时是关键字
# sizeof 使用
使用： sizeof(类型)或sizeof 表达式
返回： size_t类型
sizof 不返回0

注意sizeof不对表达式进行求值：
```
int main() 
{
    int a = 0;
    std::cout << sizeof((a = 100,++a,a++,a)) << ' ' << a << std::endl;
    return 0;
}
output:
4 0
```

参考cppref[🔗](https://zh.cppreference.com/w/cpp/language/sizeof)
# sizeof... 运算符
返回形参包中元素的数量。
```
template<int...N>
decltype(auto) paraNum()
{
	return sizeof...(N);
}
template<class ...T>
decltype(auto) paraNum(T... args)
{
	return sizeof...(args);
}
int main()
{
	auto i = paraNum<1, 2, 3, 4>();// i == 4
	auto j = paraNum(1, '2', 3LL, 4., i);//j == 5
}
```