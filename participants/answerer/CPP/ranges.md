> https://zhuanlan.zhihu.com/p/349784961

range 是一个 `concept`, 只要满足了 `ranges::begin(t)`, `ranges::end(t)` 的就是 range 了. 
我们可以大致上理解为实现了 `begin()` 和 `end()` 的东西. 比如一个 vector .
range 是一个范围, 同时具备开始和结尾, 所以可以直接传来传去. 比如下面这段代码.

```c++
using namespace std::ranges;
vector<int> a{1, 2, 3};
for (int i : (a | views::reverse))
{
std::cout << i;
}
```

上面代码中 `a | views::reverse` 就是一个 range . 这段代码也可以翻译成常见的格式

```c++
auto &&__range = a | views::reverse;
for (auto iter = __range.begin(); iter != __range.end(); ++iter)
{
    std::cout << *iter;
}
```

所以使用 range 其实就像使用任何一个普通容器一样简单.这个地方出现了一个 `a | views::reverse` . 这里 | 左边就是一个 `range` , 右边是一个函数, 这个函数接受一个 `range` , 并且加工以后, 返还一个 `range` . 而中间的 operator| 则是用来把左边的 a 传递给右边的函数. 所以你也可以写 `views::reverse(a)` . 但对于长一点的, 就不是特别好看. 比如下面这个 `range`

```c++
a | views::reverse | views::transform([](int x) { return x + 1; }) | views::filter([](int x) { return x % 2; })
```

ranges 库给我们提供了一些常用的 adaptor, 方便大家使用. 
1. 比如views::transform 用来做变换的. 比如你有一个 vector, 你想看看这个 vector 每个元素加 1 是啥样, 那就可以使用这个. 当然, 用这个对原来的 vector 没有任何影响, 但是你得到了一个新的东西.
2. views::filter 用来过滤的. 比如你有一个 vector, 你想看看这个 vector 只留下偶数是啥样, 就可以用这个.
3. views::take 用来取 range 前 N 个元素, 构成一个新的 range 的.

