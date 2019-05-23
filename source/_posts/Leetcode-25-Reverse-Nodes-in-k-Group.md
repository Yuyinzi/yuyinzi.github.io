---
title: Leetcode 25 Reverse Nodes in k-Group
date: 2019-05-20 00:15:21
tags:
- Leetcode
- Python
- list
categories:
- 算法
---

## 题目介绍

给定链表，给定正整数`k`，要求每`k`个节点进行翻转。

<!--more-->

`Example`：

```python 
Given this linked list: 1->2->3->4->5

For k = 2, you should return: 2->1->4->3->5

For k = 3, you should return: 3->2->1->4->5
```

要求只能使用固定的常数空间，并且不能更改节点的值。

## `Solutions`

### 常规解法一

可以使用栈存储每次需要翻转的节点：

```python 
class Solution(object):
    def reverseKGroup(self, head, k):
        """
        :type head: ListNode
        :type k: int
        :rtype: ListNode
        """
        stack = []
        p, prev = head, ListNode(0)
        prev.next, dummy = head, prev
        while p:
            stack.append(p)
            p = p.next
            if len(stack) == k:
                while stack:
                    prev.next = stack.pop()
                    prev = prev.next
                    # 新生成的k-group需要与后面的节点断开，否则节点会越来越多
                    prev.next = None
        for node in stack:
            prev.next = node
            prev = prev.next
        return dummy.next
```

### 常规解法二

使用计数器，每当`k`个值，进行链表翻转的操作：

```python 
class Solution(object):
    def reverseKGroup(self, head, k):
        """
        :type head: ListNode
        :type k: int
        :rtype: ListNode
        """
        dummy, counter = ListNode(0), 0
        phead, p, cur = dummy, head, head
        while p:
            counter += 1
            p = p.next
            if counter == k:
                prev, newhead = None, cur
                # 常规链表反转
                for i in range(k):
                    nxt = cur.next
                    cur.next = prev
                    prev = cur
                    cur = nxt
                # 生成新的k-group
                phead.next = prev
                # 并且要确定下一个k-group与本k-group的连接头
                phead = newhead
                counter = 0
        phead.next = cur
        return dummy.next
```

