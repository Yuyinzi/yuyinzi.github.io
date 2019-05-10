---
title: Leetcode 5 求最长回文子序列
date: 2019-05-10 17:37:41
tags:
- 算法
- Python
- Leetcode
categories:
- 算法
---

## 题目介绍

给定一个字符串`s`，求最长的回文子序列。

<!--more-->

`Examples`:

```python 
Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.

Input: "cbbd"
Output: "bb"
```

## `Solution`

### 解法一：`DP`

其中`dp[j][i]`表示以`j`为起点，`i`为终点(`[j,i]`)的最长回文子序列。那么，一个序列是回文子序列应当满足：

1. `s[i]==s[j]`
2. 如果`i-j<=2`，此时有(`a`,`aa`,`aba`)三种情况，都为回文子序列。
3. 如果`i-j>2`，应当考虑`dp[j+1][i-1]`是否是回文子序列

```python 
class Solution(object):
    def longestPalindrome(self, s):
        """
        :type s: str
        :rtype: str
        """

        dp = [[0 for i in range(len(s))] for j in range(len(s))]
        res = ""
        for i in range(len(s)):
            for j in range(i + 1):
                if s[i] == s[j] and (i - j <= 2 or dp[j + 1][i - 1]):
                    dp[j][i] = i - j + 1
                    if i - j + 1 > len(res):
                        res = s[j:i + 1]
        return res
```

但是这种方法空间复杂度比较高。

### 解法二：尝试构造回文子序列

这是`Leetcode`上大神的解法：

```python 
def longestPalindrome(self, s):
    res = ""
    for i in xrange(len(s)):
        # odd case, like "aba"
        tmp = self.helper(s, i, i)
        if len(tmp) > len(res):
            res = tmp
        # even case, like "abba"
        tmp = self.helper(s, i, i+1)
        if len(tmp) > len(res):
            res = tmp
    return res

def helper(self, s, l, r):
    while l >= 0 and r < len(s) and s[l] == s[r]:
        l -= 1; r += 1
    return s[l+1:r]
```

尝试以每一个位置为中心，往两边遍历，判断是否能生成回文子序列。需要注意的是，要考虑`aba`和`abba`两种情况。

