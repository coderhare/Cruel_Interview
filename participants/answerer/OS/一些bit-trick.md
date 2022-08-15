> 多是来自data-lab的一些位运算优化，很多考虑性能的程序都会考虑使用一些trick来优化一下常数

#### 不用 for 循环，找到二进制最右侧的 1
应该是lowbit
```c++
x&-x
```
#### 将二进制表示的最右侧1置0
```c++
x&(x-1)
```

#### 用一条语句判断一个整数是否是 2 的整数次方
```c++
x&(x-1)==0
```

#### 判断整数 m 的二进制需要改变多少位才能变为 n
```c++
__builtin_popcount(n ^ m)
```

### 计算二进制中位的个数
分治计算
```c++
int bitCount(int x) {
  int _mask1 = (0x55)|(0x55<<8);
  int _mask2 = (0x33)|(0x33<<8);
  int _mask3 = (0x0f)|(0x0f<<8);
  int mask1 = _mask1|(_mask1<<16);
  int mask2 = _mask2|(_mask2<<16);
  int mask3 = _mask3|(_mask3<<16);
  int mask4 = (0xff)|(0xff<<16);
  int mask5 = (0xff)|(0xff<<8);

  int ans = (x & mask1) + ((x>>1) & mask1);
  ans = (ans & mask2) + ((ans>>2) & mask2);
  ans = (ans & mask3) + ((ans>>4) & mask3);
  ans = (ans & mask4) + ((ans>>8) & mask4);
  ans = (ans & mask5) + ((ans>>16) & mask5);

  return ans;
}
```

### 实现<=
直接做减法会有可能导致溢出，因此，需要做一些分类讨论:如果符号不同，只有 x 最高位是 1、y 最高位是 0 时才有 x <= y；如果符号相同，直接做减法也不会溢出。
```c++
int isLessOrEqual(int x, int y)
{
  int diffSign = !!((x >> 31) ^ (y >> 31));
  int diff = y + ~x + 1;
  return (diffSign & x >> 31) | (!diffSign & !(diff >> 31));
}
```

### 实现log2(x)
标准库的传参是是实数类型，然后这里的是整型，可以二分来做，但是得结合位运算，有点难度.
整型的版本本质就是找到最左边的1的位置。
断缩小最高位 1 所在的区间，每次将区间缩小一半。

使用一个变量 ans 保存区间右下标，初始时为 0，即最低位：

先移动 16 位，然后判断是不是大于 0。如果大于 0，说明 1 所在位置是大于 16 的，ans = ans + 16；如果是等于 0，那么说明 1 所在位置是小于 16 的。
下一次以 ans 为最低位，再右移 8 位，判断是否大于 0。如果大于 0，说明 1 所在位置在大于 8 位；如果等于 0，说明 1 所在位置是小于 8 的。 以此类推，相当于每一步都将 1 所在区间缩小一半，经过 5 次后，就确定了 1 的位置

```c++
int ilog2(int x)
{
  int ans = 0;
  ans = (!!(x >> 16)) << 4;                // ans = ans + 16?
  ans = ans + ((!!(x >> (8 + ans))) << 3); // ans = ans + 8?
  ans = ans + ((!!(x >> (4 + ans))) << 2); // ans = ans + 4?
  ans = ans + ((!!(x >> (2 + ans))) << 1); // ans = ans + 2?
  ans = ans + (!!(x >> (1 + ans)));        // ans = ans + 2?
  return ans;
}
```

### 计算机计算负数
由补码的性质，等于一个数取反+1
```c++
-x = ~x + 1
```