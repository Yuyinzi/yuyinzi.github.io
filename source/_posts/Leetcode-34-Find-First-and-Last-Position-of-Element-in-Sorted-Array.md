---
title: Leetcode 34 Find First and Last Position of Element in Sorted Array
date: 2019-05-22 15:39:19
tags: 
- Leetcode
- Python
categories:
- 算法
---

## 题目描述

给定已经排序好的序列`nums`，当给定`target`时，要求在时间复杂度为`O(logn)`内，查找`target`在`nums`中的出现范围，如果不存在，则返回`[-1,-1]`。

<!--more -->

`Examples`：

```python 
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]
  
Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]
```

## `Solution`

因为有序且要求时间复杂度为`O(logn)`，所以应当采用二分查找。

首先查找第一次出现的位置，如果不存在，可以直接返回`-1`，接着查找最右边出现的位置。所以需要查找两次。

```python 
class Solution(object):
    def searchRange(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        res = [-1, -1]
        if not nums:
            return res
        start, end = 0, len(nums)-1
        # 不断往左边查找
        while start < end:
            mid = start + (end - start) / 2
            # 只有确认不在[start, mid]之间，才能移动start
            if nums[mid] < target:
                start = mid + 1
            # 其他情况比如mid处值等于target，或者mid处的值大于target
            # 应当保守一点移动
            else:
                end -= 1
                # end = mid
        if nums[start] == target:
            res[0] = start
        else:
            return [-1, -1]
        
        end = len(nums) - 1
        while start < end:
            # 使得mid更偏向于end，比如[5,8,9]查找8时
            # 即使start+=1也不会跳过，使start不断趋近于end
            mid = start + (end - start) / 2 + 1
            if nums[mid] > target:
                end = mid - 1
            else:
                start += 1
                # start = mid
        
        if nums[start] == target:
            res[1] = start
        return res
```

> 注释的代码是大神的代码，效率更高。

总之二分查找的左右范围是让人头疼的事情~