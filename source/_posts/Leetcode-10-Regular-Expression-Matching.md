---
title: Leetcode 10 Regular Expression Matching
date: 2019-05-17 15:29:59
tags:
- Leetcode
- Python
---

## 题目介绍

给定字符串`s`，以及正则匹配模式`p`，判断是否能匹配成功。其中`.`表示可以匹配任意一个字符，`*`表示可以匹配0个或者任意多个前缀字符。

<!-- more -->

`Examples：`

```shell 
Input:
s = "aa"
p = "a"
Output: false
Input:
s = "aa"
p = "a*"
Output: true
Input:
s = "ab"
p = ".*"
Output: true
Input:
s = "aab"
p = "c*a*b"
Output: true
Input:
s = "mississippi"
p = "mis*is*p*."
Output: false
```

## `Solution`

直观的可以采用递归来做：

1. 如果`p`第二个字符是`*`，表示可以匹配0个或者多个：
   1. 从匹配0个开始尝试，即匹配`s`与`p[2:]`(刚开始写一直出错，因为没有从0开始尝试，写成了`if not s: return self.isMatch(s, p[2:])`，当出现`aaa`与`a*a`时，就出错了)
   2. 如果匹配0个失败，尝试匹配多个，即匹配`s[1:]`与`p`，前提条件是`s[0]==p[0]`或者`p[0]`为`.`
2. 如果`p`第二个字符并不是`*`，那就老老实实一个一个匹配：
   1. 如果`s[0]==p[0]`或者`p[0]=='.'`，那么继续往下匹配，即匹配`s[1:]`与`p[1:]`
   2. 如果不相等，即不匹配，返回`False`
3. 如果`p`或者`s`同时为空，匹配结束

```python 
class Solution(object):
    def isMatch(self, s, p):
        """
        :type s: str
        :type p: str
        :rtype: bool
        """
        if not s and not p:
            return True
        
        if len(p) > 1 and p[1] == '*':
            if self.isMatch(s, p[2:]):
                return True
            if len(s) > 0 and (p[0] == s[0] or p[0] == '.'):
                return self.isMatch(s[1:], p)
            else:
                return False
        elif (len(s) > 0 and len(p) > 0) and (p[0] == s[0] or p[0] == '.'):
            return self.isMatch(s[1:], p[1:])
        else:
            return False
```

还有一个方法是动态规划，下次再补充~

