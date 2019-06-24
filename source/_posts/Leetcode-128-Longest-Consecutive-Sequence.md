---
title: Leetcode 128 Longest Consecutive Sequence
date: 2019-06-24 14:25:23
tags:
- Python
- Leetcode
- Union-Find
categories:
- 算法
---

## 题目介绍

给定未排序的整数序列，判断里面最长的连续元素长度。需要在`O(n)`复杂度内完成。

<!-- more -->

`Example:`

```python 
Input: [100, 4, 200, 1, 3, 2]
Output: 4
Explanation: The longest consecutive elements sequence is [1, 2, 3, 4]. Therefore its length is 4.
```

## `Solutions`

### 简单解法

因为在`O(n)`复杂度内，所以不能进行排序。在讨论区看到一个很简单的解法：

对每个元素，判断其`+1`、`+2`是否也在序列中，如果`元素-1`出现在此序列中，表明已经被遍历过可以跳过了。其中`in`在`set`中的时间复杂度为`O(1)`。

```python 
class Solution(object):
    def longestConsecutive(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        nums = set(nums)
        res = 0
        for num in nums:
            if num - 1 in nums:
                continue
            best = 1
            while num + 1 in nums:
                best += 1
                num += 1
            res = max(best, res)
        return res
```

### `Union-Find`

`union-find`可以分为两种，`quick-find`和`quick-union`。顾名思义，前者是查询比较快，后者是联结比较快。原因在于，`quick-find`的联结过程是需要遍历所有的节点，将原本指向某一节点的节点指向需要联结的节点。而`quick-union`是需要遍历一个节点直到其根节点，然后再将根节点与联结的节点的根节点进行联结。

这题采用`quick-union`的思路。连接的是下标，不是`nums`中的值。

```python 
class Solution(object):
    def longestConsecutive(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        self.idx = [0 for i in range(len(nums))]
        d = {}
        self.start(nums)
        for i in range(len(nums)):
            if nums[i] in d:
                continue
            else:
                d[nums[i]] = i
            if nums[i] + 1 in d:
                self.union(i, d[nums[i] + 1])
            if nums[i] - 1 in d:
                self.union(d[nums[i] - 1], i)
        return self.count()

    def start(self, nums):
        for i in range(len(nums)):
            self.idx[i] = i

    def find(self, p):
        # 寻找最终的根节点
        while p != self.idx[p]:
            p = self.idx[p]
        return p

    def union(self, p, q):
        proot = self.find(p)
        qroot = self.find(q)
        # 将p的根节点改为q的根节点
        self.idx[proot] = qroot

    def count(self):
        res_count = {}
        res = 0
        for i in range(len(self.idx)):
          	# 关键要统计每一个节点的根节点
            root = self.find(self.idx[i])
            if root not in res_count:
                res_count[root] = 1
            else:
                res_count[root] += 1
            res = max(res_count[root], res)
        return res
```

