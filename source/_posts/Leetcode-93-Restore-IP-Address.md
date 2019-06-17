---
title: Leetcode 93 Restore IP Address
date: 2019-06-17 14:56:02
tags:
- Leetcode
- Python
- backtracking
categories:
- 算法
---

## 题目介绍

给定只包含数字的字符串，判断是否是合法的`IP`地址。

<!-- more -->

`Examples:`

```python 
Input: "25525511135"
Output: ["255.255.11.135", "255.255.111.35"]
```

## `Solution`

合法的`IP`地址应当是：

1. 只由四个部分组成
2. 每一部分从`0`到`255`
3. 除了`0`本身，不能包含前导`0`

可以使用回溯的方法来实现：

```python 
class Solution(object):
    def restoreIpAddresses(self, s):
        """
        :type s: str
        :rtype: List[str]
        """
        ans = []

        def helpfunc(i, res):
            # 已经有三个部分，剩余的部分大于3位数，直接返回
            if len(res) == 3 and len(s) - i > 3:
                return
            # 超出四个部分或者不足以凑够四个部分，直接返回
            if i > len(s) or len(res) > 4:
                return
            if len(res) == 4 and i == len(s):
                ans.append(".".join(res))
                return
            
            if i < len(s):
                # 单个数字
                helpfunc(i + 1, res + [s[i]])
                # 两位数
                if len(s) - i >= 2:
                    if int(s[i]) > 0:
                        helpfunc(i + 2, res + [s[i:i + 2]])
                # 三位数
                if len(s) - i >= 3:
                    if int(s[i]) != 0 and int(s[i:i + 3]) <= 255:
                        helpfunc(i + 3, res + [s[i:i + 3]])
        helpfunc(0, [])
        return ans
```

