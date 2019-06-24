---
title: Leetcode 127 Word Ladder
date: 2019-06-24 18:10:38
tags:
- Python
- Leetcode
- BFS
categories:
- 算法
---

## 题目介绍

给定开始的单词`beginWord`，结束的单词`endWord`，以及变化的词表`wordList`，判断最少需要多少次，可以利用`wordList`实现`beginWord`到`endWord`的转变。每一次只能改变一个字母。

<!--more -->

`Examples:`

```python 
Input:
beginWord = "hit",
endWord = "cog",
wordList = ["hot","dot","dog","lot","log","cog"]

Output: 5

Explanation: As one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog",
return its length 5.


Input:
beginWord = "hit"
endWord = "cog"
wordList = ["hot","dot","dog","lot","log"]

Output: 0

Explanation: The endWord "cog" is not in wordList, therefore no possible transformation.
```

## `Solution`

可以使用`BFS`来做，例如从`hit`开始查找只变化一位的单词，再根据这个单词寻找是否存在只变化一位的单词…以此类推。而最终结果可以看成是一个树的层高，所以需要使用队列的思想，每次从左边弹出，从右边压入。并且由于可能会遍历到以前遍历过的单词，所以还需要标记是否已经使用过。如图：

```python 
				hit
  				 |
    			hot
      		  /     \
        	dot     lot
          /   \    /   \
        lot  dog  dot   log
    (repeat)    (repeat)
              |          |
             cog        cog
            (return)   
```

如果不对重复的值加以判断，那么`dot->lot->dot->...`会出现无尽的情况。

```python 
import string
class Solution(object):
    def ladderLength(self, beginWord, endWord, wordList):
        """
        :type beginWord: str
        :type endWord: str
        :type wordList: List[str]
        :rtype: int
        """
        stack = []
        if beginWord == endWord:
            return 1
        # visited
        d = {}
        stack.append(beginWord)
        for word in wordList:
            d[word] = 1
        d[beginWord] = 1
        res = 1
        while stack:
            length = len(stack)
            for i in range(length):
                # 从左边弹出，保证是同层
                word = stack.pop(0)
                # 去重，第二次出现必然会导致更长的结果
                if word in d:
                    del d[word]
                else:
                    continue
                if word == endWord:
                    return res
                for i in range(len(word)):
                    for alpha in string.ascii_letters[:26]:
                        tempword = word[:i] + alpha + word[i + 1:]
                        if tempword in d:
                            stack.append(tempword)
            # 每遍历完一层，结果+1
            res += 1

        return 0
```

