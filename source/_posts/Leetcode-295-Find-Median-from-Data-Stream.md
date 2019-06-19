---
title: Leetcode 295 Find Median from Data Stream
date: 2019-06-19 17:32:50
tags:
- Python
- Leetcode
- heap
categories:
- 算法
---

## 题目介绍

从数据流中实时计算中位数。

<!-- more -->

`Example:`

```python 
addNum(1)
addNum(2)
findMedian() -> 1.5
addNum(3) 
findMedian() -> 2
```

## `Solution`

如果数据量比较大，将所有数据存下来，每次新增一个数据再进行排序，复杂度会比较高。这个时候可以采用两个堆的方法，一个堆(最大堆)放着较小的数，另一个堆(最小堆)放更大的数。

始终保持最小堆的个数大于等于最大堆。当两个堆元素相等时，中位数为两个堆顶元素之和的一半。当堆元素不同时，最小堆的堆顶元素。

需要注意的是：

1. 当最大堆与最小堆元素相同时，最终的效果是最小堆的元素个数加一，但是不能仅仅将元素添加至最小堆，因为有可能新增加的元素比最大堆中最大的元素要小。因此，需要先将元素添加至最大堆，再将最大的元素添加至最小堆中。
2. 当最小堆比最大堆元素多一时，最终要将元素添加至最大堆，同样的也不能直接添加至最大堆。需要添加至最小堆，将最小的元素添加至最大堆。

只有这样才能保证平衡。

```python 
class MedianFinder(object):

    def __init__(self):
        """
        initialize your data structure here.
        """
        self.max_heap = []
        self.min_heap = []

    def addNum(self, num):
        """
        :type num: int
        :rtype: None
        """
        if len(self.max_heap) == len(self.min_heap):
            max_value = -heappushpop(self.min_heap, -num)
            heappush(self.max_heap, max_value)
        else:
            min_value = heappushpop(self.max_heap, num)
            heappush(self.min_heap, -min_value)

    def findMedian(self):
        """
        :rtype: float
        """
        if len(self.max_heap) == len(self.min_heap):
            return (-1.0 * self.min_heap[0] + 1.0 * self.max_heap[0]) / 2
        else:
            return self.max_heap[0]
```

