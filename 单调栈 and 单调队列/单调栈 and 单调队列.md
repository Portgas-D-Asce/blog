单调队列中的队列实际上是一个双端队列。

单调栈 / 单调队列可以用来干很多事，比较典型的应用就是实现最大栈，实现最大队列。

## 1 单调栈
### 1.1 实现最大栈
最大栈实现需要借助两个栈：
- 栈 $data$：保存数据；
- 栈 $mx$：保存最大值信息；

**push(x)**
- 直接将 $x$ push 到 $data$;
- 如果 $mx$ 为空 或 $x$ 大于等于 $mx$ 栈顶元素，才将 $x$ push 到 $mx$；

**pop()**
- 如果 $mx$ 栈顶元素和 $data$ 栈顶元素相同，则对 $mx$ 进行 pop；
- 直接从 $data$ 中 pop；

**top()**
- 返回 $data$ 栈顶元素；

**max()**
- 返回 $mx$ 栈顶元素；

**empty()**
- 返回 $data$ 是否为空；

**size()**
- 返回 $data$ 的大小；

```cpp
//实际上就是维护了一个单调（递增）栈
class MaxStack {
private:
    stack<int> data;
    stack<int> mx;
public:
    void push(int x) {
        data.push(x);
        if(mx.empty() || x >= mx.top()) {
            mx.push(x);
        }
    }
    void pop() {
        if(data.top() == mx.top()) {
            mx.pop();
        }
        data.pop();
    }
    int top() {
        return data.top();
    }
    int max() {
        return mx.top();
    }
    bool empty() {
        return data.empty();
    }
    int size() {
        return data.size();
    }
};
```
### 1.2 实现 prefix first less array
维护的是单调递增栈
```cpp
//prefix first less array
vector<int> prefix(const vector<int> &nums) {
    int n = nums.size();
    vector<int> prefix_first_less(n, -1);
	stack<int> s1;
	for(int i = n - 1; i >= 0; --i) {
		while(!s1.empty() && nums[i] < nums[s1.top()]) {
		    prefix_first_less[s1.top()] = i;
			s1.pop();
		}
		s1.push(i);
	}
    return prefix_first_less;
}

//suffix first less array
vector<int> suffix(const vector<int> &nums)	{
    int n = nums.size();
	vector<int> suffix_first_less(n, n);
	stack<int> s2;
	for(int i = 0; i < n; ++i) {
		while(!s2.empty() && nums[i] < nums[s2.top()]) {
		    suffix_first_less[s2.top()] = i;
			s2.pop();
		}
		s2.push(i);
	}
    return suffix_first_less;
}
```

## 2 单调队列
### 2.1 最大队列
最大队列实现需要借助 一个队列 和 一个双端队列：
- 队列 $data$ 保存数据；
- 双端队列 $mx$ 保存最大值信息；

**push(x)**
- 直接将 $x$ push 到 $data$ 中；
- 从后往前，把 $mx$ 中比 $x$ 小的都 弹出，再把 $x$ push 到 $mx$ 中；

**pop()**
- 如果 $mx$ 队首等于 $data$ 队首，则将 $mx$ 队首弹出；
- 直接将 $data$ 队首弹出；

**front()**
- 返回 $data$ 队首；

**mx()**
- 返回 $mx$ 队首；

**empty()**
- 返回 $data$ 是否为空；

**size()**
- 返回 $data$ 的大小；

```cpp
//实际上就是维护一个单调（递减）双端队列
class MaxQueue {
private:
    queue<int> data;
    deque<int> mx;
public:
    void push(int x) {
        data.push(x);
        while(!mx.empty() && mx.back() < x) {
            mx.pop_back();
        }
        mx.push_back(x);
    }
    void pop() {
        if(mx.front() == data.front()) {
            mx.pop_front();
        }
        data.pop();
    }
    int front() {
        return data.front();
    }
    int max() {
        return mx.front();
    }
    bool empty() {
        return data.empty();
    }
    int size() {
        return data.size();
    }
};
```

### 2.2 长度不超过 k 的最大和子数组
设原数组为 $nums$，其前缀和数组（$prefix\ sum\ array$）为 $sum$，则：
- 以 $nums$ 第 $i$ 项结尾的 长度不超过 $k$ 的子数组的最大和 $mx = sum[i] - min(sum[i - 1], sum[i - 2], ..., sum[i - k])$;
- $min(sum[i - 1], sum[i - 2], ..., sum[i - k])$ 可以通过维护一个长度不超过 $k$ 的最小队列来实现；
```cpp
class MinQueue {
private:
    queue<int> data;
    deque<int> mn;
public:
    void push(int x) {
        data.push(x);
        while(!mn.empty() && mn.back() > x) {
            mn.pop_back();
        }
        mn.push(x);
    }
    void pop() {
        if(data.front() == mn.front()) {
            mn.pop_front();
        }
        data.pop();
    }
    int front()() {
        return data.front();
    }
    int min() {
        return mn.front();
    }
    int size() {
        return data.size();
    }
    bool empty() {
        return data.empty();
    }
};

int max_sum_within_k(const vector<int> &nums, int k) {
    int sum = 0, mx = -2147483648;
    MinQueue mq;
    mq.push(0);
    for(int i = 0; i < nums.size(); ++i) {
        sum += nums[i];
        mx = max(mx, sum - mq.min());
        mq.push(sum);
        if(mq.size() > k) {
            mq.pop();
        }
    }
    return mx;
}
```

思考，按这个思路来考虑 最大和子数组（卡丹算法）也可以这么做：
- 先求 $nums$ 的 前缀和数组 $sum$；
- 遍历 $sum$ 维护一个最小值变量 $mn$，则以 $nums$ 第 $i$ 项结尾的子数组组大和 $mx = sum[i] - mn$；

```cpp
int max_sum(const vector<int> &nums) {
    int mn = 0, sum = 0, mx = -2147483647;
    for(int i = 0; i < nums.size(); ++i) {
        sum += nums[i];
        mx = max(mx, sum - mn);
        mn = min(mn, sum);
    }
    return mx;
}
```

## 单调栈的深层含义
单调栈：
- 单调递增栈：栈中每个元素都是后一个元素的“第一个比它小元素”；
- 单调递减栈：栈中每个元素都是后一个元素的“的一个比它大元素”；

从头/尾开始？

严格/非严格单调？

存数字还是下标？