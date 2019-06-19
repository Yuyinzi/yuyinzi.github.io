---
title: Leetcode 51 N-Queens
date: 2019-06-19 14:19:21
tags:
- Python
- Leetcode
- DFS
categories:
- 算法
---

## 题目介绍

求`N`皇后问题的所有解法。

<!-- more -->

`Examples`：

```python 
Input: 4
Output: [
 [".Q..",  // Solution 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // Solution 2
  "Q...",
  "...Q",
  ".Q.."]
]
```

## `Solution`

`N`皇后问题是一个标准的`DFS`问题，任意两个皇后不能出现在同一行，同一列或者对角线上(`45°`和`135°`)。针对`Python`，还需要考虑可变变量的问题。

对于同一行或者同一列，因为棋盘是`N*N`，所以只要使用一个`visit`数组就可以标识是否出现过。

下面是一个比较普通的方法：

```python 
class Solution(object):
    def solveNQueens(self, n):
        """
        :type n: int
        :rtype: List[List[str]]
        """
        visit = [0 for i in range(n)]
        ans = []

        def helpfunc(i, res):
            if i == n:
                ans.append(res)
                return
            for k in range(n):
               # 检查行列
                if not visit[k]:
                    p, q, m = i, k, k
                    # 检查对角线
                    while p-1 >= 0:
                        if q-1 >= 0 and res[p-1][q-1] == 'Q':
                            break
                        if m+1 <= n-1 and res[p-1][m+1] == 'Q':
                            break
                        p -= 1
                        q -= 1
                        m += 1
                    else:
                        temp = ['.'] * n
                        temp[k] = 'Q'
                        visit[k] = 1
                        helpfunc(i+1, res+[''.join(temp)])
                        visit[k] = 0

        helpfunc(0, [])
        return ans
```

当然，也可以使用一个数组来表示对角线是否被使用。假设元素坐标为`[i,j]`，则对角线的值为`i+j`或者`i-j`。