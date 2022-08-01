> 参考链接：https://mp.weixin.qq.com/s/Is35KL3Yr1l0yC43LtAvew

`void_t`是C++17引入的一个新特性，它的定义可能如下，很简单:
```c++
template< class... >
using void_t = void;
```
搭配`SFINAE`能在模板元编程中发挥巨大作用：
1. 在编译期判断类是否有某个类型using
```c++
template <class, class = std::void_t<>>
struct has_type : std::false_type {};

template <class T>
struct has_type<T, std::void_t<typename T::type>> : std::true_type {};
```
2. 判断是否有某个成员
```c++
template <class, class = std::void_t<>>
struct has_a_member : std::false_type {};

template <class T>
struct has_a_member<T, std::void_t<decltype(std::declval<T>().a)>> : std::true_type {};
```

3. 判断某个类是否可以迭代
```c++
template <typename, typename = void>
constexpr bool is_iterable{};

template <typename T>
constexpr bool is_iterable<T, std::void_t<decltype(std::declval<T>().begin()), decltype(std::declval<T>().end())>> = true;
```

4. 判断某个类是否有某个函数
```c++
template <class T, class = void>
struct has_hello_func : std::false_type {};

template <class T>
struct has_hello_func<T, std::void_t<decltype(std::declval<T>().hello())>> : std::true_type {}
```