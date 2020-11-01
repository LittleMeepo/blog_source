## 遍历tuple

#### 常规思维错误方法

```c++
template<typename... Args>
std::ostream& operator<<(std::ostream& os, const std::tuple<Args...>& t)
{
    os << "(" << std::get<0>(t);
    for (size_t i = 1; i < sizeof...(Args) << ++i)
        os << ", " << std::get<i>(t); // i 出错
    return os << "]";
}

int main(int, char**)
{
    cout << make_tuple("InsideZhang", 23, "HeNan") << endl;
            // 编译出错，局部变量i不可作为非类型模板参数
    return 0;
}
```

### C++11 正确方法

```c++
template<typename Tuple, size_t N>
struct tuple_print
{
    static void print(const Tuple& t, std::ostream& os)
    {
        tuple_print<Tuple, N-1>::print(t, os);
        os << ", " << std::get<N-1>(t); 
    }
};
// 类模板的特化版本
template<typename Tuple>
struct tuple_print<Tuple, 1>
{
    static void print(const Tuple& t, std::ostream& os)
    {
        os << "(" << std::get<0>(t); 
    }
};

// operator<<
template<typename... Args>
std::ostream& operaotr<<(std::ostream& os, const std::tuple<Args...>& t)
{
    tuple_print<decltype(t), sizeof...(Args)>::print(t, os);
    return os << ")";
}
```

### C++14方法

通过`integer_sequence`快速实现

```c++
#include <iostream> 
#include <tuple>
#include <string>
#include <initializer_list>

template<typename Tuple, int... N>
void print(const Tuple& t, std::index_sequence<N...>) {
	std::initializer_list<int>{ ((std::cout << std::get<N>(t) << " "), 0)... };
	//((std::cout << std::get<N>(t) << std::endl), ...); c++17 fold表达式
}

template<typename... Args>
void fun(const std::tuple<Args...>& t) {
	print(t, std::index_sequence_for<Args...>{});
}

int main() {
	int i = 3;
	auto t = std::make_tuple("hello", 1, "world", 2, "!", i);
	fun(t);
	return 0;
}
```

库中辅助函数：

```c++
template <size_t... _Vals>
using index_sequence = integer_sequence<size_t, _Vals...>;

template <size_t _Size>
using make_index_sequence = make_integer_sequence<size_t, _Size>;

template <class... _Types>
using index_sequence_for = make_index_sequence<sizeof...(_Types)>;
```



