---
title: Leetcode 39 Combination Sum
date: 2019-05-23 15:46:28
tags:
- Leetcode
- Python
- backtracking
categories:
- 算法
---

## 题目介绍

给定不重复的正整数序列`nums`，找到所有加起来之和等于`target`的组合。每个数可以被任意多次使用。

要求：不能包含重复的结果。

<!-- more -->

`Examples`：

```python 
Input: candidates = [2,3,6,7], target = 7,
A solution set is:
[
  [7],
  [2,2,3]
]

Input: candidates = [2,3,5], target = 8,
A solution set is:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```

## `Solution`

这题可以使用回溯来做，因为不能出现重复的结果，所以通过排序后可以解决。又任意一个数字可以使用任意多次，那么在进行遍历的时候，应该将当前下标也包括进去。

可以优化的点：因为所有数包括`target`都是正数，因此可以通过判断值与`target`的大小提前结束判断。

```python 
class Solution(object):
    def combinationSum(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        candidates.sort()
        ans = []
        def helpfunc(i, target, res):
            if target < 0:
                return
            if target == 0:
                ans.append(res)
                return
            for j in range(i, len(candidates)):
                if candidates[j] > target:
                    return
                helpfunc(j, target-candidates[j], res+[candidates[j]])
                
        helpfunc(0, target, [])
        return ans
```

## 延伸

### `Combination Sum II`

与之相对应的还有[Leetcode 40 Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

这题不同的地方在于，给定的序列中含有重复的字符，但每个数字只能使用一次。因此在寻找下一个数字时，应当从当前下标的下一个开始寻找。然而，这样会出现重复的结果：

- `nums=[1,1,2,5,6,7,10], target=8`，需要避免出现多个`[1,2,5]`、`[1,7]`。此时`i=0`， `j=1`(`j`为下一个数字的位置)
- `nums=[1,2,2,2,5], target=5`，需要避免出现多个`[1,2,2]`。此时`i=1`，`j=3`。

只有当连续的数字都为结果的一部分时，才能进行保留(此时必定`j=i`)。因此当`j-i>=1`，并且`nums[j-1]==nums[j]`时这种情况出现时，需要排除在外。

```python 
class Solution(object):
    def combinationSum2(self, candidates, target):
        """
        :type candidates: List[int]
        :type target: int
        :rtype: List[List[int]]
        """
        candidates.sort()
        ans = []
        
        def helpfunc(i, target, res):
            if target < 0:
                return
            if target == 0:
                ans.append(res)
                return
            for j in range(i, len(candidates)):
                if j-i >=1 and candidates[j] == candidates[j-1]:
                    continue
                if candidates[j] > target:
                    return
                helpfunc(j+1, target-candidates[j], res+[candidates[j]])
        helpfunc(0, target, [])
        return ans
```

### `Permutations`

这题来自于[Leetcode 46 Permutations](https://leetcode.com/problems/permutations/)，算是标准的回溯。

要求给定不重复的数字，求出全排列。

`Examples`：

```python 
Input: [1,2,3]
Output:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]
```



对于回溯来说，需要构造传进下一个子树的状态，并且在回溯返回时复原这个状态。

```python 
class Solution(object):
    def permute(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        ans = []
        def helpfunc(res, t):
            if len(res) == len(nums):
                ans.append(res)
                return
            for j in range(len(t)):
              	# 标记这个数是否被访问过
                if t[j] == "":
                    continue
                temp, t[j] = t[j], ""
                helpfunc(res+[temp], t)
                # 复原，取消标记
                t[j] = temp
        helpfunc([], nums)
        return ans
```

### `Permutations II`

针对46的变种，如果给定的数字有重复的情况下，即[Leetcode 47 Permutations II](https://leetcode.com/problems/permutations-ii/)。

当给定的数字重复的情况下，为了产生不重复的结果，首先需要排序；接着在回溯的过程，需要判断相邻的两个数字是否相等。因此可以对46稍微改动一下：

```pytho 
class Solution(object):
    def permuteUnique(self, nums):
        """
        :type nums: List[int]
        :rtype: List[List[int]]
        """
        nums.sort()
        ans = []
        def helpfunc(res, t):
            if len(res) == len(nums):
                ans.append(res)
                return
            for j in range(len(t)):
                if j>0 and t[j] == t[j-1]:
                    continue
                if t[j] == "":
                    continue
                temp, t[j] = t[j], ""
                helpfunc(res+[temp], t)
                t[j] = temp
        helpfunc([], nums)
        return ans
```

