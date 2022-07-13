## C++的concept
> 链接：https://zhuanlan.zhihu.com/p/266086040


Concepts是C++20提出的一项比较重要的功能。Concepts是用来约束模板类型的语法糖。原来，模板通过SIFNAE机制进行模板的约束匹配，这样的匹配方式，我知道的缺陷有几个： 1. 匹配失败时报错信息难以阅读 2. 匹配的逻辑写的实在是难以读懂，跟模板本身逻辑耦合在一起。 3. 不同地方的匹配可能需要写很多相同模式的约束匹配的代码。Concepts的提出就是为了解决上述问题，它通过将模板的类型约束抽象出来，然后在模板定义时再使用。这样成功解耦了模板类型约束和模板本身的一些类型逻辑。 一、Concepts的定义下面是concept的定义的形式。template < template-parameter-list >
concept  concept-name = constraint-expression;
其中，constraint-expression是一个可以被eval为bool的表达式或者编译期函数。 在使用定义好的concept时，constraint-expression会根据上面template-parameter-list传入的类型，执行编译期计算，判断使用该concept的模板定义是否满足。 如果不满足，则编译期会给定一个具有明确语义的错误，即 这个concept没有匹配成功啦啦这种。 注意到，上述匹配的行为都是在编译期完成的，因此concept其实是zero-cost的。 举个例子来描述一下，最基本的concept的定义。// 一个永远都能匹配成功的concept
```c++
template <typename T>
concept always_satisfied = true;

// 一个约束T只能是整数类型的concept，整数类型包括 char, unsigned char, short, ushort, int, unsinged int, long等。
template <typename T>
concept integral = std::is_integral_v<T>;

// 一个约束T只能是整数类型，并且是有符号的concept
template <typename T>
concept signed_integral = integral<T> && std::is_signed_v<T>;
接下来，我们再简单示例一下如何使用一个concept// 任意类型都能匹配成功的约束，因此mul只要支持乘法运算符的类型都可以匹配成功。
template <always_satisfied T>
T mul(T a, T b) {
    return a * b;
}

// 整型才能匹配add函数的T
template <integral T>
T add(T a, T b) {
    return a + b;
}

// 有符号整型才能匹配subtract函数的T
template <signed_integral T>
T subtract(T a, T b) {
    return a - b;
}

int main() {
    mul(1, 2); // 匹配成功, T => int
    mul(1.0f, 2.0f);  // 匹配成功，T => float

    add(1, -2);  // 匹配成功, T => int
    add(1.0f, 2.0f); // 匹配失败, T => float，而T必须是整型
    subtract(1U, 2U); // 匹配失败，T => unsigned int,而T必须是有符号整型
    subtract(1, 2); // 匹配成功, T => int
}
```
### Concept的本质的基本理解Concept其实是一个语法糖，它的本质可以认为是一个模板类型的bool变量。定义一个concept本质上是在定义一个bool类型的编译期的变量。使用一个concept本质上是利用SFINAE机制来约束模板类型。约束模板类型，在之前的C++也可以做。但是，有了concept之后，做类型约束后，代码不仅清晰了很多，而且脑细胞也节省了不少。 为了加深对concept的本质的理解，不如试试，上面add函数，用传统的C++11应该怎么写。如下面所示：template <typename T, std::enable_if_t<std::is_integral_v<T>, T> = 0 >
T add_original(T a, T b) {
    return a + b;
}
对于C++大佬，上面这段拗口的模板代码非常简单，但是我第一次学习的时候，觉得很绕。这个是如何匹配只能是整数的呢？ 这个其实是利用模板特化和SFINAE机制来做的。下面是std::enable_if_t在VS中的STL的实现。// STRUCT TEMPLATE enable_if

```c++
template <bool _Test, class _Ty = void>
struct enable_if {}; // no member "type" when !_Test

template <class _Ty>
struct enable_if<true, _Ty> { // type is _Ty for _Test
    using type = _Ty;
};

template <bool _Test, class _Ty = void>
using enable_if_t = typename enable_if<_Test, _Ty>::type;
虽然旧的实现方法代码量更少，但是从语言设计来说，这样写我认为实在是丑陋而且难懂。我觉得主要缺点有以下：
1. add_original函数的模板中引入了一个多余的模板参数，而这个模板参数其实是没有作用的。这点我觉得比较丑陋。
2. 模板类型T的约束的逻辑，写的跟模板声明一样，这样导致逻辑耦合了。
### Concept的花式使用方法据说C++委员会为了满足各种不同喜好的使用的提案，直接支持了好多种Concept的使用方法。对此，我觉得真的脑壳疼，C++真的越走越脱离群众了。具体有以下几种，简单总结就是 有3种方式，另外再加上与auto关键字的一些结合方式。// 约束函数模板方法1
template <my_concept T>
void f(T v);

// 约束函数模板方法2
template <typename T>
requires my_concept<T>
void f(T v);

// 约束函数模板方法3
template <typename T>
void f(T v) requires my_concept<T>;

// 直接约束C++14的auto的函数参数
void f(my_concept auto v);

// 约束模板的auto参数
template <my_concept auto v>
void g();

// 约束auto变量
my_concept auto foo = ...;
Concept当然也可以用在lambda函数上，使用方法跟上面一样，也有同样数量的花式用法// 约束lambda函数的方法1
auto f = []<my_concept T> (T v) {
  // ...
};
// 约束lambda函数的方法2
auto f = []<typename T> requires my_concept<T> (T v) {
  // ...
};
// 约束lambda函数的方法3
auto f = []<typename T> (T v) requires my_concept<T> {
  // ...
};
// auto函数参数约束
auto f = [](my_concept auto v) {
  // ...
};
// auto模板参数约束
auto g = []<my_concept auto v> () {
  // ...
};
```
### concept的组合(与或非)
concept的本质是一个模板的编译期的bool变量，因此它可以使用C++的与或非三个操作符。
当然，理解上也就跟我们常见的bool变量一样啦。 例如，我们可以在定义concept的时候，使用其他concept或者表达式，进行逻辑操作。
```c++
template <typename T>
concept Integral = std::is_integral<T>::value;
template <typename T>
concept SignedIntegral = Integral<T> && std::is_signed<T>::value;
template <typename T>
concept UnsignedIntegral = Integral<T> && !SignedIntegral<T>;
当然，我们也可以在使用concept的时候使用 逻辑操作符。template <typename T>
requires Integral<T> && std::is_signed_v<T>
T add(T a, T b);
```