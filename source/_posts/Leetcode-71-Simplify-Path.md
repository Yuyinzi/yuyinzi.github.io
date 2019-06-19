---
title: Leetcode 71 Simplify Path
date: 2019-06-19 15:10:15
tags:
- Python
- Leetcode
categories:
- 算法
---

## 题目介绍

将不规范的`Unix`路径改为规范的路径。其中`..`代表回到上级目录，`.`代表当前目录。

<!-- more -->

`Examples:`

```python 
Input: "/home/"
Output: "/home   # 最后不应当有斜线
  
Input: "/../"
Output: "/"
  
Input: "/home//foo/"
Output: "/home/foo"   # 去除多余的斜线
  
Input: "/a/./b/../../c/"
Output: "/c"          # a->b->a->/->c

Input: "/a//b////c/d//././/.."
Output: "/a/b/c"       # a->b->c->d->c
```

## `Solution`

可以利用栈来实现，如果出现`..`，则弹出，如果是`.`，不进行操作，如果非空，则入栈。

没有用`split`的实现：

```python 
class Solution(object):
    def simplifyPath(self, path):
        """
        :type path: str
        :rtype: str
        """
        stack = []

        i = 0
        while i < len(path):
            j = i+1
            while j < len(path) and path[j] != '/':
                j += 1
            words = path[i+1:j]
            if words == '..':
                if stack:
                    stack.pop()
            elif words == '.' or not words:
                pass
            else:
                stack.append(words)
            i = j
        res = ''
        while stack:
            res = '/' + stack.pop() + res
        if not res:
            res = '/'
            
        return res
```

如果结果为空，则返回根目录。

使用`split`，更加优雅一点：

```python 
class Solution(object):
    def simplifyPath(self, path):
        """
        :type path: str
        :rtype: str
        """
        stack = []

        dirs = path.split('/')
        for dir in dirs:
            if dir == '..':
                if stack:
                    stack.pop()
            elif dir == '.' or not dir:
                pass
            else:
                stack.append(dir)
        res = ''
        while stack:
            res = '/' + stack.pop() + res
        if not res:
            res = '/'
            
        return res
```

