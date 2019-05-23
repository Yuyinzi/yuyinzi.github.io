---
title: Leetcode 3 最长不重复字符子序列
date: 2019-05-10 15:49:04
tags:
- 算法
- Python
- Leetcode
- dict
categories:
- 算法
---

## 题目介绍

`Longest Substring Without Repeating Characters`:

给定字符串，求其最长不重复字符的子序列。

<!--more-->

`Examples`:

```python 
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

## `Solution`

考虑建立一个数组用来表示每个字符出现的最后一次位置，如果遍历到的字符所对应的值已经不是初始值(例如`-1`)，表示字符曾经出现过，因此应该从该字符上一次出现的下一个位置计算长度。

关键在于记录重复字符最后一次出现的位置，才能确定子序列的起点。

```python 
class Solution(object):
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        pos = [-1] * 256
        res, start = 0, -1
        for i in range(len(s)):
          	# 如果从没有出现过，则可以更新该字符的位置
            if pos[ord(s[i])] == -1:
                pos[ord(s[i])] = i
            else:
                # 如果已经出现过，在以前出现的最后一次位置与当前起点中找到新的起点
                # 保证该起点不小于当前起点
                start = max(pos[ord(s[i])], start)
                # 更新字符的位置
                pos[ord(s[i])] = i
            res = max(res, i - start)
        return res
```

需要注意的问题：

1. 初始值应当小于`0`，否则无法区分该值是否是位置
2. `start=max(pos[ord(s[i])], start)`，`start`应当不小于当前的`start`值，不然例如`abba`时，会出现计算错误