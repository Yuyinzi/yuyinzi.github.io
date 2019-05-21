---
title: Leetcode 31 Next Permutation
date: 2019-05-21 18:36:57
tags:
- Leetcode
- Python
categories:
- 算法
---

## 题目介绍

找到给定数组的字典序的下一个排列，如果不存在，应当返回最小的排列。要求原地替换数字，使用常数空间的内存。

<!-- more -->

`Example`：

```python 
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1
```

## `Solution`

这题没有想到好的办法，看了大神的解法，记录一下这种算法。

步骤如下：

1. 从右至左，查找第一个`nums[k]<nums[k+1]`的位置，即`k`右边的子序列单调不增；
2. 如果不存在，说明整个序列单调不增，返回排完序的整个数组；
3. 再次从右至左，找到第一个比`nums[k]`更大的位置`l`，并且执行交换`swap(nums[k], nums[l])`；
4. 然后再将`k`右边的序列按升序排列

```python 
class Solution(object):
    def nextPermutation(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        i = len(nums)-2
        while i >= 0:
            if nums[i] < nums[i+1]:
                break
            i -= 1
        if i < 0:
            nums[:] = nums[::-1]
        else:
            for j in range(len(nums)-1, i-1, -1):
                if nums[j] > nums[i]:
                    nums[j], nums[i] = nums[i], nums[j]
                    break
            nums[i+1:] = nums[-1:i:-1]
```

