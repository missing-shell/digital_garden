## 标签
#dfs #bfs #二叉树 #链表 
## 解答方案
### 层序遍历
#bfs 
> 在遍历每层时记录当前层的**头节点**

- 时间复杂度：$O(N)$。我们需要遍历这棵树上所有的点，时间复杂度为 $O(N)$。
- 空间复杂度：$O(N)$。即队列的空间代价。
#### Code
```c++
class Solution {
public:
    Node* connect(Node* root) {
        queue<Node*> que;
        if (root != NULL) que.push(root);
        while (!que.empty()) {
            int size = que.size();
            vector<int> vec;
            Node* nodePre;
            Node* node;
            for (int i = 0; i < size; i++) {
                if (i == 0) {
                    nodePre = que.front(); // 取出一层的头结点
                    que.pop();
                    node = nodePre;
                } else {
                    node = que.front();
                    que.pop();
                    nodePre->next = node; // 本层前一个节点next指向本节点
                    nodePre = nodePre->next;
                }
                if (node->left) que.push(node->left);
                if (node->right) que.push(node->right);
            }
            nodePre->next = NULL; // 本层最后一个节点指向NULL
        }
        return root;
    }
};
```
### 层序遍历+链表
#bfs #链表 
- 使用每层建立的`next`指针
- 因为必须处理树上的所有节点，所以无法降低时间复杂度，但是可以尝试降低*空间复杂度*。
- 一边遍历当前层的节点，一边把下一层的节点连接起来。这样就无需存储下一层的节点，只需要拿到下一层链表的*头节点*。
- 时间复杂度：$O(n)$，其中 n 为二叉树的节点个数。
- 空间复杂度：$O(1)$。只用到若干额外变量。
---
#### 思路
- 从第一层开始（第一层只有一个 root 节点），每次循环：
- 遍历当前层的链表节点，通过节点的 left 和 right 得到下一层的节点。
- 把下一层的节点从左到右连接成一个链表。
- 拿到下一层链表的头节点，进入下一轮循环。
#### Code
```c++
class Solution {
public:
    Node *connect(Node *root) {
        Node *dummy = new Node();
        Node *cur = root;
        while (cur) {
            dummy->next = nullptr;
            Node *nxt = dummy; // 下一层的链表
            while (cur) { // 遍历当前层的链表
                if (cur->left) {
                    nxt->next = cur->left; // 下一层的相邻节点连起来
                    nxt = cur->left;
                }
                if (cur->right) {
                    nxt->next = cur->right; // 下一层的相邻节点连起来
                    nxt = cur->right;
                }
                cur = cur->next; // 当前层链表的下一个节点
            }
            cur = dummy->next; // 下一层链表的头节点
        }
        delete dummy;
        return root;
    }
};
```
### 建立`list`
- 首先通过递归建立以最左边节点为首的空数组`pre`。
- 使用`dfs`求解，参数分别为当前节点`node`和深度`depth`。
- 当`depth`等于数组的长度，说明`node`是这一层最左边的节点
- 递归边界：`node==nullptr`。
- 递归入口：`dfs(root,0)`。
#### Code
```c++
class Solution {
    vector<Node *>pre;
public:
    Node* connect(Node* root) {
        dfs(root,0);
        return root;
    }
    void dfs(Node *node,int depth)
    {
        if(node==nullptr)
        {
            return;
        }
        if(depth==pre.size())
        {
            pre.push_back(node);
        }else{
            pre[depth]->next=node;
            pre[depth]=node;
        }
        dfs(node->left,depth+1);
        dfs(node->right,depth+1);
    }
};
```