## 红黑树认知
### 性质

红黑树，是一种平衡二叉搜索树，它满足以下 5 条性质：
- 性质 1 : 所有节点，非黑即红；
- 性质 2 : 根节点为黑色；
- 性质 3 : 叶节点（NIL）为黑色；
- 性质 4 : 红色节点的两个子结点都是黑色；
- 性质 5 : 对于每个节点，从该节点到叶节点的简单路径上，黑色节点数目相同；

### 如何保证平衡
同 AVL 树是一样的，红黑树本质上还是通过树高来控制平衡的，AVL的树高控制条件更严格；

红黑树中最长路径不会超过最短路径的 2 倍；
- 为什么不超过 2 倍？
  - 最长路径和最短路径上黑色节点个数相同（性质 5）；
  - 路径上黑色节点个数一定大于红色节点个数（性质 4）；
- 一棵有 n 个内部节点的

## 插入

以下内容对于理解红黑树插入操作很重要，目前不理解没关系，但一定要结合后续内容好好理解：
- 插入操作如果导致冲突，那么冲突一定是 **双红冲突** （孙子节点为红色，父节点也为红色，俗称父子矛盾）；
- 插入操作的核心就是如何解决双红冲突（姑且称之为 **插入调整**）；
- 插入调整策略：站在 **爷爷节点** 往下看；


### 插入的节点是红色

为什么插入节点是红色的？黑色的不行吗？
- 如果是黑色的，插入新节点 **一定** 会产生冲突（违反性质 5）；
- 如果是红色，**可能** 会产生冲突（双红，违反性质 4）；
- 显然，插入红色节点更好。

### 情况一

叔叔节点为红色时。

{% asset_img 1.png %}

解决办法：
- 叔节点和父节点改成黑色（解决双红冲突）；
- 爷爷节点改成红色（保证黑高不变）；
- 但爷爷节点和它的父节点可能产生新的冲突，需要不断地处理，直到没有冲突为止；

### 情况二
叔节点为黑色时。

以 LL 型（爷爷节点的左孩子为红色 且 左孩子的左孩子为红色）为例：
{% asset_img 2.png %}

解决办法：
- 抓着爷爷节点 “大右旋”
- 将父节点调整为黑色（解决双红冲突），将爷爷节点调整为红色（保证黑高不变）；

其它类似情况还有：
- RR 型：爷爷节点的右孩子为红色 且 右孩子的右孩子为红色
  - 抓着爷爷节点 “大左旋”；
  - 同 LL 型；
- LR 型：爷爷节点的左孩子为红色 且 左孩子的右孩子为红色
  - 抓着爷爷节点的左孩子 “小左旋”（无需进行颜色调整）；
  - 按照 LL 型进行处理；
- RL 型：爷爷节点的右孩子为红色 且 右孩子的左孩子为红色
  - 抓着爷爷节点的右孩子 “小右旋”（无需进行颜色调整）；
  - 按照 RR 型进行处理；

<!--无论是情况一还是情况二，最终结果都出奇的一致：爷爷和孙子戴红帽子，父亲和叔叔戴黑帽子（果然隔代亲）-->

## 删除
以下内容对于理解红黑树删除操作很重要，目前不理解没关系，但一定要结合后续内容好好理解：
- 删除操作产生的冲突一定是 “双重黑” 冲突；
- 删除操作的核心就是如何解决 “双重黑” 冲突（**删除调整**）；
- 删除调整策略：站在 **父节点** 看。

### 什么时候删除调整
同 AVL 树一样，删除度为 2 的节点可以转化为删除度为 0 或 1 的节点，删除节点实际上只有两种情况：删除度为 0 的节点 和 删除度为 1 的节点。

何时需要进行删除调整：
- 当删除点是红色时：
  - 度为 0（无需删除调整）：删除即可；
  - 度为 1（无需删除调整）：独生子取代父亲即可；
- 当删除节点是黑色时：
  - 度为 0 时（需要删除调整）：删除后，产生了一个双重黑的 NIL （违反性质 5）；
  - 度为 1 时（无需删除调整）：它的独生子一定为红色（如果不为红色，就违反性质 5 了）；独生子取代父亲，并把独生子染成黑色；

**结论：只有删除黑色节点且导致双层黑的时候，才需要删除调整。**

### 情况一
双重黑的兄弟是黑色 且 兄弟的两个儿子也是黑色：
{% asset_img 3.png %}

解决方法：
- 双重黑多的一层黑给父节点，兄弟节点染成红色；
  - 如果父节点本身为黑，则父节点变成双重黑，问题没解决（矛盾转移了）
  - 如果父节点本身为红色，问题解决；

### 情况二
双重黑的兄弟为黑色 且 兄弟的两个儿子中有红色。

以 RR 型（双重黑的右兄弟为黑色，右兄弟的右儿子为红色）为例：

{% asset_img 4.png %}
解决方法：
- 抓着父节点“大左旋”；
- 把右兄弟染成父节点颜色；
- 父节点染成黑色，右兄第的右儿子染成黑色；
- 双重黑去掉一重黑；

其它类似情况：
- LL 型：左兄弟为黑色，左兄弟的左孩子为红色；
  - 抓着父节点“大右旋”；
  - 把左兄弟染成父节点颜色；
  - 父节点染成黑色，左兄第的左儿子染成黑色；
  - 双重黑去掉一重黑；
- RL 型：右兄弟为黑色，右兄弟的左孩子为红色；
  - 先抓着右兄弟进行 “小右旋”；
  - 右兄弟染红色，右兄弟的左孩子染成黑色；
  - 最后按照 RR 型处理；
- LR 型：左兄弟为黑色，左兄弟的右孩子为红色；
  - 先抓着左兄弟进行 “小左旋”；
  - 左兄弟染成红色，左兄弟的右孩子染成黑色；
  - 最后按照 LL 型处理；
- **注意：** LL 型的优先级高于 LR 型；RR 型的优先级高于 RL 型；

<!--对于AVL树，LR情况可以直接转换成LL类型，RL可以之间转换成RR类型，对于红黑树，也类似，但它需要加额外的染色，才能完成真正的转变-->
### 情况三
当双重黑兄弟为红色时。

以右兄弟为红色为例：

{% asset_img 5.png %}

解决办法：
- 抓着父节 “大左旋”；
- 右兄弟染成黑色，父节点染成红色；
- 问题转化为 “双重黑的右兄弟为黑色” 的情况；

类似情况：
- 当左兄弟为红色时：
  - 抓着父节点“大右旋”；
  - 左兄弟染黑，父节点染成红色；
  - 问题转化为 “双重黑的左兄弟为黑色” 的情况；

## 小结

红黑树调整 与 AVL 树调整：
- AVL 树调整：只进行 大左/右旋，小左/右旋操作；
- 红黑树调整：每次进行完 大左/右旋，小左/右旋后，需要进行一次颜色调整；

## 实现
```cpp
#include <stdio.h>
#include <malloc.h>

typedef struct Node {
    int val;
    int color;//0代表红色，1代表黑色，2代表双重黑
    struct Node *left, *right;
} Node;

Node NIL_NODE;
#define NIL (&NIL_NODE)
__attribute__((constructor))
void init_nil() {
    NIL->val = 0;
    NIL->color = 1;
    NIL->left = NIL->right = NIL;
}

Node *init(int val) {
    Node *p = (Node *)malloc(sizeof(Node));
    p->val = val;
    p->color = 0;
    p->left = p->right = NIL;
    return p;
}

void clear(Node *root) {
    if(root == NIL) return;
    clear(root->left);
    clear(root->right);
    free(root);
}

Node *left_rotate(Node *root) {
    Node *temp = root->right;
    root->right = temp->left;
    temp->left = root;
    return temp;
}

Node *right_rotate(Node *root) {
    Node *temp = root->left;
    root->left = temp->right;
    temp->right = root;
    return temp;
}

//如果没有用 NIL 节点取代 NULL，函数实现会较为复杂
Node *maintain_insert(Node *root) {
    if(root->left->color == 1 && root->right->color == 1) return root;
    if(root->left->color == 0 && root->right->color == 1) {
        //LR型冲突
        if(root->left->right->color == 0) {
            root->left = left_rotate(root->left);
        }

        //没有发生冲突
        if(root->left->left->color == 1) return root;

        //LL型冲突
        root = right_rotate(root);
    } else if(root->right->color == 0 && root->left->color == 1){
        //RL型冲突
        if(root->right->left->color == 0) {
            root->right = right_rotate(root->right);
        }

        //没有发生冲突
        if(root->right->right->color == 1) return root;

        //RR型冲突
        root = left_rotate(root);
    }
    //如果父节点和叔节点均为红色，直接跳到这块进行颜色调整（即使没有冲突，调整下也是没有问题的）
    //LL、LR、RR、RL型旋转后，需要进行颜色调整
    root->color = 0;
    root->left->color = root->right->color = 1;
    return root;
}

Node *insert_recursion(Node *root, int val) {
    if(root == NIL) return init(val);
    if(val == root->val) return root;
    if(val < root->val) {
        root->left = insert_recursion(root->left, val);
    } else {
        root->right = insert_recursion(root->right, val);
    }
    return maintain_insert(root);
}

Node *insert(Node *root, int val) {
    root = insert_recursion(root, val);
    //根节点染成黑色
    root->color = 1;
    return root;
}

Node *predecessor(Node *root) {
    Node *temp = root->left;
    while(temp->right != NIL) temp = temp->right;
    return temp;
}

Node *maintain_erase(Node *root) {
    if(root->left->color != 2 && root->right->color != 2) return root;
    //如果双重黑的兄弟节点为红色（情况三）
    if(root->left->color == 0) {
        root->left->color = 1;
        root->color = 0;
        root = right_rotate(root);
        //万幸 root->right 为红色，双重黑问题一定不会上浮
        root->right = maintain_erase(root->right);
        return root;
    }
    if(root->right->color == 0) {
        root->right->color = 1;
        root->color = 0;
        root = left_rotate(root);
        //万幸 root->right 为红色，双重黑问题一定不会上浮
        root->left = maintain_erase(root->left);
        return root;
    }

    //双重黑的兄弟节点为黑色（情况一）
    //考虑到NIL，必须要用 ！= 0 而不能用 == 1
    if((root->left->color == 1 && root->left->left->color != 0 && root->left->right->color != 0) ||
       (root->right->color == 1 && root->right->left->color != 0 && root->right->right->color != 0)) {
        root->color += 1;
        root->left->color -= 1;
        root->right->color -= 1;
        return root;
    }

    //（情况二）
    if(root->right->color == 1) {
        //RL 型
        if(root->right->right->color != 0) {
            root->right->color = 0;
            root->right = right_rotate(root->right);
            root->right->color = 1;
        }
        //RR 型
        root->left->color -= 1;
        root = left_rotate(root);
        root->color = root->left->color;
    } else {
        //LR 型
        if(root->left->left->color != 0) {
            root->left->color = 0;
            root->left = left_rotate(root->left);
            root->left->color = 1;
        }
        //LL型
        root->right->color -= 1;
        root = right_rotate(root);
        root->color = root->right->color;
    }
    root->left->color = root->right->color = 1;
    return root;
}

Node *erase_recursion(Node *root, int val) {
    if(root == NIL) return NIL;
    if(val < root->val) {
        root->left = erase_recursion(root->left, val);
    } else if(val > root->val) {
        root->right = erase_recursion(root->right, val);
    } else {
        if(root->left == NIL || root->right == NIL) {
            Node *temp = root->left ！= NIL ? root->left : root->right;
            temp->color += root->color;
            free(root);
            return temp;
        } else {
            Node *temp = predecessor(root);
            root->val = temp->val;
            root->left = erase_recursion(root->left, temp->val);
        }
    }
    return maintain_erase(root);
}

Node *erase(Node *root, int val) {
    root = erase_recursion(root, val);
    root->color = 1;
    return root;
}

int search(Node *root, int val) {
    if(root == NIL) return 0;
    if(root->val == val) return 1;
    if(val < root->val) 
    {
        return search(root->left, val);
    }
    return search(root->right, val);
}

void output(Node *root) {
    if(root == NIL) return;
    int left = root->left != NIL ? root->left->val : 0;
    int right = root->right != NIL ? root->right->val : 0;
    printf("(%d[%d], %d, %d)\n", root->val, root->color, left, right);
    
    output(root->left);
    output(root->right);
}

int main() {
    int op, val;
    Node *root = NIL;
    while(scanf("%d%d", &op, &val) != EOF) {
        switch(op) {
            case 1:
                root = insert(root, val);
                break; 
            case 2:
                root = erase(root, val);
                break;
        }
        output(root);
        printf("---------\n");
    }
    return 0;
}
```

