应用范围：无向连通图。
- 边权重可正，可负，可为零；

## Kruskal

$kruskal$ 是基于 **并查集** 来实现的：
- 先将所有边按权重排序；
- 按权重从小到达遍历边：
  - 如果边的两个顶点属于已经属于同一集合，该边不属于最小生成树；
  - 否则，该边属于最小生成树；

```cpp
vector<int> init(int n) {
    vector<int> uf(n);
    for(int i = 0; i < n; ++i) uf[i] = i;
    return uf;
}

int find(vector<int> &uf, int x) {
    return uf[x] == x ? x : (uf[x] = find(uf, uf[x]));
}

bool merge(vector<int> &uf, int x, int y) {
    int px = find(uf, x), py = find(uf, y);
    if(px == py) return false;
    uf[x] = y;
    return true;
}

int mst(vector<tuple<int, int, int>> &edges, int n) {
    int total = 0;
    vector<int> uf = init(n);

    sort(edges.begin(), edges.end());

    for(int i = 0; i < edges.size(); ++i) {
        int w = get<0>(edges[i]);
        int u = get<1>(edges[i]);
        int v = get<2>(edges[i]);
        if(merge(uf, u, v)) total += w;
    }
    return total;
}
```
时间复杂度：$O(ElogV)$

## Prim
开/闭集合（以 $dijkstra$ 算法为例）：
- **开集合：** 还未找到最短路径的节点的集合；
- **闭集合：** 已经找到最短路径的节点的集合；

$prim$ 算法 和 $dijkstra$ 算法十分相似，区别在于：
- **$dijkstra$ 算法：** 总是把 **开集合中距离起点最近的点加入到闭集合**；
- **$prime$ 算法：** 总是把 **开集合中距离已生成的最小生成树最近的点加入到闭集合**；

实现
```cpp
int mst(const vector<vector<pair<int, int>>> &outgoing) {
    int total = 0, n = outgoing.size(), mx = 1000000;
    vector<int> visited(n, 0), dist(n, mx);
    
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    //不需要进行初始化
    /*for(int i = 0; i < n; ++i) {
        pq.push({mx, i});
    }*/
    
    pq.push({0, 0});
    dist[0] = 0;
    while(!pq.empty()) {
        auto pr = pq.top();
        pq.pop();
        int u = pr.second;
        if(visited[u]) continue;
        visited[u] = 1;
        total += pr.first;
        
        for(int i = 0; i < outgoing[u].size(); ++i) {
            int v = outgoing[u][i].first;
            if(visited[v]) continue;
            int w = outgoing[u][i].second;
            //松弛后的边添加进来了
            //松弛前的边依旧保存在优先队列中
            //pq.size() < 松弛次数 < 边的条数
            if(w < dist[v]) pq.push({dist[v] = w, v});
        }
    }
    return total;
}
```
时间复杂度：$O(ElogV)$

## 完全图最小生成树算法

该方法本质上是 $prim$ 算法。

```cpp
int mst(const vector<vector<int>> &matrix) {
    int total = 0, n = matrix.size(), mx = 1000000;
    //标记及距离集于一身，prim 也可以这样
    vector<int> dist(n, mx);
    
    int u = 0;
    dist[u] = 0;
    for(int i = 0; i < n; ++i) {
        total += dist[u];
        //将 u 标记为已访问状态
        dist[u] = 0;
        
        //找到距离已形成最小生成树最近的点
        int w = mx, idx = 0;
        for(int v = 1; v < n; ++v) {
            if(dist[v] == 0) continue;
            //松弛
            dist[v] = min(dist[v], matrix[u][v]);
            //取最小
            if(dist[v] < w) w = dist[idx = v];
        }
        u = idx;
    }
    return total;
}
```
时间复杂度：$O(V^2)$， 显然，当在图是完全图时，效率高于 $prim$ 和 $kruskal$ 算法（时间复杂度低，没用 优先队列/并查集 等数据结构）。

