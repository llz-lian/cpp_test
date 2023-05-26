# 知乎文章：[C++ Tuple 异质容器的实现](https://zhuanlan.zhihu.com/p/453530590)
## 递归实现tuple类
首先是声明，和空的特化（递归出口）。
```c++
template<class... Args>
struct Mytuple;
template<>
struct Mytuple<>
{
	Mytuple() = default;
	~Mytuple() = default;
};
```
然后是中间类，展开拿出第一个，剩下的继承掉，构造函数注意forward。
```c++
template<class T,class... Rest>
struct Mytuple<T,Rest...>:public Mytuple<Rest...>
{
	T t;
	using next_type = Mytuple<Rest...>;
	Mytuple(T && value,Rest &&... args)
		:t(std::forward<T>(value)), next_type(std::forward<Rest>(args)...)
	{}
}
```
之后尝试实现一下get<x>(Mytuple &)，有概念谁用SFINAE?
```c++
template<size_t N, class T, class ...Args>
	requires (N<sizeof...(Args)+1)
decltype(auto) get(Mytuple<T, Args...>& my)
{
	if constexpr (N == 0)
	{
		return (my.t);
	}
	else
	{
		return (get<N - 1, Args...>(my));
	}
}
int main()
{
	Mytuple<int, double, long> t{1,2,3};
	auto& i_ret = get<2>(t);
}
```
结构化绑定接口：
```c++
struct std::tuple_size;
template<size_t N>
struct std::tuple_element;

constexpr decltype(auto) get(Mytuple<Args...>& my);

```
`struct std::tuple_size`获取tuple有多少个类。`struct std::tuple_element`根据N得到相应类型。

简单实现一下：
```c++
template<class ...Args>
struct std::tuple_size<Mytuple<Args...>>
{
	constexpr static size_t value = sizeof...(Args);
};

struct std::tuple_element<N, Mytuple<T, Args...>>
{
	using type = std::tuple_element<N - 1, Mytuple<Args...>>::type;
};
template<class T, class ...Args>
struct std::tuple_element<0, Mytuple<T, Args...>>
{
	using type = T;
};

template<class T,class... Rest>
struct Mytuple<T,Rest...>:public Mytuple<Rest...>
{
	T t;
	using next_type = Mytuple<Rest...>;
	Mytuple(T && value,Rest &&... args)
		:t(std::forward<T>(value)), next_type(std::forward<Rest>(args)...)
	{}
	template<size_t N,class ...Args>
		requires(N < sizeof...(Args))
	constexpr decltype(auto) get(Mytuple<Args...>& my)
	{
		return __get(my);
	}
	template<size_t N,class T ,class ...Args>
	constexpr decltype(auto) __get(Mytuple<T,Args...>& my)
	{
		if constexpr (N == 0)
		{
			return (t);
		}
		else
		{
			return (Mytuple<Args...>::__get(my));
		}
	}
}
```
类内get方法的第二种实现方式：
```c++
template<size_t N, class T, class ...Args>
struct TupleType
{
	using type = TupleType<N - 1,Args...>::type;
};

template<class T, class ...Args>
struct TupleType<0,T,Args...>
{
	using type = Mytuple<T, Args...>;
};
template<class T,class... Rest>
struct Mytuple<T,Rest...>:public Mytuple<Rest...>
{
	T t;
	using next_type = Mytuple<Rest...>;
	Mytuple(T && value,Rest &&... args)
		:t(std::forward<T>(value)), next_type(std::forward<Rest>(args)...)
	{}
	template<size_t N,class ...Args>
		requires(N < sizeof...(Args))
	constexpr decltype(auto) get(Mytuple<Args...>& my)
	{
		using tupel_type = TupleType<N, Args...>::type;
		return (static_cast<tupel_type&>(my).t);
	}
}
```
所有代码：
```c++
template<class... Args>
struct Mytuple;
template<>
struct Mytuple<>
{
	Mytuple() = default;
	~Mytuple() = default;
};
template<class ...Args>
struct std::tuple_size<Mytuple<Args...>>
{
	constexpr static size_t value = sizeof...(Args);
};
template<size_t N, class T, class ...Args>

struct std::tuple_element<N, Mytuple<T, Args...>>
{
	using type = std::tuple_element<N - 1, Mytuple<Args...>>::type;
};
template<class T, class ...Args>
struct std::tuple_element<0, Mytuple<T, Args...>>
{
	using type = T;
};

template<size_t N, class T, class ...Args>
struct TupleType
{
	using type = TupleType<N - 1,Args...>::type;
};

template<class T, class ...Args>
struct TupleType<0,T,Args...>
{
	using type = Mytuple<T, Args...>;
};


template<class T,class... Rest>
struct Mytuple<T,Rest...>:public Mytuple<Rest...>
{
	T t;
	using next_type = Mytuple<Rest...>;
	Mytuple(T && value,Rest &&... args)
		:t(std::forward<T>(value)), next_type(std::forward<Rest>(args)...)
	{}

	template<size_t N,class ...Args>
		requires(N < sizeof...(Args))
	constexpr decltype(auto) get(Mytuple<Args...>& my)
	{
		using tupel_type = TupleType<N, Args...>::type;
		return (static_cast<tupel_type&>(my).t);
	}
	template<size_t N,class T ,class ...Args>
	constexpr decltype(auto) __get(Mytuple<T,Args...>& my)
	{
		if (N == 0)
		{
			return (t);
		}
		else
		{
			return (Mytuple<Args...>::__get(my));
		}
	}
	template<size_t N>
		requires(N < sizeof...(Rest) + 1)
	constexpr decltype(auto) get()
	{
		using tupel_type = TupleType<N, T, Rest...>::type;
		return (static_cast<tupel_type&>(*this).t);
	}
};
int main()
{
	Mytuple<int, double, long, std::string> t{ 1,2,3,"1234" };
	auto& i_ret = t.get<0>();
	auto& d_ret = t.get<1>();
	auto& l_ret = t.get<2>();
	auto& str_ret = t.get<3>();
	auto&& [i, d, l,str] = t;
}
```