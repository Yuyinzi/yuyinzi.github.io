---
title: Leetcode 79 Word Search
date: 2019-06-12 17:49:47
tags:
- Python
- Leetcode
- backtracking
categories:
- 算法
---

## 题目介绍

给定一个二维数组以及单词，判断单词是否存在于该数组中。要求按顺序，且只能相邻(水平或垂直方向)查找。每个字母只能使用一次。

<!--more-->

`Examples:`

```python 
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false
```

## `Solution`

使用回溯，当查询到某个字母相等时，再以该坐标出发去寻找是否相邻位置有下一个字母。而由于每个字母只能使用一次，因此，需要对已经遍历过的字母做标记。

```python 
class Solution(object):
    def exist(self, board, word):
        """
        :type board: List[List[str]]
        :type word: str
        :rtype: bool
        """
        if not board or not word:
            return False
        m, n = len(board), len(board[0])
        
        def helpfunc(i, j, k):
            # 优先判断k，否则只有一个字母的单词会返回False 
            if k == len(word):
                return True
            if i<0 or i>=m or j<0 or j>=n:
                return False
            # 标记已经使用的字母
            if board[i][j] == word[k]:
                board[i][j] = 0

                if helpfunc(i, j-1, k + 1) or helpfunc(i, j+1, k+1) or helpfunc(i-1, j, k+1) or helpfunc(i+1, j, k+1):
                    return True
                # 恢复标记
                board[i][j] = word[k]
        for i in range(m):
            for j in range(n):
                if helpfunc(i, j, 0):
                    return True
        return False
```

