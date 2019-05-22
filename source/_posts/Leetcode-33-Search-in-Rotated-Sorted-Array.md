---
title: Leetcode 33 Search in Rotated Sorted Array
date: 2019-05-22 11:32:27
tags:
- Leetcode
- Python
categories:
- 算法
---

## 题目介绍

假定一个升序序列，在某个下标处发生了旋转，例如`[0,1,2,4,5,6,7]=>[4,5,6,7,0,1,2]`。给定一个`target`，要求在`O(logn)`的时间复杂度内进行查找，如果存在返回下标，否则返回`-1`。假定数组中不存在重复的数字。

<!-- more -->

`Examples`：

```python 
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1
```

## `Solution`

时间复杂度在`O(logn)`，说明只能使用二分查找。对于这个数组，可能有下面几种情况：

1. `nums[end]<nums[mid]`：表明右半部分的值是旋转而来的，左半部分是单调的
   1. `[4,5,6,7,8,0,1,2]`
   2. `[4,5,6,7,0,1,2]`
2. `nums[end]>nums[mid]`: 表明右半部分是单调的
   1. `[0,1,2,3,4,5,6,7,8]`
   2. `[8,0,1,2,3,4,5,6,7]`

因此，每次查找`target`的时候，首先确定是否在单调的部分。

```python 
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        start, end = 0, len(nums)-1
        while start <= end:
            mid = start + (end-start)/2
            if nums[mid] == target:
                return mid
            if nums[end] < nums[mid]:
              	# 可以确定落在左边单调递增区间
                if nums[mid] > target and nums[start] <= target:
                    end = mid -1
                else:
                    start = mid + 1
            else:
              	# 可以确定落在右边单调递增区间
                if nums[mid] < target and nums[end] >= target:
                    start = mid + 1
                else:
                    end = mid -1
        return -1
```

