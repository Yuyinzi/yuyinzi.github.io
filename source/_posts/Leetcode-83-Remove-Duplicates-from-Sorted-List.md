---
title: Leetcode 83 Remove Duplicates from Sorted List
date: 2019-06-13 21:27:40
tags:
- Python
- Leetcode
- list
categories:
- 算法
---

## 题目介绍

给定已经排好序的链表，删除所有重复的节点，只保留一个。

<!-- more -->

`Examples:`

```python 
Input: 1->1->2
Output: 1->2
  

Input: 1->1->2->3->3
Output: 1->2->3
```

## `Solution`

可以构造一个虚构的头节点`phead`，使得当前节点`cur`指向它。遍历节点，当找到最后一个不与前面节点重复的节点`p`时，使得`cur`的`next`指向`p`。

```python 
class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        phead = ListNode(0)
        phead.next = head
        cur, p = phead, head
        while p:
            while p.next and p.val == p.next.val:
                p = p.next
            cur.next = p
            cur = cur.next
            p = p.next
        return phead.next
```

## 延伸

与这题类似的还有[Remove Duplicates from Sorted List II](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/)，只是需要将重复的节点都不保留。

`Examples:`

```python 
Input: 1->2->3->3->4->4->5
Output: 1->2->5

Input: 1->1->1->2->3
Output: 2->3
```

可以采用同样的思路，有了`phead`的存在，即使链表为`1->1->1`，也可以处理。只是相比而言需要多一点判断：

当`p`与`p.next`的值不相等时，有两种情况：

1. `p`是重复节点的最后一个，此时有`cur.next != p`，所以需要抛弃`p`节点，使得`cur`的`next`指向`p.next`，但是此时有可能`p.next`是下一个重复节点的第一个节点，所以不需要更改`cur`；
2. `p`是单独的不重复的节点，只有这种情况，才能使得`cur`向前移动。

```python 
class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if not head:
            return head
        phead = ListNode(0)
        phead.next = head
        cur, p = phead, head
        while p:
            while p.next and p.val == p.next.val:
                p = p.next
            if cur.next == p:
                cur = cur.next
                p = p.next
            else:
                cur.next = p.next
                p = p.next
        return phead.next
```

