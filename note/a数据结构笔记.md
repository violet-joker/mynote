## 二叉搜索/排序树(Binary Search/Sort Tree)

### 要点：

+ 左子树 < 根节点 < 右子树

+ 删除：要删除的节点左右子树都存在时，与前驱/后继节点交换数值，
再继续删除操作。其他删除情况简单，此不赘述。(画个图一目了然)

+ 找前驱：左子树的最右叶子节点

+ 找后继：右子树的最左叶子节点


### 代码实现

```c++
#include <iostream>
#include <algorithm>

using namespace std;
const int N = 1e3 + 10;
int idx, t;

struct Node {
    int key;
    int l, r;
} tr[N];

int get_node(int key) {
    tr[++idx].key = key;
    return idx;
}

void add(int key, int& p) {
    if (!p) {
        p = get_node(key);
    } else if (key < tr[p].key) {
        add(key, tr[p].l);
    } else {
        add(key, tr[p].r);
    }
}

void remove_by_front(int key, int& p) {
    if (!p) {
        cout << "key " << key << " not exit" << endl;
    } else if (key < tr[p].key) {
        remove_by_front(key, tr[p].l);
    } else if (key > tr[p].key) {
        remove_by_front(key, tr[p].r);
    } else {
        if (!tr[p].l && !tr[p].r) {
            // 无左右子树，直接删除叶子节点
            p = 0;
        } else if (!tr[p].l) {
            // 只有右子树，右子树替换删除节点
            p = tr[p].r;
        } else if (!tr[p].r) {
            // 只有左子树
            p = tr[p].l;
        } else {
            // 在左子树找前驱(最大的小于key的节点)
            int q = tr[p].l;
            while (tr[q].r) q = tr[q].r;
            swap(tr[p].key, tr[q].key);
            remove_by_front(key, tr[p].l);
        }
    }
}

void remove_by_back(int key, int& p) {
    if (!p) {
        cout << "key " << key << " not exit" << endl;
    } else if (key < tr[p].key) {
        remove_by_back(key, tr[p].l);
    } else if (key > tr[p].key) {
        remove_by_back(key, tr[p].r);
    } else {
        if (!tr[p].l && !tr[p].r) {
            p = 0;
        } else if (!tr[p].l) {
            p = tr[p].r;
        } else if (!tr[p].r) {
            p = tr[p].l;
        } else {
            // 在右子树找后继(最小的大于key的节点)
            int q = tr[p].l;
            while (tr[q].r) q = tr[q].r;
            swap(tr[p].key, tr[q].key);
            remove_by_back(key, tr[p].l);
        }
    }
}

void dfs(int& p) {
    if (!p) return;
    dfs(tr[p].l);
    cout << tr[p].key << " ";
    dfs(tr[p].r);
}

int main() {
    int f[N], n = 20;
    for (int i = 0; i < n; i++) f[i] = i;
    random_shuffle(f, f+n);
    for (int i = 0; i < n; i++) add(f[i], t);
    dfs(t);
    cout << endl;
    for (int i = 0; i < n; i++) if (i & 1) remove_by_front(i, t);
    dfs(t);
    cout << endl;
}

```

## 平衡二叉树(AVL)

### 要点：

+ 左 < 根 < 右

+ 左右子树高度之差不超过1，否则旋转调整平衡

+ LL, RR, LR, RL旋转方式画图一目了然

+ 删除策略与BST一致，多一步判断是否旋转调整平衡

+ 插入和删除的都需要维护树高，可利用递归的特性，pushup回溯时维护

```c++
/*
注意！！！只有子树出现高度差才是LR或RL型旋转！可能出现父节点平衡因子
为2，子节点平衡因子为0。
例如下图：
                 7
          /             \
        3                9
      /    \           /
    1       4         8
  /  \       \
0     2       5
*/
```

### 代码实现 

```c++
#include <iostream>
#include <algorithm>

using namespace std;
const int N = 1e3 + 10;
int idx, t;
struct Node {
    int key, h;
    int l, r;
} tr[N];

int get_node(int key) {
    tr[++idx].key = key; 
    tr[idx].h = 1;
    return idx;
}

// 维护树高
void pushup(int& p) {
    // 注意p为0时直接返回，否则会更新tr[0].h的值
    // 而tr[0]实际上作为哨兵便于操作，不能更改
    if (!p) return;
    tr[p].h = max(tr[tr[p].l].h, tr[tr[p].r].h) + 1;
}

// 右旋
void zig(int& p) {
    int q = tr[p].l;
    tr[p].l = tr[q].r;
    tr[q].r = p;
    p = q;
    pushup(tr[p].r);
    pushup(p);
}

// 左旋
void zag(int& p) {
    int q = tr[p].r;
    tr[p].r = tr[q].l;
    tr[q].l = p;
    p = q;
    pushup(tr[p].l);
    pushup(p);
}

// 检查&调整 平衡
void check(int& p) {
    // L*
    if (tr[tr[p].l].h - tr[tr[p].r].h == 2) {
        int& q = tr[p].l;
        if (tr[tr[q].l].h - tr[tr[q].r].h == -1) {
            // LR型，先左旋，再右旋
            zag(q), zig(p);
        } else {
            // LL型，右旋
            zig(p);
        }
    }
    // R*
    if (tr[tr[p].l].h - tr[tr[p].r].h == -2) {
        int& q = tr[p].r;
        if (tr[tr[q].l].h - tr[tr[q].r].h == 1) {
            // RL型，先右旋，再左旋
            zig(q), zag(p);
        } else {
            // RR型，左旋
            zag(p);
        }
    }
}

void insert(int key, int& p) {
    if (!p)
        p = get_node(key);
    else if (key < tr[p].key)
        insert(key, tr[p].l);
    else
        insert(key, tr[p].r);
    pushup(p);
    check(p);
}

void remove(int key, int& p) {
    if (!p)
        cout << "key " << key << " not exit!" << endl;
    else if (key < tr[p].key) 
        remove(key, tr[p].l);
    else if (key > tr[p].key)
        remove(key, tr[p].r);
    else {
        // 左右子树均不存在，直接删除叶子节点
        if (!tr[p].l && !tr[p].r)
            p = 0;
        else if (!tr[p].l) // 只有右子树
            p = tr[p].r;
        else if (!tr[p].r) // 只有左子树
            p = tr[p].l;
        else {
            // 左右子树均存在，和BST删除策略一致，这里采取替换前驱节点
            int q = tr[p].l;
            while (tr[q].r) q = tr[q].r;
            swap(tr[p].key, tr[q].key);
            remove(key, tr[p].l);
        }
    }
    pushup(p);
    check(p);
}

// 中序遍历检验
void dfs(int& p) {
    if (!p) return;
    dfs(tr[p].l);
    cout << tr[p].key << " ";
    dfs(tr[p].r);
}

int main() {
    // 生成随机测试数据
    int f[N], n = 20;
    for (int i = 0; i < n; i++) f[i] = i;
    random_shuffle(f, f+n);
    for (int i = 0; i < n; i++) {
        cout << f[i] << " ";
        insert(f[i], t);
    }
    cout << endl;

    dfs(t);
    cout << endl;

    // 简单测试一下删除操作
    for (int i = 0; i < n; i++) {
        if (i & 1) remove(i, t);
        else cout << i << " ";
    }
    cout << endl;
    dfs(t);
    cout << endl;
}

```

## 红黑树
