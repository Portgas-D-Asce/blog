## $shift\ and$ 算法
```cpp
//s 为主串，t 为模式串
int shift_and(const string &s, const string &t) {
    vector<int> d(256, 0);
    int m = s.size(), n = t.size();
    for(int i = 0; i < n; ++i) {
        d[t[i]] |= 1 << i;
    }
    int p = 0, mask = 1 << n - 1;
    for(int i = 0; i < m; ++i) {
        p = (p << 1 | 1) & d[s[i]];
        if(p & mask) return i - n + 1;
    }
    return -1;
}
```

## $shift\ or$ 算法
```cpp
//s 为主串，t 为模式串
int shift_or(const string &s, const string &t) {
    vector<int> d(256, -1);
    int m = s.size(), n = t.size();
    for(int i = 0; i < n; ++i) {
        d[t[i]] &= ~(1 << i);
    }
    int p = 0, mask = 1 << n - 1;
    for(int i = 0; i < m; ++i) {
        p = (p << 1) | d[s[i]];
        if(!(p & mask)) return i - n + 1;
    }
    return -1;

}
```

$s$ 为母串，长度为 $m$；$t$ 为模式串,长度为 $n$：

当模式串长度 $n < 32$ 时，可以用 $int$ 类型来保存 $p$ 和 $d$ 中的元素， 更新 $p$ 的时间复杂度为 $O(1)$，$shift\ and$ 算法时间复杂度为 $O(m)$；

当模式串过长时，需要自定义数据类型来保存 $p$ 和 $d$ 中的元素。问题是，能实现一种自定义数据类型使得更新 $p$ 的时间复杂度为 $O(1)$ 吗？
- 如果可以，自定义数据类型该怎么定义？
- 如果不可以，那么 $shift\ and$ 算法的时间复杂度就不为 $O(m)$ 了。所以说，$shift\ and$ 算法只适用于模式串较短的匹配任务？