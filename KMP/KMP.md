## 1 问题描述
给定两个字符串 **主串**、**模式串**，判断 **模式串** 是否为 **主串** 的子串: 
- 如果是，返回 **模式串** 在 **主串** 中的位置；
- 否则，返回 -1；

## 2 Brute Force
设 主串 的长度为 $m$, 模式串 的长度为 $n$ （$m >= n$），Brute Force 匹配过程：
- 先用主串中用 以第 0 个字符开始的长度为 $n$ 的子串 与 模式串 进行匹配；如果匹配成功，则直接返回 0；如果匹配失败，再用 以 第 1 个字符开始的长度为 $n$ 的子串 与 模式串进行匹配......

时间复杂度为 $O(mn)$。

## 3 KMP
KMP 算法和 Brute Force 算法的主体思路是一样的，KMP 算法利用一个叫做 **部分匹配表（Partial Match Table）** 的结构跳过 Brute Force 过程中一些不必要的匹配，对 Brute Force进行了优化。

### 3.1 部分匹配表 是什么？
**1、部分匹配**
按照 Brute Force 思路，进行匹配：
```cpp
                  |
a b c a b c a b c x ...       （主串）
a b c a b c a b c y           （模式串）
                  |
```
当在上面位置发生失配时，需要右移模式串，然后从头开始比较：
```cpp
  |
a b c a b c a b c x ...       （主串）
  a b c a b c a b c y         （模式串）
  |
```
按照 Brute Force 思路继续匹配，会到达下面阶段：
```cpp
                  |
a b c a b c a b c x ...       （主串）
      a b c a b c a b c y     （模式串）
                  |
```
上面的过程实际上是：**寻找 $abcabcabc$ 的部分匹配 $abcabc$ 的过程，这个过程只跟 $abcabcabc$ 自身有关**

如果提前知道了 $abcabcabc$ 的部分匹配是 $abcabc$，那么我们完全可以一步到位：
```cpp
//发现失配
                  |
a b c a b c a b c x ...       （主串）
a b c a b c a b c y           （模式串）
                  |

//abcabcabc 的部分匹配是 abcabc，一步到位
                  |
a b c a b c a b c x ...       （主串）
      a b c a b c a b c y     （模式串）
                  |
```
部分匹配可以用来优化 Brute Force 的匹配过程。

**2、部分匹配表**
稍微改动下主串后，在如下位置发生失配：
```cpp
            |
a b c a b c x ...             （主串）
a b c a b c a b c y           （模式串）
            |
```
此时，我们需要借助 “$abcabc$ 的部分匹配是 $abc$ ” 来直接跳到下面的阶段：
```
            |
a b c a b c x ...             （主串）
      a b c a b c a b c y     （模式串）
            |
```
也就是说，单单知道 $abcabcabc$ 的部分匹配是远远不够的，还需要知道 $abcabc$ 等字符串的部分匹配。

实际上，我们需要知道的是 模式串 $abcabcabcy$ 的所有前缀（伪前缀）： 空串、$a$、$ab$、$abc$、$abca$、$abcab$、$abcabc$、$abcabca$、$abcabcab$、$abcabcabc$、$abcabcabcy$ 的部分匹配，来处理发生在在不同位置的失配情况。

### 3.2 如何使用部分匹配表进行优化？

部分匹配表是可以用来优化 Brute Force 的匹配过程的，但具体该如何实现呢？

实现的时候，部分匹配表是不能够直接使用的，需要将其转化为一个称作 $next$ 数组的东西，**$next[i]$ 的值为长度为 $i$ 的前缀的部分匹配的长度**。

$abcabcabcy$ 的部分匹配表及 $next$ 数组如下所示：
| 前缀长度 $i$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| 伪前缀 | 空串 | $a$ | $ab$ | $abc$ | $abca$ | $abcab$ | $abcabc$ | $abcabca$ | $abcabcab$ | $abcabcabc$ | $abcabcabcy$ |
| 部分匹配 | 空串 | 空串 | 空串 | 空串 | $a$ | $ab$ | $abc$ | $abca$ | $abcab$ | $abcabc$ | 空串
| $next$ | -1 | 0 | 0 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 0 

**注意：**
- 为了避免死循环，需要将 $next[0]$ 置为 -1（将 $next$ 数组中所有 0 置为 -1 也是可以的）；
- 对于有些问题是不需要用到整个模式串的部分匹配的，如：计算模式串在主串中第一次出现的位置，计算主串中包含的互不相交的模式串个数；但对于有些问题却是需要用到的，如：计算主串中包含的可相交的模式串的个数；（这点了解就行，实现的时候统一计算出来就行，没必要区分开来）

下面例子展示了 $next$ 数组如何使用：
```cpp
//失配时
0 1 2 3 4 5 6 7 8 9           （主串索引，当前索引 i = 6）
                  |
a b c a b c a b c x ...       （主串）
a b c a b c a b c y           （模式串）
                  |
0 1 2 3 4 5 6 7 8 9           （模式串索引，当前索引 j = 9）

//next[9] = 6，表明当在模式串的第 j = 9 位发生失配时，将模式串当前索引调整为 j = 6
0 1 2 3 4 5 6 7 8 9           （主串索引，当前索引 i = 9）
                  |
a b c a b c a b c x ...       （主串）
      a b c a b c a b c y     （模式串）
                  |
      0 1 2 3 4 5 6 7 8 9     （模式串索引，当前索引 j = 6）
```



### 3.3 如何计算 next 数组？

还有一个重要的问题：$next$ 数组怎么来？

以 $s = abcabcabcy$ 为例。

$next[0] = -1$ 和 $next[1] = 0$ 是固定的。

情况一：已知 $next[6] = 3$，如何求 $next[7]$：
```cpp
//next[6] = 3
0 1 2 3 4 5              （下标）
          |              （前缀长度 i = 6）
a b c a b c
      a b c a b c
          |              （部分匹配长度 j = 3）
      0 1 2 3 4 5        （下标）

//前缀长度变为 7 后 需要判断 s[6] 跟 s[3] 是否相同
0 1 2 3 4 5 6            （下标）
            |            （前缀长度 i = 7）
a b c a b c a
      a b c a b c a
            |            （部分匹配长度 j = ？）
      0 1 2 3 4 5 6      （下标）

//发现 s[6] == s[3]，于是 next[7] = next[6] + 1 = 4
```

情况二：已知 $next[9] = 6$ 如何求 $next[10]$ ?
```cpp
//s[9] = 6
0 1 2 3 4 5 6 7 8 9           （下标）
                |             （前缀长度 i = 9）
a b c a b c a b c
      a b c a b c a b c
                |             （部分匹配长度 j == 6）
      0 1 2 3 4 5 6 7 8 9     （下标）

//前缀的长度变为 10 后，需要判断 s[9] 跟 s[6] 是否相同
0 1 2 3 4 5 6 7 8 9           （下标）
                  |           （前缀长度 i = 9）
a b c a b c a b c y
      a b c a b c a b c y
                  |           （部分匹配长度 j == ？）
      0 1 2 3 4 5 6 7 8 9     （下标）

//发现s[9] ！= s[6] 无法推导出 next[10]
//但可以利用 next 数组将问题进行转化
//判断 s[9] 跟 s[3] 是否相同
0 1 2 3 4 5 6 7 8 9                 （下标）
                  |                 （前缀长度 i = 9）
a b c a b c a b c y
            a b c a b c a b c y
                  |                 （部分匹配长度 j == ？）
            0 1 2 3 4 5 6 7 8 9     （下标）
```

计算 $next$ 数组确实是一个动态规划问题，但解释起来有点越描越黑的感觉，想好了再来用动态规划描述，或搞明白状态机后再来解释吧。。。

### 3.4 实现
KMP 算法 ~~理解~~ 解释起来是有点难度的，但实现起来还是挺简单的。
```cpp
vector<int> get_next(const string& t) {
    vector<int> next(t.size() + 1, -1);
    int i = 0, j = -1;
    while(i < t.size()) {
        if(j == -1 || t[i] == t[j]) {
            next[++i] = ++j;
        }
        else {
           j = next[j];
        }
    }
    return next;
}

int kmp(const string& s, cosnt string& t) {
    const vector<int> next = get_next(t);
    int i = 0, j = 0, m = s.size(), n = t.size();
    //j 可能等于 -1，所以 j < t.size() 会导致 bug
    while(i < m && j < n) {
        if(j == -1 || s[i] == t[j]) {
            ++i; ++j;
        }
        else {
            j = next[j];  
        }
    }
    return j == t.size() ? i - j : -1;
}

//如果你想，KMP 也就是 6 行代码的事。
/*int kmp(const string &s, const string &t) {
    int i = 0, j = -1, m = s.size(), n = t.size();
    vector<int> next(n + 1, -1);
    while(i < t.size()) j == -1 || t[i] == t[j] ? next[++i] = ++j : j = next[j];
    i = 0, j = 0;
    while(i < m && j < n) j == -1 || s[i] == t[j] ? ++i, ++j : j = next[j];
    return j == t.size() ? i - j : -1;
}*/
```
计算 $next$ 数组 和 KMP 的过程是十分相似的：
- 之所以相似的原因：是因为两者都涉及匹配，且失配优化策略相同；
- 两者所解决的问题在本质上是不同的：
  - 计算 $next$ 数组的本质是：求解前缀的部分匹配；
  - $kmp$ 的本质是：判断模式串是否为主串的子串；

### 3.5 时间复杂度

## 4 扩展（可跳过）
### 4.1 深入理解部分匹配
**1、部分匹配与前/后缀**

对于一字符串 $s$，我们可以将其写为两个字符串相加的形式：$s = a + b$, 其中 $a$、$b$ 均为非空字符串，则称 $a$ 为 $s$ 的 **前缀**，$b$ 为 $s$ 的 **后缀**。

以字符串 $s = abcabcabc$ 为例：
- 字符串 $a$、$ab$、$abc$、$abca$、$abcab$、$abcabc$、$abcabca$、$abcabcab$ 都是 $abcabcabc$ 的前缀，它们所构成的集合 $\{$ $a$、$ab$、$abc$、$abca$、$abcab$、$abcabc$、$abcabca$、$abcabcab$ $\}$ 称为 $abcabcabc$ 的 **前缀集合**；
- 字符串 $bcabcabc$、$cabcabc$、$abcabc$、$bcabc$、$cabc$、$abc$、$bc$、$c$ 都是 $abcabcabc$ 的后缀，它们所构成的集合 $\{$ $bcabcabc$、$cabcabc$、$abcabc$、$bcabc$、$cabc$、$abc$、$bc$、$c$ $\}$ 称为 $abcabcabc$ 的 **后缀集合**；

前缀集合和后缀集合的交集为 $\{$ $abcabc$、$abc$ $\}$，而 $abcabcabc$ 的部分匹配是 $abcabc$。

**结论：字符串 $s$ 的部分匹配是 $s$的前缀集合和后缀集合交集中最长的那个元素。**

**2、部分匹配与错位**

先将 $abcabcabc$ 复制一份，然后按照如下放置：
```cpp
a b c a b c a b c
a b c a b c a b c
```
不断的移动（次数 >= 1）第二行，直到中间重叠部分相同：
```cpp
a b c a b c a b c
      a b c a b c a b c
```
**结论：字符串 $s$ 的部分匹配是 错位过程中中间重叠部分第一次相同时的中间重叠部分。**

上述结论跟前/后缀部分得到的结论本质上是一致的：它也是在寻找前缀集合和后缀集合交集中最长的那个元素。
- 第一行中间重叠部分代表的是 字符串的后缀；
- 第二行中间重叠部分代表的是 字符串的前缀；

## 真的懂了吗？
- Brute Force 匹配流程大概是怎样的？
- **部分匹配** 是什么？ **部分匹配表** 又是什么？
- **$next[i]$** 的含义是什么？ **$next$** 数组怎么用？ **$next$** 数组怎么计算？

## 练习
- [LeetCode 28. Implement strStr()](https://leetcode.com/problems/implement-strstr/)

## 参考
[如何更好地理解和掌握 KMP 算法?](https://www.zhihu.com/question/21923021)
