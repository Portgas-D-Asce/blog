Hash 匹配算法的核心思想：利用 Hash 函数将字符串映射为整数，将字符串比较简化为整数比较：
- 如果两个字符串所映射的整数相同，那这两个字符串才有相同的可能性；
- 否则，两个字符串一定不相同；

注意：
- 重要的不是用这种方法去完成字符串匹配任务；
- 重要的是常数时间内计算子串的哈希值；

## 1 余数性质
设 $y_1 = k_1x + b_1$，$y_2 = k_2x + b_2$，则：
$$y_1 \times y_2 \% x = b_1 * b_2\ \%\ x$$

$$(y_1 + y_2) \% x = (b_1 + b_2)\ \%\ x$$

## 2 BKDR 哈希函数

假设 BKDR 的 基数为 $b$。对于长度为 $m$ 的字符串 $s$，其哈希值的计算公式如下：
$$hash(s) = s[0] * b^{m - 1} + s[1] * b^{m - 2} + ... + s[m - 2] * b + s[m - 1]$$

考虑到上述多项式会超过 $int$ 取值范围，所以计算公式实际上是：
$$hash(s) = (s[0] * b^{m - 1} + s[1] * b^{m - 2} + ... + s[m - 2] * b + s[m - 1])\ \%\ 2^{31}$$

## 3 暴力 Hash 匹配算法
主串 $s$ 长度为 $m$，模式串 $t$ 长度为 $n$。

暴力思路：
- 主串中包含 $m - n + 1$ 个长度为 $n$ 的子串；
- 依次计算这些子串的哈希值，然后将其与模式串的哈希值进行比较：
  - 如果子串的哈希值与模式串的哈希值相同 且 逐位比较后子串和模式串完全相同，则匹配成功；
  - 否则匹配失败，继续用下一个子串与模式串进行匹配；

存在的问题：
- 将子串映射为哈希值的时间复杂度为 $O(n)$，因此上述字符串匹配思路的时间复杂度为 $O(mn)$，这跟 Brute Force 的时间复杂度是一样的，那何必多此一举将其映射为哈希值再进行比较，直接用 Brute Force 更好啊。

## 4 线性 Hash 匹配算法
实际上，主串中任意区间的子串的哈希值是可以在常数时间获取得到的。

首先计算 主串的 “hash 数组”，该数组中元素的定义如下：
$$hash[i] = (s[0] * b^{i} + s[1] * b^{i - 1} + ... + s[i - 1] * b + s[i])\ \%\ 2^{31}$$

主串中在区间 [i, j] 的子串的哈希值为：
$$hash[i, j] = (s[i] * b^{j - i} + s[i + 1] * b^{j - i - 1} + ... + s[j - 1] * b + s[j])\ \%\ 2^{31}$$

如何利用 “hash 数组” 在常数时间内计算得到 $hash[i, j]$ 呢？
$$
\begin{aligned}
hash[i, j] 
&= (s[i] * b^{j - i} + s[i + 1] * b^{j - i - 1} + ... + s[j - 1] * b + s[j])\ \%\ 2^{31} \\
&= ((s[0] * b^{j} + s[1] * b^{j - 1} + ... + s[j]) - (s[0] * b^{i - 1} + s[1] * b^{i - 2} + ... + s[i - 1]) * b^{j - i + 1})\ \%\ 2^{31} \\
&= (s[0] * b^{j} + s[1] * b^{j - 1} + ... + s[j])\ \%\ 2^{31} - (s[0] * b^{i - 1} + s[1] * b^{i - 2} + ... + s[i - 1])\ \%\ 2^{31} * b^{j - i + 1}\ \%\ 2^{31} \\
&= (hash[j] - hash[i - 1] * (b^{j - i + 1}\ \%\ 2^{31}))\ \%\ 2^{31}
\end{aligned}
$$

注意：上述等式是完全想等的，即：
- 对于同一个字符串，所映射出来的哈希值一定相同，要么同时为 -1 （-1 转化为正余数也就是 2147483647）要么同时为 2147483647，不会出现一个为 -1 而另一个为 2147483647 的情况；
- 因此 hash 函数返回 $unsigned int$ 还是 $int$ 是没有区别的；

## 5 实现
```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

//用long long 发生冲突的可能性更小，但效率不及 int
class HashCode {
private:
    string s;
    int b; //基数为素数
    vector<int> pow_array;
    vector<int> hash_array;
public:
    HashCode(const string &s, int b)
        : s(s), b(b) {
        //次方求模
        pow_array.resize(s.size());
        pow_array[0] = 1;
        for(int i = 1; i < s.size(); ++i) {
            pow_array[i] = pow_array[i - 1] * b;
        }

        //计算 “hash 数组”
        hash_array.resize(s.size() + 1);
        hash_array[0] = 0;
        for(int i = 0; i < s.size(); ++i) {
            hash_array[i + 1] = hash_array[i] * b + s[i];
        }
    }

    //对于两个位置不同的相同子串，它们的哈希值：
    //要么同时为 -1， 要么同时为 2147483647；
    //不会出现一个为 -1 一个为 2147483647 的情况；
    int code(int p, int r) {
        return hash_array[r + 1] - hash_array[p] * pow_array[r - p + 1];
    }

    //rk算法
    int rk(const string &t) {
        int hash_code = t[0];
        for(int i = 1; i < t.size(); ++i) {
            hash_code = hash_code * b + t[i];
        }
        for(int i = 0; i <= s.size() - t.size(); ++i) {
            int temp = code(i, i + t.size() - 1);
            if(temp == hash_code) return i;
        }
        return -1;
    }
};

int main() {
    string s, t;
    while(cin >> s >> t) {
        HashCode hc(s, 13);
        cout << hc.rk(t) << endl;
    }
    return 0;
}
```

## 练习
- [海贼OJ 275. 兔子与兔子](http://oj.haizeix.com/problem/275)

