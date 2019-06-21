---
title: Leetcode 120 Triangle
date: 2019-06-21 16:08:58
tags:
- Python
- Leetcode
- DP
categories:
- 算法
---

## 题目介绍

给定一个三角形状的数组，寻找从顶部到底部上所有值之和最小的路径。只能移动至下一层的相邻节点。

<!-- more -->

`Example:`

```python 
[
     [2],
    [3,4],
   [6,5,7],
  [4,1,8,3]
]
The minimum path sum from top to bottom is 11 (i.e., 2 + 3 + 5 + 1 = 11)
```

能否只使用`o(n)`空间实现，`n`是三角形的行数。

## `Solution`

针对底部的每一个节点，有很多路径可以到达，寻求一个最优解，因此这是一个动态规划问题。每个节点可以由上一层与其同下标或者下标小于1的节点向下而来。

因此可以采用一个一维数组，表示最后一行(也即是三角形的行数)中到每一个节点的路径之和。由于每个节点只受其下标或者比其小1的下标的节点影响，因此针对每一行，应当从后往前遍历，不断更改每一层的节点路径之和。

```python 
class Solution(object):
    def minimumTotal(self, triangle):
        """
        :type triangle: List[List[int]]
        :rtype: int
        """
        dp = [float('inf') for i in range(len(triangle))]
        if triangle:
            dp[0] = triangle[0][0]
            # 遍历每一层
            for i in range(1, len(triangle)):
                # 从后往前遍历每一层的节点
                for j in range(len(triangle[i])-1, -1, -1):
                    dp[j] = min(triangle[i][j] + dp[j], triangle[i][j] + dp[max(j - 1, 0)])
        return min(dp)
```

