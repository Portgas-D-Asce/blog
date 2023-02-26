关键词：区间信息、单点修改、单点查询、区间查询、懒标记。
## 1 线段树概要
线段树是对 **区间信息** 进行快速查询/修改的一种数据结构。

线段树可以在 $O(logn)$ 的时间复杂度下实现 **单点修改**、 **区间修改**、**区间查询**（区间求和，区间求乘积，区间最大值，区间最小值等等）操作。

## 2 如何存储线段树
树高说明：
- 按边算高；
- 根节点为空，树高为 -1；
- 只有一个根节点，树高为 0；
- 根节点所在层为树的第 0 层；

对于一个长度为 $n$ 的数组，假设其线段树的高度为 $H$：
- 线段树的第 $0$ 到 $H - 1$ 层，构成了一棵满二叉树；
- 线段树叶子节点数为： $n \in (2^{H - 1}, 2^H]$；

通常都是将线段树看成一棵满二叉树（只不过最后一层有些节点为空罢了），然后用一维数组来存储这个满二叉树（也就是线段树）。

那么一个长度为 $n$ 的数组，其线段树该用多长的数组来保存呢？
- 一棵高度为 $H$ 的满二叉树，包含 $2^{H + 1} - 1$ 个节点；可以先根据 $n$ 计算出线段树的高度 $H$，然后用上述公式确定数组的长度；
- 为了方便，一般直接将数组长度设置为 $4n$；

思考：当将线段树堪称一棵完全二叉树，原数组长度 和 存储线段树的数组的长度 之间的关系是怎样的？

## 3 如何构建线段树

## 4 单点修改 与 区间查询
为了更加深刻的理解本节内容，我们通过一个形象的例子进行介绍。姑且就称作是 “人口统计” 吧：
- 已知数据：全国每家每户的人口数（原数组）；
- 建立线段树：就是对全国人口进行分级统计的过程：每个村有多少人，每个乡/镇有多少人，每个县有多少人，...，每个省有多少人，全国有多少人；

### 4.1 单点修改
单点修改：可以看成某个家庭有新生儿诞生（或老人去世），这时需要对分级统计的结果进行修改；
- 首先，找到记录该家庭人口的数据，然后对其人口数据进行修改（递归的过程）；
- 该家庭人口数变了，那它所在的村，乡/镇，县，...,全国的人口统计结果也需要更新（回溯的过程）；

### 4.2 区间查询
区间查询：可以看成是查找任意区域的人口数目：
- 暴力做法：找到查询区域范围内所有家庭，然后把他们的人口数加起来就行了（即，查询区域 = 家庭1 + 家庭2 + ... + 家庭n）；
- 基于分级统计结果的方法：如果某个县完全属于查询区域，那直接把该县的人口加到结果上就可以了，没必要再去查看它下面的每个家庭到底有多少人（查询区域 = 村1 + 村2 + ... + 村n + 乡1 + 乡2 + ... + 乡n + 县1 + 县2 + ... + 县n + ...）；

### 4.3 实现
```cpp
void update(int *seg, int idx) {
    seg[idx] = seg[idx << 1] + seg[idx << 1 | 1];
}

//nums:原数组，[p, r]:对哪个区间建线段树；seg:线段树；idx:线段树的节点索引；
void build(int *nums, int p, int r, int *seg, int idx) {
    if(p == r) {
        seg[idx] = nums[p];
        return;
    }
    int q = p + r >> 1;
    build(nums, p, q, seg, idx << 1);
    build(nums, q + 1, r, seg, idx << 1 | 1);
    update(seg, idx);
}

//seg,线段树；idx,节点下标；[p, r],节点所维护的区间；x:修改原数组中第几个元素；y:修改后的值；
void modify(int *seg, int idx, int p, int r, int x, int y) {
    if(p == r) {
        seg[idx] = y;
        return;
    }
    int q = p + r >> 1;
    //如果待修改元素在当前节点所维护区间的左半部分
    if(x <= q) { 
        modify(seg, idx << 1, p, q, x, y);
    } else { //如果待修改元素在当前节点所维护区间的右半部分
        modify(seg, idx << 1 | 1, q + 1, r, x, y);
    }
    update(seg, idx);
}

//seg,线段树；idx，节点下标；[p, r]，节点所维护的区间；[x, y]:待查询区间；
int query(int *seg, int idx, int p, int r, int x, int y) {
    //如果节点所维护的区间在查询区间范围内
    if(x <= p && r <= y) return seg[idx];
    int q = p + r >> 1;
    int sum = 0;

    //当前节点所维护区间的左半部分[p, q];
    //如果查询区间和左半部分存在交集
    if(x <= q) {
        sum += query(seg, idx << 1, p, q, x, y);
    }

    //当前节点所维护区间的右半部分[q + 1, r];
    //如果查询区间和右半部分存在交集
    if(y > q) {
        sum += query(seg, idx << 1 | 1, q + 1, r, x, y);
    }
    return sum;
}
```

## 5 区间修改 与 区间查询
若需要进行区间修改，则需要通过 “懒标记” 的方式，来实现线段树。
- 基于单点修改的区间查询：可以对区间范围内的所有元素（$m$ 个）执行一次 单点修改操作，但其时间复杂度为 $mlogn$；
- 使用懒标记的区间修改的时间复杂度为 $logn$；
  - 并没有将修改落实到每个元素，而是当某个区域内所有元素都需要修改时，仅对该区域进行简单标记；
  - 只有当查询的时候，才会将标记落实为修改；

### 5.3 实现
```cpp
//seg：线段树；idx：节点下标
void update(int *seg, int idx) {
    seg[idx] = seg[idx << 1] + seg[idx << 1 | 1];
}

//懒标记下放
//seg：线段树；tag：懒标记数组；idx：节点下标；[p, r]：节点所维护的区间；
void down(int *seg, int *tag, int idx, int p, int r) {
    if(tag[idx] == 0) return;
    int q = p + r >> 1;
    seg[idx << 1] += (q - p + 1) * tag[idx];
    tag[idx << 1] += tag[idx];
    seg[idx << 1 | 1] += (r - q) * tag[idx];
    tag[idx << 1 | 1] += tag[idx];
    tag[idx] = 0;
}

//seg：线段树；tag：懒标记数组；idx：节点下标；[p, r]：节点所维护的区间；nums：原数组；
void build(int *seg, int *tag, int idx, int p, int r, int *nums) {
    if(p == r) {
        seg[idx] = nums[p];
        return;
    }
    int q = p + r >> 1;
    build(seg, tag, idx << 1, p, q, nums);
    build(seg, tag, idx << 1 | 1, q + 1, r, nums);
    update(seg, idx);
    tag[idx] = 0;
}
//将 [x, y] 区间内所有元素统一增加 z（当然也可以将所有元素设置为 z）
//seg：线段树；tag：懒标记数组；idx：节点下标；[p, r]：节点所维护的区间；[x, y]：需要修改的区间；z：增量；
void modify(int *seg, int *tag, int idx, int p, int r, int x, int y, int z) {
    //如果节点所维护的区间在修改区间范围内
    if(x <= p && r <= y) {
        seg[idx] += (r - p + 1) * z;
        tag[idx] += z;
        return;
    }

    //直接去掉也是没有问题的
    down(seg, tag, idx, p, r);

    int q = p + r >> 1;
    //当前节点所维护区间的左半部分[p, q];
    //如果待修改区间和左半部分存在交集
    if(x <= q) {
        modify(seg, tag, idx << 1, p, q, x, y, z);
    }
    //当前节点所维护区间的右半部分[q + 1, r];
    //如果待修改区间和右半部分存在交集
    if(y > q) {
        modify(seg, tag, idx << 1 | 1, q + 1, r, x, y, z);
    }
    update(seg, idx);
}

int query(int *seg, int *tag, int idx, int p, int r, int x, int y) {
    //如果节点所维护的区间在查询区间范围内
    if(x <= p && r <= y) return seg[idx];

    down(seg, tag, idx, p, r);
    int q = p + r >> 1;
    int sum = 0;
    
    //当前节点所维护区间的左半部分[p, q];
    //如果查询区间和左半部分存在交集
    if(x <= q) {
        sum += query(seg, tag, idx << 1, p, q, x, y);
    }

    //当前节点所维护区间的右半部分[q + 1, r];
    //如果查询区间和右半部分存在交集
    if(y > q) {
        sum += query(seg, tag, idx << 1 | 1, q + 1, r, x, y);
    }
    return sum;
}
```