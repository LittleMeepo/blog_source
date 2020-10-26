## 二叉树

#### Leetcode 94 [二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

> 给定一个二叉树，返回它的中序 遍历。
>
> 示例:
>
> ```c++
> 输入: [1,null,2,3]
>    1
>     \
>      2
>     /
>    3
> 
> 输出: [1,3,2]
> ```

递归解法

```c++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> stk;
        while (root != nullptr || !stk.empty()) {
            while (root != nullptr) {
                stk.push(root);
                root = root->left;
            }
            root = stk.top();
            stk.pop();
            res.push_back(root->val);
            root = root->right;
        }
        return res;
    }
};
```

#### Leetcode 95 [不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

> 给定一个整数 *n*，生成所有由 1 ... *n* 为节点所组成的 **二叉搜索树** 。

```c++
class Solution {
public:
    vector<TreeNode*> generateTrees(int n) {
        if(n==0){
            return {};
        }
        return generateTrees(1,n);
    }

    vector<TreeNode*> generateTrees(int start,int end)
    {
        if(start>end)
        {
            return {nullptr};
        }
        vector<TreeNode*> allTrees;
        for(int i=start;i<=end;i++)
        {
            vector<TreeNode*> leftTrees=generateTrees(start,i-1);
            vector<TreeNode*> rightTrees=generateTrees(i+1,end);

            for(auto &lTree:leftTrees)
            {
                for(auto &rTree:rightTrees)
                {
                    TreeNode* node=new TreeNode(i);
                    node->right=rTree;
                    node->left=lTree;
                    allTrees.push_back(node);
                }
            }
        }
        return allTrees;
    }
};
```

#### Leetcode 101 [对称二叉树](https://leetcode-cn.com/problems/symmetric-tree/)

> 给定一个二叉树，检查它是否是镜像对称的。

```c++
class Solution {
public:
    bool func(TreeNode *left,TreeNode *right){
        if(left==NULL&&right==NULL){
            return true;
        }
        if(left!=NULL&&right!=NULL&&left->val==right->val){
            return func(left->left,right->right)&&func(left->right,right->left);
        }
        return false;
    }
    bool isSymmetric(TreeNode* root) {
        if(root==NULL){
            return true;
        }
        return func(root->left,root->right);
    }
};
```

