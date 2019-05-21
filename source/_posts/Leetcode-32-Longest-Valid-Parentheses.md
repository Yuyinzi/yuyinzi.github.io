---
title: Leetcode 32 Longest Valid Parentheses
date: 2019-05-21 23:26:45
tags:
- Leetcode
- Python
categories:
- 算法
---

## 题目介绍

给定由`'('`和`')'`组成的字符串，判断最长的合法的括号长度。

<!--more-->

`Examples`：

```python 
Input: "(()"
Output: 2
Explanation: The longest valid parentheses substring is "()"

Input: ")()())"
Output: 4
Explanation: The longest valid parentheses substring is "()()"
```

## `Solutions`

### 使用`stack`

习惯性碰见括号的匹配使用栈，但是如果出现了`()(()`这种情况，不好判断。因此，每次可以将半括号的下标存入栈中，只有未匹配的半括号才会留在栈中，可以视为分隔点，然后计算每一段的长度，取最大值即可。

- 出现`)`，如果栈顶存放的是`(`，那么可以弹出
  - 如果栈顶存放的是`)`，不能匹配，那么继续将其下标压入栈中
  - 如果栈为空，不能匹配，也需要将其下标压入栈中
- 出现`(`，直接将下标压入栈中

最后还需要计算最后一个节点与0之间的长度大小。比如`())`这种情况，最长的值出现在分割点前。

```python 
class Solution(object):
    def longestValidParentheses(self, s):
        """
        :type s: str
        :rtype: int
        """
        stack = []
        res = 0
        for i in range(len(s)):
            if s[i] == ')':
                if not stack:
                    stack.append(i)
                elif s[stack[-1]] == '(':
                    stack.pop()
                else:
                    stack.append(i)
            else:
                stack.append(i)
        if not stack:
            return len(s)
        end = len(s)
        while stack:
            start = stack.pop()
            res = max(end-start-1, res)
            end = start
        return max(res, end)
```

### `DP`

这个方法理解了很久，思路是设置一个`dp`数组，其中`dp[i]`表示以`i`结尾的最长合法括号的长度，有以下几种情况：

1. 如果`s[i]=='('`，那么`dp[i]=0`
2. 如果`s[i]==')'`：
   1. 如果`s[i-1]=='('`，则可以延长，即`dp[i]=dp[i-2]+2`
   2. 如果`s[i-1]==')'`，则需要判断`i-1-dp[i-1]`位置(`i-1`位置所能匹配到的左括号的前一个位置)处是否是`(`，如果是则`dp[i]=dp[i-1]+2+dp[i-2-dp[i-1]]`，表示以`i-2-dp[i-1]`为分隔点的两端连接起来(因为`s[i-1]==')'`，所以此时应该是`dp[i-1]+2`)

```python 
class Solution(object):
    def longestValidParentheses(self, s):
        """
        :type s: str
        :rtype: int
        """
        res = 0
        dp = [0] * len(s)
        for i in range(1, len(s)):
            if s[i] == ')':
                if s[i-1] == '(':
                    dp[i] = dp[i-2]+2 if i-2>=0 else 2
                    res = max(dp[i], res)
                else:
                    if i-1-dp[i-1]>=0 and s[i-1-dp[i-1]] == '(':
                        dp[i] = dp[i-1]+2+dp[i-2-dp[i-1]]
                        res = max(dp[i], res)
        return res
```

相比之下，还是解法一更直观一点，不过也可能因为我动归太菜了~~