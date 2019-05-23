---
title: Leetcode 23 Merge k Sorted Lists
date: 2019-05-19 16:11:15
tags:
- Leetcode
- Python
- priority queue
- list
categories:
- 算法
---

## 题目介绍

合并`k`个已经排序的链表。

<!-- more -->

`Example:`

```python 
Input:
[
  1->4->5,
  1->3->4,
  2->6
]
Output: 1->1->2->3->4->4->5->6
```

## `Solutions`

### 常规解法

常规解法，使用遍历，缺点就是十分的慢：

```python 
class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[ListNode]
        :rtype: ListNode
        """
        dummy = ListNode(0)
        p = dummy
        k = len(lists)
        while True:
            val = float('inf')
            index = -1
            for i in range(k):
                if lists[i]:
                    if lists[i].val < val:
                        val = lists[i].val
                        index = i
            if index == -1:
                return dummy.next
            p.next = lists[index]
            p = p.next
            lists[index] = lists[index].next
```

### 优先队列

每次都将`(node.val, node)`加入队列：

```python 
from Queue import PriorityQueue
class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[ListNode]
        :rtype: ListNode
        """
        dummy = ListNode(0)
        p = dummy
        q = PriorityQueue()
        for i in range(len(lists)):
            if lists[i]:
                q.put((lists[i].val, lists[i]))
        while not q.empty():
            cur = q.get()[1]
            p.next, p = cur, cur
            if cur.next:
                q.put((cur.next.val, cur.next))
        return dummy.next
```

需要注意的是：

```python 
p.next = cur
p = p.next
```

不能表示为：
`p.next, p = cur, p.next`
因为会产生一个中间变量`p.next`，而不是已经赋值为`cur`的`p.next`。例如：

```python 
>>> a, b = 1, 2
>>> a, b = a+b, a
>>> a
3
>>> b
1
```

这种特点可以很方便的解决链表的问题，例如：[leetcode 24 Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)

```python 
class Solution(object):
    def swapPairs(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        dummy = ListNode(0)
        dummy.next = head
        prev, p = dummy, head
        while p and p.next:
            # 0. 原始序列0->1->2->3->4
            # 1. 节点p的下个节点指向下个节点的下个节点,1->3
            # 2. 节点p指向下一个节点,p为2
            # 3. 节点p的下个节点指向p,2->1
            # 综上，节点p一直都是以前的变量，可以看做temp=p，操作的都是temp
            p.next, p, p.next = p.next.next, p.next, p
            # 同理，此时p=2, 0->2后，prev=1，p=3
            prev.next, prev, p = p, p.next, p.next.next
        return dummy.next
```

### 堆

与使用优先队列类似，也可以使用最小堆：

```python 
from heapq import *
class Solution(object):
    def mergeKLists(self, lists):
        """
        :type lists: List[ListNode]
        :rtype: ListNode
        """
        dummy = ListNode(0)
        p = dummy
        q = []
        for i in range(len(lists)):
            if lists[i]:
                heappush(q, (lists[i].val, lists[i]))
        while q:
            cur = heappop(q)[1]
            p.next, p = cur, cur
            if cur.next:
                heappush(q, (cur.next.val, cur.next))
        return dummy.next
```

