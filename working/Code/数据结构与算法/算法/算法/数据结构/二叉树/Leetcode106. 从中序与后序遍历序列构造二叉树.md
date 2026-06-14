## 思路
### 分析
- 中序：左中右
- 后序：左右中
- 后序最后一个节点就是**中间节点**
- 通过这个中间节点可以将中序数组分割为*左中序*和*右中序*数组
- 同样的，后序中的左子树部分和中序中的左子树部分的数目相同，实现后序分割为*左后序*和*右后序*数组。
- 所以当前节点的左子树由*左中序*和*左后序*共同决定
> 可由**中序数组**与（`前序`或`后序`）数组组合而成确定唯一的二叉树
### 代码思路
- 第一步：如果数组大小为零的话，说明是空节点了。
- 第二步：如果不为空，那么取后序数组最后一个元素作为节点元素。
- *优化*：如果此时的后序数组大小为1（节点既是中间节点又是叶子节点）说明遍历结束，直接返回当前`root`节点
- 第三步：找到后序数组最后一个元素在中序数组的位置，作为切割点
- 第四步：切割中序数组，切成中序左数组和中序右数组 （顺序别搞反了，一定是先切中序数组）
- 第五步：切割后序数组，切成后序左数组和后序右数组
- 第六步：递归处理左区间和右区间
```c++
TreeNode* traversal (vector<int>& inorder, vector<int>& postorder) {

    // 第一步
    if (postorder.size() == 0) return NULL;

    // 第二步：后序遍历数组最后一个元素，就是当前的中间节点
    int rootValue = postorder[postorder.size() - 1];
    TreeNode* root = new TreeNode(rootValue);

    // 叶子节点
    if (postorder.size() == 1) return root;

    // 第三步：找切割点
    int delimiterIndex;
    for (delimiterIndex = 0; delimiterIndex < inorder.size(); delimiterIndex++) {
        if (inorder[delimiterIndex] == rootValue) break;
    }

    // 第四步：切割中序数组，得到 中序左数组和中序右数组
    // 第五步：切割后序数组，得到 后序左数组和后序右数组

    // 第六步
    root->left = traversal(中序左数组, 后序左数组);
    root->right = traversal(中序右数组, 后序右数组);

    return root;
}
```
### 数组切割
> 统一使用左闭右开原则

- 通过`rootValue`在中序数组中的索引，可将中序数组分为左右子树
- 左子树的中序数组和后续数组的大小应当一致，可将后序数组分为左右子树
- 后序数组在分割前需重新设置一下数组大小`postorder.resize(postorder.size() - 1);`
## Code
### 递归
- 时间复杂度：$O(n)$，其中 n 为二叉树的节点个数。
- 空间复杂度：$O(1)$。只用到若干额外变量。
```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left),
 * right(right) {}
 * };
 */
class Solution {
public:
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        if (postorder.size() == 0)
            return nullptr;
        int rootValue=postorder[postorder.size()-1];
        TreeNode* root = new TreeNode(rootValue);

        if (postorder.size() == 1)
            return root;

        int index = 0;
        for (index = 0; index < inorder.size(); index++) {
            if (inorder[index] == root->val) {
                break;
            }
        }
        vector<int> left_inorder(inorder.begin(), inorder.begin() + index);
        vector<int> right_inorder(inorder.begin() + index + 1, inorder.end());

        postorder.resize(postorder.size() - 1);
        vector<int> left_postorder(postorder.begin(), postorder.begin() + index);
        vector<int> right_postorder(postorder.begin() + index, postorder.end());

        root->left = buildTree(left_inorder, left_postorder);
        root->right = buildTree(right_inorder, right_postorder);

        return root;
    }
};
```
