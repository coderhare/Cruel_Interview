数位DP的模板比较固定，一般都需要记录的有：位数，当前某个位的计数情况，与原数last比较之后的大小限制，以及前导零限制。
> https://leetcode.cn/problems/number-of-digit-one/ 、

借此可以很容易完成下题
```c++
class Solution {
    int f[10][10][2][2];
public:
    int countDigitOne(int n) {
        vector <int> a;
        while(n){
            a.push_back(n % 10);
            n /= 10;
        }
        auto dfs = [&](auto & dfs, int pos, int cur, bool limit, bool lead)->int{
            int ans = 0;
            if(pos == -1) return cur;
            if(f[pos][cur][limit][lead] != -1) return f[pos][cur][limit][lead];
            for (int v = 0; v <= (limit? a[pos]: 9); v++) {
                if(lead && !v) ans += dfs(dfs, pos - 1, cur, limit && v == a[pos], 1);
                else ans += dfs(dfs, pos - 1, cur + (v == 1), limit && v == a[pos], 0);
            }
            return f[pos][cur][limit][lead] = ans;
        };
        return dfs(dfs, a.size() - 1, 0, 1, 1);
    }
};
```