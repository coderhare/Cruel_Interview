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

