**$dijkstra$ ：** 用于求解代权有向图的单源最短路径问题，要求权重值非负。

**开/闭集合：**
- **开集合：** 还未找到最短路径的节点的集合；
- **闭集合：** 已经找到最短路径的节点的集合；

**算法步骤：** 不断地从开集合中取出距离起点最近的点，将其加入到闭集合当中，并对与该点相邻的点进行松弛操作。

```cpp
vector<int> dijkstra(const vector<vector<pair<int, int>>> &outgoing， int s) {
    int n = outgoing.size(), mx = 2147483647;

    vector<int> visited(n, 0), dist(n, mx);
    dist[s] = 0;

    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({dist[s], s});
    while(!pq.empty()) {
        int u = pq.top().second;
        pq.pop();

        //第一处使用 visited 数组
        if(visited[u]) continue;
        visited[u] = 1;
        for(int i = 0; i < outgoing[u].size(); ++i) {
            int v = outgoing[u][i].first, w = outgoing[u][i].second;

            //第二处使用 visited 数组
            if(visited[v]) continue;
            int temp = dist[u] + w;
            if(temp < dist[v]) pq.push({dist[v] = temp, v});
        }
    }
    return dist;
}
```

时间复杂度：$O(ElogV)$

思考 1：开集合使用优先队列维护，为什么无需初始化？
```cpp
for(int i = 0; i < n; ++i) {
    pq.push({mx, i});
}
```

思考 2：经行松弛操作后，会向优先队列中插入一条新边，而被替代的旧边并未被删除，这样是否会导致优先队列中的元素越来越多，最终导致效率低下？
- 不会，$pq.size()$ < 松弛次数 < 边的条树；

思考 3：$visited$ 数组有什么用？代码中有两处使用了 $visited$ 数组，使用动机是什么？

思考 4：当图为完全图时，可以使用双重循环代替优先队列（最小生成树的 $prim$ 算法也是一样的），时间复杂度可以达到 $O(V^2)$。
