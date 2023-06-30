```c++
#include<type_traits>
template<bool v,class First,class ...Rest>
struct _get_or
{
	using type = First;
};
template<class FalseT,class First,class...Rest>
struct _get_or<false, FalseT, First, Rest...>
{
	using type = typename _get_or<First::value , First, Rest...>::type;
};
template<class First,class ... Rest>
struct get_or:_get_or<First::value,First,Rest...>::type // false, false_type,false_type
{

};
template<class T,class ...C>
inline constexpr bool is_any_of_in = get_or<std::is_same<T, C>...>::value;

int main()
{
	is_any_of_in<int,const int,int,double,float,int*>;
}

```