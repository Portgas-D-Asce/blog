# 简介

- 用来处理 **不相交集合** 问题
- 逻辑结构：树型数据结构
- 存储结构：顺序存储结构

# 操作
定义在其上的两个操作：

- **Find：** 查找节点所属于的集合
- **Union：** 合并两个不相交的集合

# 优化
- **路径压缩：** 让查找路径上的所有节点均指向根节点，提高下一次查找效率；
- **启发式合并：** 作用比较小，把 “秩” 小的集合合并到 “秩” 大的集合，使得合并后集合的 “秩” 处于较小状态，有利于查找。

# 实现
```cpp
class UnionFind {
private:
    vector<int> parent;
public:
    UnionFind(int n) {
        parent = vector<int>(n, 0);
        for(int i = 0; i < n; ++i) {
            parent[i] = i;
        }
    }

    int find(int node) {
        return parent[node] == node ? node : parent[node] = find(parent[node]);
    }

    void union1(int node1, int node2) {
        int root1 = find(node1);
        int root2 = find(node2);
        //if(root1 == root2) return;
        parent[root1] = root2;
    }
};
```

# 应用

图论中最小生成树算法：Kruskal

