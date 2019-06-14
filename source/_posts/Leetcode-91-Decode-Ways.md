---
title: Leetcode 91 Decode Ways
date: 2019-06-14 17:30:22
tags:
- Python
- Leetcode
- DP
categories:
- 算法
---

## 题目介绍

假设一条由`A-Z`组成的消息可以使用下面的方式进行编码：

```python 
'A' -> 1
'B' -> 2
...
'Z' -> 26
```

给定一个非空的只包含数字的字符串，判断有多少种解码的方式。

<!-- more -->

`Examples:`

```python 
Input: "12"
Output: 2
Explanation: It could be decoded as "AB" (1 2) or "L" (12).

Input: "226"
Output: 3
Explanation: It could be decoded as "BZ" (2 26), "VF" (22 6), or "BBF" (2 2 6).
```

## `Solution`

一开始使用递归去做，出现了超时。一看大神的解答，需要采用动态规划：

- 如果单独的数字不是0，那么可以尝试一位的解码
- 如果两位数字在10到26之间，可以尝试二位的解码

```python 
class Solution(object):
    def numDecodings(self, s):
        """
        :type s: str
        :rtype: int
        """
        dp = [0 for i in range(len(s)+1)]
        if len(s) and int(s[0]) == 0:
            return 0
        dp[1] = 1
        dp[0] = 1
        
        for i in range(2, len(dp)):
            first = int(s[i-1:i])
            second = int(s[i-2:i])
            
            if 1<= first <= 9:
                dp[i] += dp[i-1]
            if 10 <= second <= 26:
                dp[i] += dp[i-2]
        return dp[-1]
```

