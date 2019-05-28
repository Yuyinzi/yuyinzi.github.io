---
title: Leetcode 114 Flatten Binary Tree to Linked List
date: 2019-05-28 13:44:53
tags:
- Leetcode
- Python
categories:
- 算法
---

## 题目介绍

给定二叉树，将其原地变成一个链表。

<!--more-->

`Example`:

```python 
    1                       
   / \
  2   5     
 / \   \
3   4   6

   1
     \
      2
       \
        3
         \
          4
           \
            5
             \
              6
```

## `Solutions`

### 直观解法

发现链表的结果与先序遍历一致，因此先进行先序遍历，再根据遍历的结果构造链表。

```python 
# Definition for a binary tree node.
# class TreeNode(object):
#     def __init__(self, x):
#         self.val = x
#         self.left = None
#         self.right = None

class Solution(object):
    def flatten(self, root):
        """
        :type root: TreeNode
        :rtype: None Do not return anything, modify root in-place instead.
        """
        res = []
        def helpfunc(root):
            if not root:
                return
            res.append(root)
            helpfunc(root.left)
            helpfunc(root.right)
        
        helpfunc(root)
        for i in range(1, len(res)):
            res[i-1].right = res[i]
            res[i-1].left = None
```

### 逆序遍历

逆序遍历即按照[右子树，左子树，根]的方式来遍历，所以遍历结果为`6->5->4->3->2->1`，将遍历结果逐一进行类似于链表反转的操作：

```python 
class Solution(object):
    def __init__(self):
        self.prev = None
    def flatten(self, root):
        """
        :type root: TreeNode
        :rtype: None Do not return anything, modify root in-place instead.
        """
        if not root:
            return
        self.flatten(root.right)
        self.flatten(root.left)
        root.right = self.prev
        root.left = None
        self.prev = root
```

### 非递归解法

利用栈，先存右子树再存左子树，其实也是前序遍历的非递归写法。

```python 
class Solution(object):
    def flatten(self, root):
        """
        :type root: TreeNode
        :rtype: None Do not return anything, modify root in-place instead.
        """
        stack = []
        if root:
            stack.append(root)
            while stack:
                node = stack.pop()
                if node.right:
                    stack.append(node.right)
                if node.left:
                    stack.append(node.left)
                if stack:
                    node.right = stack[-1]
                    node.left = None
```

