---
title: Leetcode 92 Reverse Linked List II
date: 2019-06-14 18:38:29
tags:
- Python
- Leetcode
- list
categories:
- 算法
---

## 题目介绍

部分反转链表：给定位置`m`和`n`，要求一次遍历反转从`m`到`n`位置的链表。其中`1<=m<=n<=length`。

<!-- more -->

`Examples:`

```python 
Input: 1->2->3->4->5->NULL, m = 2, n = 4
Output: 1->4->3->2->5->NULL
```

## `Solutions`

### 使用栈

第一个想法是将所有要逆序的节点保存到栈里头，然后进行连接。虽然不满足一次遍历，不过这个方法比较直观好理解：

```python 
class Solution(object):
    def reverseBetween(self, head, m, n):
        """
        :type head: ListNode
        :type m: int
        :type n: int
        :rtype: ListNode
        """
        stack = []
        dummy = ListNode(0)
        dummy.next = head
        p, prev = head, dummy
        count = 1
        while p:
            if m-1 >= count:
                prev = prev.next
            if count > n:
                break
            if m <= count <= n:
                stack.append(p)
            p = p.next
            count += 1
        while stack:
            prev.next = stack.pop()
            prev = prev.next
        prev.next = p
        return dummy.next
```

### 原地反转

这个时候需要利用三个指针：

```shell 
dummy ->1 ->2 ->3 ->4 ->5
       |    |   |
       pre  s   t
```

每次固定`pre`不变，然后：

1. `s.next=t.next`，即`2->4`
2. `t.next=pre.next`，即`3->2`
3. `pre.next = t`，即`1->3`
4. `t=s.next`

经过第一次反转，有：

```python 
dummy ->1 ->3 ->2 ->4 ->5
       |        |   |
       pre      s   t
```

再重复上面的步骤翻转一次即可。

```python 
class Solution(object):
    def reverseBetween(self, head, m, n):
        """
        :type head: ListNode
        :type m: int
        :type n: int
        :rtype: ListNode
        """
        dummy = ListNode(0)
        dummy.next = head
        count = 1
        p = dummy
        
        while p and count < m:
            p = p.next
            count += 1
        
        pre, s, t = p, p.next, p.next.next
            
        for i in range(m, n):
            s.next = t.next
            t.next = pre.next
            pre.next = t
            t = s.next
        return dummy.next
```

