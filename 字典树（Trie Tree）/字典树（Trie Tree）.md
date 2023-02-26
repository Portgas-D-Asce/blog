## 9 字典树（Trie）
```cpp
#include <stdio.h>
#include <malloc.h>
#include <string.h>

#define base 26

typedef struct Node {
    int flag;
    struct Node *child[base];
} Node;

Node *init() {
    Node *p = (Node *)malloc(sizeof(Node));
    p->flag = 0;
    memset(p->child, 0, sizeof(p->child));
    return p;
}

void clear(Node *root) {
    if(root == NULL) return;
    for(int i = 0; i < base; ++i) {
        clear(root->child[i]);
    }
    free(root);
}

void insert(Node *p, const char *s) {
    for(int i = 0; s[i]; ++i) {
        int idx = s[i] - 'a';
        if(p->child[idx] == NULL) {
            p->child[idx] = init();
        }
        p = p->child[idx];
    }
    p->flag = 1;
}

void output(Node *root, int k, char *s) {
    s[k] = 0;
    if(root->flag) {
        printf("%s\n", s);
    }
    for(int i = 0; i < base; ++i) {
        if(root->child[i] == NULL) continue;
        s[k] = 'a' + i;
        output(root->child[i], k + 1, s);
    }
}

int main() {
    int n;
    char str[100];
    scanf("%d", &n);
    Node *root = init();
    for(int i = 0; i < n; ++i) {
        scanf("%s", str);
        insert(root, str);
    }
    output(root, 0, str);
    clear(root);

    return 0;
}
```