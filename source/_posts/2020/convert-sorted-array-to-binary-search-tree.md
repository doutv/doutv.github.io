---
title: convert-sorted-array-to-binary-search-tree
date: 2020-07-04 08:50:00
tags:
    - Algorithm
    - LeetCode
    - Tree
    - Recursion
categories:
    - [Algorithm]
mathjax: true

---

[108. 将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)，这是一道有趣的树上递归题

1. 乍一看，这题不很简单嘛，直接递推就好了，`nums[mid]`的左边是左子树，右边是右子树，最终树的形状是一个人字形
2. 原来这个条件`本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。`每个节点都要满足，那应该要用上递归了。
3. 递归算法的设计：
   1. 其实蛮套路的，树上递归的题目代码都很短，基本就是处理左子树-->处理右子树-->合并左右子树
   2. 我是先用直觉写出递归，然后手推一下样例看看对不对，发现问题后再修改递归
   3. 每个$[L,R]$区间的处理方式与$[0,nums.size()-1]$是一样的，容易想到是以`mid=(l+r)/2`为分界，将当前树分成左右两棵子树，最后再考虑边界情况即可
4. 代码：
```cpp
class Solution {
public:
    TreeNode *build(vector<int> &nums,int l,int r)
    {
        if (l>r)
            return nullptr;
        int mid=(l+r)/2;
        TreeNode *cur=new TreeNode(nums[mid]);
        cur->left=build(nums,l,mid-1);
        cur->right=build(nums,mid+1,r);
        return cur;
    }
    TreeNode* sortedArrayToBST(vector<int>& nums) 
    {
        return build(nums,0,nums.size()-1);
    }
};
```
递归的代码看起来真的爽👍