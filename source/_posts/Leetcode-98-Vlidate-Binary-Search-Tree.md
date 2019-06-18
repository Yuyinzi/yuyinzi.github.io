---
title: Leetcode 98 Vlidate Binary Search Tree
date: 2019-06-18 19:41:50
tags:
- Python
- Leetcode
- DFS
categories:
- 算法
---

## 题目介绍

判断给定二叉树是否为一棵平衡二叉树。

<!-- more -->

`Examples:`

```python 
    2
   / \
  1   3

Input: [2,1,3]
Output: true
  
    5
   / \
  1   4
     / \
    3   6

Input: [5,1,4,null,null,3,6]
Output: false
```

## `Solution`

仅仅使用递归判断左子树是否平衡或者右子树是否平衡有可能出现右子树中的值比当前根节点还要小。利用先序遍历，平衡二叉树的结果是一个升序序列，因此，只要判断新增加的节点是否比前一节点大即可。

```python 
class Solution(object):
    def isValidBST(self, root):
        """
        :type root: TreeNode
        :rtype: bool
        """
        
        max_val = [float('-inf')]
        def helpfunc(root):
            if not root:
                return True
            if not helpfunc(root.left):
                return False
            if root.val <= max_val[0]:
                return False
            else:
                max_val[0] = root.val
            if not helpfunc(root.right):
                return False
            return True
        
        return helpfunc(root)
```

一种更优雅的方式，是采用递归，但是每次传入范围：

```python 
class Solution(object):
    def isValidBST(self, root):
        """
        :type root: TreeNode
        :rtype: bool
        """
        min_value, max_value = float('-inf'), float('inf')
        
        def helpfunc(root, min_value, max_value):
            if not root:
                return True
            if root.val <= min_value or root.val >= max_value:
                return False
                
            return helpfunc(root.left, min_value, root.val) and helpfunc(root.right, root.val, max_value)
        
        return helpfunc(root, min_value, max_value)
```

