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

## 延伸

[Leetcode 96 Unique Binary Search Trees](https://leetcode.com/problems/unique-binary-search-trees/)

这道题目只要求求出总共有多少种可能。根据上面的思路，可以得出：

`dp[start][end] = dp[start][k-1] * dp[k+1][end], for k in [start, end]`

很自然的会想到使用二维的`dp`数组，但是需要注意的是：

1. 当`k==start`或者`k==end`时，会导致越界，此时应当单独赋值。例如：

   当`nums=1,2`时，此时`dp[1][2]= dp[1][0] * dp[2,2] + dp[1][1]*dp[3][2]`，因为`dp[1][2]`和`dp[3][2]`可以代表空节点，所以应当有意义。

2. 当`nums=1,2,3`时，求`dp[1][3]`需要依赖`dp[2][3]`，因此需要从左至右，从下至上进行计算。

```python 
class Solution(object):
    def numTrees(self, n):
        """
        :type n: int
        :rtype: int
        """
        if n < 1:
            return 1
        dp = [[0 for i in range(n + 1)] for j in range(n + 1)]

        for i in range(n, 0, -1):
            for j in range(1, n + 1):
                if i == j:
                    dp[i][j] = 1
                else:
                    for k in range(i, j+1):
                        if k == i:
                            dp[i][j] += dp[k+1][j]
                        elif k == j:
                            dp[i][j] += dp[i][k-1]
                        else:
                            dp[i][j] += dp[i][k-1] * dp[k + 1][j]

        return dp[1][n]
```

然而，可以使用一维数组解决，因为只要考虑个数，不用考虑特定的下标，即`dp[2][3]`和`dp[1][2]`的值其实没有本质区别。

假设`G[n]=F(1,n)+F(2,n)+...+F(n,n)`，`F(i,n)`代表以`i`为根有多少种树。

同时，`F(i,n)=G(i-1)*G(n-i)`，即以`i`为根，左边树的个数为以`i-1`为根的树种类，右边树的个数为`[i+1, n]`,等价于`G(n-i)`。

特别地，当`n==0`或`n==1`时，只有1种树，这个作为初始条件。

以`G[2]`为例，`G[2]=F(1,2)+F(2,2)=G[0]*G[1]+G[1]*G[0]`。

```python 
class Solution(object):
    def numTrees(self, n):
        """
        :type n: int
        :rtype: int
        """
        G = [0 for i in range(n+1)]
        
        G[0] = G[1] = 1
        
        for i in range(2, n+1):
            for j in range(1, i+1):
                G[i] += G[i-j] * G[j-1]
        
        return G[n]
```



