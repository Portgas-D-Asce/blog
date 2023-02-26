二分法的本质： **用来求解单调函数的零点！！！**
- 说得更清楚点就是：求解方程 $f(x) = y$ 的根，$f(x)$ 单调， y表示一个数字。

# 二分法求解方程任意解

## $f(x)$ 以数组形式给出

示例：
- 给定一个单调递增数组 $nums$，判断目标 $tar$ 是否存在；
- 如果存在返回 $tar$ 的下标；否则返回 -1 ；
```cpp
int binary_search(int *nums, int n, int tar) {
    int p = 0, r = n - 1;
    while(p <= r) {
        int q = p + r >> 1;
        if(nums[q] == target) return q;
        nums[q] < target ? p = q + 1 : r = q - 1;
    }
    return -1;
}
```
本质：
- 数组可以看作是自变量是数组下标，因变量为数组种元素的函数，即 $f(x) = nums[x]$；
- 上述问题的本质就是求解方程 $f(x) = nums[x] = tar$ 的解；
## $f(x)$ 为整形函数

示例：
- 判断正整数 $num$ 是否为平方数；
- 如果是平方数，返回 $num$ 的正平方根；否则返回 -1；

```cpp
int f(int x) {
    return x * x;
}

int binary_search(int (*f)(int), int num) {
    int p = 0, r = num >> 1;
    while(p <= r) {
        int q = p + r >> 1;
        int val = f(q);
        if(val == num) return q;
        val < target ? p = q + 1 : r = q - 1;
    }
    return -1;
}
```
本质：求解方程 $f(x) = x * x = num$ 的整数解。
## $f(x)$ 为浮点函数

示例：求解正浮点数 $num$ 的平方根。
```cpp
double f(double x) {
    return x * x;
}

double binary_search(double (*f)(double), double num) {
    double p = 0, r = max(1, num / 2);
    while(r - p > 0.0000001) {
        double q = (p + r) / 2;
        f(q) < num ? p = q : r = q;
    }
    return p;
}
```
本质：求解方程 $f(x) = x * x = num$ 的浮点解。

# 二分法求解方程最优解

二分最优化问题有两种类型：
- 00001111型：查找第一个 1 ；
- 11110000型：查找最后一个 1 ；

## 00001111型

```cpp
int binary_search01(int *nums, int n) {
    //使用 虚拟上界（r = n） 兼容 00000000型 情况
    int p = 0, r = n;

    //或 p != r
    while(p < r) {
        //为了避免死循环，不能额外 + 1
        int q = p + r >> 1;
        nums[q] == 1 ? r = q : p = q + 1;
    }
    return p;
}
```

## 11110000型

```cpp
int binary_search10(int *nums, int n) {
    //使用 虚拟下界（p = -1） 兼容 00000000型 情况
    int p = -1, r = n - 1;

    //或 p != r
    while(p < r) {
        //为了避免死循环，额外 + 1
        int q = p + r + 1 >> 1;
        nums[q] == 1 ? p = q : r = q - 1;
    }
    return p;
}
```
