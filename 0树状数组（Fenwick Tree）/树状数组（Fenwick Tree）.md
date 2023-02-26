# 前缀和数组 和 差分数组

设有原数组 $nums$:
- 数组 $sum$ 称为 $nums$ 的前缀和数组，$sum[i] = nums[0] + nums[1] + ... + nums[i]$；
- 数组 $dif$ 称为 $nums$ 的差分数组， $dif[i] = nums[i] - nums[i - 1]$；
- **$nums$ 是 $dif$ 的前缀和数组，同时也是 $sum$ 的差分数组**；
- 前缀和数组 和 差分数组 只是原数组的等价表示形式，没有额外增加/删除信息；
- 前缀和数组可以简化 **查询原数组任意区间内元素和** 操作；时间复杂度变化：$O(n)->O(1)$；
- 差分数组可以简化 **原数组任意区间内元素统一增加 x** 操作；时间复杂度变化：$O(n)->O(1)$；

# 树状数组

![10.png](http://localhost:8080/image/10)

树状数组：本质上就是 **高级版的前缀和数组**。
- 前缀和数组：
  - 直接存储了前缀和信息；
  - **构建时间复杂度：** $O(n)$；
  - **查询原数组任意区间内元素的和：** 时间复杂度 $O(1)$；
  - **修改原数组中某个元素的值需要对前缀和数组进行维护：** 时间复杂度为 $O(n)$；
- 树状数组：
  - 间接存储了前缀和信息；
  - **构建时间复杂度：** $O(n)$；
  - **查询原数组任意区间内元素的和：** 实际上是查询两次前缀和，然后作差；时间复杂度 $O(logn)$；
  - **修改原数组中某个元素的值需要对树状数组进行维护：** 时间复杂度为 $O(logn)$；

规律：
- 第一层：以 1 为首项 2 为公差的等差数列；以每个节点为根节点的子树中节点个数为 1;
- 第二层：以 2 为首项 4 为公差的等差数列；以每个节点为根节点的子树中节点个数为 2;
- 第三层：以 4 为首项 8 为公差的等差数列；以每个节点为根节点的子树中节点个数为 4;
- 第四层：以 8 为首项 16 为公差的等差数列；以每个节点为根节点的子树中节点个数为 8;
- ...
- 第 $n$ 层：以 $2^{n - 1}$ 为首项 $2^n$ 为公差的等差数列；以每个节点为根节点的子树中节点个数为 $2^{n - 1}$;

## $lowbit$ 函数是什么？

lowbit 是什么？
- $lowbit(x)$：$x$ 的二进制表示最后一位 $1$ 的位权；
- $lowbit(x) = x\ \&\ (-x)$；

```cpp
x 的二进制表示为：0...1000000
x 的反码为：     1...0111111
x 的补码为：     1...1000000（也就是 -x 的二进制表示）
x & (-x)：      0...1000000
```
## 如何利用 $lowbit$ 函数构建树状数组？

```cpp
//nums 表示原数组，ft 表示 nums 的树状数组；
vector<int> build(const vector<int> &nums) {
    vector<int> ft(nums.size() + 1, 0);
    for(int i = 0; i < nums.size(); ++i) {
        ft[i + 1] += nums[i];
        int j = i + lowbit(i);
        if(j <= nums.size()) ft[j] += ft[i];
    }
    return ft;
}
```

## 在树状数组中 $lowbit$ 函数的含义是什么？

通过利用 $lowbit$ 构建 $nums$ 的树状数组 $ft$，可以发现 $lowbit$ 函数有两层含义：
- 第一层含义：$ft[i]$ 维护的是 位置 $i$ 前面 $lowbit(i)$ 个元素的和；
- 第二层含义：$ft[i]$ 与其父节点 下标之间的差距为 $lowbit(i)$，即其父节点为 $ft[i + lowbit(i)]$；

## 实现

```cpp
//树状数组下标必须是从 1 开始的
class FenwickTree{
private:
    vector<int> ft;
    int lowbit(int x) {
        return x & (-x);
    }
public:
    FenwickTree(int n) {
        ft.resize(n + 1, 0);
    }
    FenwickTree(const vector<int> &nums) {
        ft.resize(nums.size() + 1, 0);
        //这种构建方式时间复杂度为 O(nlogn)
        /*for(int i = 0; i < nums.size(); ++i) {
            add(i + 1, nums[i])；
        }*/

        //这种构建方式的时间复杂度为 O(n)
        for(int i = 1; i <= nums.size(); ++i) {
            ft[i] += nums[i - 1];
            int idx = i + lowbit(i);
            if(idx <= nums.size()) ft[idx] += ft[i];
        }
    }
    //第 idx 个元素 + x，nums[idx] + x when 1-indexed or nums[idx - 1] + x when 0-indexed
    void add(int idx, int x) {
        while(idx < ft.size()) {
            ft[idx] += x;
            idx += lowbit(idx);
        }
    }
    //前缀和查询
    int sum(int idx) {
        int res = 0;
        while(idx) {
            res += ft[idx];
            idx -= lowbit(idx);
        }
        return res;
    }
};
```

# 线段树与树状数组

两点：
- 能用树状数组解决的问题，则一定能用线段树解决；
- 能用树状数组解决的问题，就用树状数组解决（效率更高，空间占用更少）；

# 练习

- [海贼OJ 328. 楼兰图腾](http://oj.haizeix.com/problem/328)
- [海贼OJ 329. 弱化的整数问题](http://oj.haizeix.com/problem/329)
- [海贼OJ 330. 加强的整数问题](http://oj.haizeix.com/problem/330)
- [海贼OJ 331. Lost_cows](http://oj.haizeix.com/problem/331)
- [海贼OJ 332. 买票](http://oj.haizeix.com/problem/332)
- [LeetCode 1649. Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/)