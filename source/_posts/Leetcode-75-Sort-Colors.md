---
title: Leetcode 75 Sort Colors
date: 2019-06-12 16:51:50
tags:
- Python
- Leetcode
- two pointers
categories:
- 算法
---

## 题目介绍

由`n`个元素组成的数组，每个元素的取值只能为`0`、`1`、`2`，将该数组进行排序。要求只遍历一次，不适用内置的排序函数。

<!-- more -->

`Examples:`

```shell 
Input: [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
```

## `Solutions`

因为只能遍历一次，所以需要采用两个指针。

### 两次遍历

第一次遍历将`0`放好，第二次遍历将`2`放好。但是需要两次遍历，不符合题目要求：

```python 
class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        zero, nzero = 0, 0
        while nzero < len(nums):
            if nums[nzero] == 0:
                nums[nzero], nums[zero] = nums[zero], nums[nzero]
                zero += 1
            nzero += 1
        two, ntwo = len(nums)-1, len(nums)-1
        while ntwo >= 0 and nums[ntwo] != 0:
            if nums[ntwo] == 2:
                nums[ntwo], nums[two] = nums[two], nums[ntwo]
                two -= 1
            ntwo -= 1
```

### 一次遍历

可以将两次遍历合为一次，需要注意的是：

当遍历的值为`2`时，与`two`指针指向的元素进行交换，但是有可能交换回来的值仍然为`2`，所以不能增加`p`；

当遍历的值为`1`时，即使交换回来的值仍然为`1`，也表明`p`前面的数是有序的，所以需要增加`p`。这是因为`p`永远不可能落后于指向`0`的位置`zero`。

```python 
class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        zero, two, p = 0, len(nums) - 1, 0
        while p <= two:
            if nums[p] == 2:
                nums[p], nums[two] = nums[two], nums[p]
                two -= 1
            elif nums[p] == 0:
                nums[p], nums[zero] = nums[zero], nums[p]
                zero += 1
                p += 1
            else:
                p += 1
```

