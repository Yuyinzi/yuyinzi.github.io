---
title: Leetcode 235 Lowest Common Ancestor of a Binary Search Tree
date: 2019-05-28 15:56:04
tags:
- Leetcode
- Python
categories:
- 算法
---

## 题目介绍

在二叉查找树中，查找给定两个不同节点的最近的公共父节点(节点可以当做自己的父节点)。

<!-- more -->

`Examples`:

![binarysearchtree_improved](/assets/binarysearchtree_improved.png)

```python 
Input: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
Output: 6
Explanation: The LCA of nodes 2 and 8 is 6.

Input: root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
Output: 2
Explanation: The LCA of nodes 2 and 4 is 2, since a node can be a descendant of itself according to the LCA definition.
```

## `Solutions`

因为是二叉查找树，所以特点有任意节点都大于其左子树，小于其右子树。两个节点的分布可以有两种情况：

1. 两个节点都在左子树或者右子树
2. 两个节点分别为在某个节点的左子树和右子树上

对于情况一，必有一个节点为公共父节点，则公共父节点为最先找到的节点。对于情况二，公共父节点为当前父节点。

```python 
class Solution(object):
    def lowestCommonAncestor(self, root, p, q):
        """
        :type root: TreeNode
        :type p: TreeNode
        :type q: TreeNode
        :rtype: TreeNode
        """
        if p.val < root.val and q.val < root.val:
            return self.lowestCommonAncestor(root.left, p, q)
        elif p.val > root.val and q.val > root.val:
            return self.lowestCommonAncestor(root.right, p, q)
        else:
            return root
```

## 延伸

对应的还有[Leetcode 236 Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)。

这题不同的地方在于，并不是二叉查找树，所以不能根据值的大小来进行判断。但是仍然可以利用递归的思想，每次查找都返回公共父节点：

- 分别在当前节点的左子树和右子树中进行查找
  - 如果当前节点的左右子树都有返回值，说明它们都查找到自己作为公共父节点，因此公共父节点应该为当前节点
  - 如果只有一个子树有返回值，说明两个节点都在一个子树中

```python 
    def lowestCommonAncestor(self, root, p, q):
        """
        :type root: TreeNode
        :type p: TreeNode
        :type q: TreeNode
        :rtype: TreeNode
        """
        if not root or p.val == root.val or q.val == root.val:
            return root
        if root:
            left_res = self.lowestCommonAncestor(root.left, p, q)
            right_res = self.lowestCommonAncestor(root.right, p, q)
            
            if left_res and right_res:
                return root
            return left_res or right_res
```

