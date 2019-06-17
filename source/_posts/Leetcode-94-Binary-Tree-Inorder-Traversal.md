---
title: Leetcode 94 Binary Tree Inorder Traversal
date: 2019-06-17 16:12:23
tags:
- Python
- Leetcode
- Tree
categories:
- 算法
---

## 题目介绍

给定二叉树，返回其中序遍历。使用递归和迭代方法。

<!-- more -->

`Example:`

```python 
Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [1,3,2]
```

## `Solution`

### 递归法

使用递归比较简单：

```python 
class Solution(object):
    def inorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res = []
        def helpfunc(root):
            if not root:
                return
            helpfunc(root.left)
            res.append(root.val)
            helpfunc(root.right)
        helpfunc(root)
        return res
```

### 迭代法

中序遍历迭代需要一个栈，思想为：

1. 不断遍历左子树，将节点压栈，直到没有左节点
2. 将该节点弹出栈，如果有右子树，则重复1

```python 
class Solution(object):
    def inorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        stack, res = [], []
        if not root:
            return res

        node = root
        while stack or node:
            while node:
                stack.append(node)
                node = node.left
            # 当前node节点为空，前一个节点没有左子树
            node = stack.pop()
            res.append(node.val)
            # 开始查看其右子树
            node = node.right
        return res
```

## 先序遍历

[Leetcode 144 Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)

### 递归

首先仍然是相对简单的递归方法：

```python 
class Solution(object):
    def preorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res = []
        
        def helpfunc(root):
            if not root:
                return
            res.append(root.val)
            helpfunc(root.left)
            helpfunc(root.right)
            
        helpfunc(root)
        return res
```

### 非递归

非递归方法有两种：

**方法一**

1. 遍历到节点，加入结果中
2. 先压栈其右节点
3. 再压栈其左节点

这样可以达到遍历根节点->左节点->右节点的效果：

```python 
class Solution(object):
    def preorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res, stack = [], []
        if not root:
            return res
        stack.append(root)
        while stack:
            node = stack.pop()
            res.append(node.val)
            if node.right:
                stack.append(node.right)
            if node.left:
                stack.append(node.left)
        return res
```

**方法二**

有了中序遍历的例子，可以采用类似的思路，仍然不断寻找左节点，只是每当寻找到就添加至结果列表。

```python 
class Solution(object):
    def preorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res, stack = [], []
        if not root:
            return res
        node = root
        while stack or node:
            while node:
                res.append(node.val)
                stack.append(node)
                node = node.left
            # 前一个节点没有左节点，查找其右节点
            node = stack.pop()
            node = node.right
        return res
```

## 后序遍历

[Leetcode 145 Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/)

### 递归

递归解法：

```python 
class Solution(object):
    def postorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res = []
        
        def helpfunc(root):
            if not root:
                return
            helpfunc(root.left)
            helpfunc(root.right)
            res.append(root.val)
            
        helpfunc(root)
        return res
```

### 非递归

非递归相对来说麻烦一点，但是发现规律是，根节点->右子树->左子树，再将遍历结果逆序就能得到后序遍历的结果。于是又可以稍稍改变代码：

```python 
class Solution(object):
    def postorderTraversal(self, root):
        """
        :type root: TreeNode
        :rtype: List[int]
        """
        res, stack = [], []
        if not root:
            return res
        node = root
        while stack or node:
            while node:
                res.append(node.val)
                stack.append(node)
                node = node.right
            node = stack.pop()
            node = node.left
            
        return res[::-1]
```



 