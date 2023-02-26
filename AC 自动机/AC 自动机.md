## 10 AC 自动机
```cpp
#include <iostream>
#include <string>
#include <queue>
using namespace std;

const int SIZE = 26;
const char BASE = 'A';

class TrieNode {
public:
    bool flag;
    TrieNode *fail;
    TrieNode *child[SIZE];
    TrieNode() {
        flag = false;
        fail = nullptr;
        for(int i = 0; i < SIZE; ++i) {
            child[i] = nullptr;
        }
    }
    ~TrieNode() {
        for(int i = 0; i < SIZE; ++i) {
            delete child[i];
        }
    }
};

class AC_Automaton {
private:
    TrieNode *root;
    TrieNode *temp;
public:
    Ac_Automaton() {
        root = new TrieNode();
        temp = new TrieNode();
        root->fail = temp;
        for(int i = 0; i < SIZE; ++i) {
            temp->child[i] = root;
        }
    }
    ~Ac_automaton() {
        for(int i = 0; i < SIZE; ++i) {
            temp->child[i] = nullptr;
        }
        delete root;
        delete temp;
    }
    void insert(const string &s) {
        TrieNode *cur = root;
        for(int i = 0; i < SIZE; ++i) {
            int idx = s[i] - BASE;
            if(cur->child[idx] == nullptr) {
                cur->child[idx] = new TrieNode();
            }
            cur = cur->child[idx];
        }
        cur->flag = true;
    }
    void build() {
        queue<TrieNode *> que;
        que.push(root);
        while(!que.empty()) {
            TrieNode *u = que.front();
            que.pop();
            for(int i = 0; i < SIZE; ++i) {
                if(u->child[i] == nullptr) continue;
                TrieNode *v = u->child[i];
                TrieNode *pre = u;
                while(pre->fail->child[i] == nullptr) {
                    pre = pre->fail;
                }
                v->fail = pre->fail->child[i];
                que.push(v);
            }
        }
    }
    int match(const string &s) {
        int cnt = 0;
        TrieNode *cur = root;
        for(int i = 0; i < s.size(); ++i) {
            int idx = s[i] - BASE;
            while(cur->child[idx] == nullptr) {
                cur = cur->fail;
            }
            cur = cur->child[i];

            TrieNode *pre = cur;
            while(pre != root) {
                if(pre->flag){
                    ++cnt;
                }
                pre = pre->fail;
            }
        }
        return cnt;
    }
};
```