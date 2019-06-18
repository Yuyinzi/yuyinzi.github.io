---
title: Leetcode 95 Unique Binary Search Trees II
date: 2019-06-18 15:22:16
tags:
- Python
- Leetcode
- DP
categories:
- 算法
---

## 题目介绍

给定正整数`n`，利用`1`到`n`构造所有可能的二叉树，并返回。

<!-- more -->

`Example`:

```python 
Input: 3
Output:
[
  [1,null,3,2],
  [3,2,null,1],
  [3,1,null,null,2],
  [2,1,3],
  [1,null,2,null,3]
]
Explanation:
The above output corresponds to the 5 unique BST's shown below:

   1         3     3      2      1
    \       /     /      / \      \
     3     2     1      1   3      2
    /     /       \                 \
   2     1         2                 3
```

## `Solution`

一开始想用回溯去做，但是发现`root`所做的更改将会影响到前面的结果。后面发现应该使用动态规划(为啥哪都可以用上动态规划= =)：

取每一个节点`i`，那么它的左子树由`[1,i-1]`组成，右子树由`[i+1, n]`组成。左右子树的构成方法又可以分解为更小的问题。以此类推。

```python 
class Solution(object):
    def generateTrees(self, n):
        """
        :type n: int
        :rtype: List[TreeNode]
        """ 
        if n == 0:
            return []
        def helpfunc(start, end):
            temp = []
            # 如果右边界比左边界小，不存在子树，返回[None]用以遍历
            if start > end:
                return [None]
            # 如果左右边界相等，返回节点自身，例如求2，3的组成，当节点为2时，其右子树应当为[3]
            if start == end:
                temp.append(TreeNode(start))
                return temp
            for i in range(start, end + 1):
                ltrees = helpfunc(start, i - 1)
                rtrees = helpfunc(i + 1, end)

                for ltree in ltrees:
                    for rtree in rtrees:
                        # 每次都能生成不同的根节点
                        node = TreeNode(i)
                        node.left = ltree
                        node.right = rtree
                        temp.append(node)
            return temp

        ans = helpfunc(1, n)
        return ans
```

