---
title: Leetcode 15 3Sum
date: 2019-05-16 22:09:48
tags:
- Leetcode
- Python
categories:
- 算法
---

## 题目介绍

给定一个数组，返回所有不重复的三数之和为0的组合。

<!-- more -->

`Example`:

```python 
Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]
```

## 解题思路

刚开始使用的回溯来做，结果超时了。大神的方法是，此题使用两个指针来做，复杂度为`o(N^2)`。特此记录，学习一下。

首先需要将给定的字符串排序，这样可以更好的筛选：

1. 大于0后的数不用考虑，因为3个正数相加，永远都会大于0
2. 如果一个数与前一个数相同，那么可以跳过，因为这个数如果满足条件，那么它已经出现过了
3. 当三个数之和大于0，说明右边的指针应该往左移动；三个数之和小于0，说明左边的指针应该往右移动
4. 三数之和等于0，则左右指针都要移动
5. **需要注意的是**，当三数之和为0，左右指针同时只移动一位，仍有可能与原来的数相同，从而出现重复的答案，例如`**[-2,0,0,2,2]**`

```python 
class Solution(object):
    def threeSum(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        res = []
        nums.sort()
        
        for i in range(len(nums)-2):
            if nums[i] > 0:
                break
            if i > 0 and nums[i] == nums[i-1]:
                continue
            
            l, r = i+1, len(nums)-1
            while l < r:
                total = nums[i] + nums[l] + nums[r]
                if total == 0:
                    res.append([nums[i], nums[l], nums[r]])
                    l += 1
                    r -= 1
                    while l < r and nums[l] == nums[l-1]:
                        l += 1
                    while l < r and nums[r] == nums[r+1]:
                        r -= 1
                elif total < 0:
                    l += 1
                else:
                    r -= 1
        return res
```

