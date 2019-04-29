---
title: 常用排序算法的Python实现
date: 2019-04-12 16:57:30
tags:
- Python
- 算法
categories:
- 算法
---

使用`python`实现冒泡排序，插入排序，选择排序，希尔排序，归并排序，快速排序和堆排序。

<!-- more -->

## 冒泡排序

思路：通过交换相邻的两个数，使得更小的数在前较大的数在后(或反之)，每次便利后，最大的数则被交换至最后。对`n`个数需要`o(n^2)`的比较次数，对于包含大量元素的数列排序是很没有效率的。

基本的`python`实现：

```python
def BubbleSort(nums):
    for i in range(len(nums)):
        for j in range(1, len(nums)-i):
            if nums[j] < nums[j-1]:
                nums[j], nums[j-1] = nums[j-1], nums[j]

    return nums
```

## 插入排序

思路：对于未排序数据，在已排序数据中从后向前扫描，找到相应位置并插入。需要用到`o(1)`的空间，平均算法复杂度为`o(n^2)`(最好情况下是`o(n)`)，同样的也不适合数据量大的排序应用。

```python
def InsertSort(nums):
    for i in range(len(nums)-1):
        key = nums[i+1]
        for j in range(i, -2, -1):
            if key < nums[j]:
                nums[j+1] = nums[j]
            else:
                break
        nums[j+1] = key
    return nums
```

## 选择排序

思路：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕(`维基百科`）。我写冒泡排序时，总是会和选择排序混在一起，与冒泡排序不同的是，每一次交换一对元素，至少有一个将会被移到最终位置上。


```python
def SlectSort(nums):
    for i in range(len(nums)):
        min_idx = i
        for j in range(i+1, len(nums)):
            if nums[j] < nums[min_idx]:
                min_idx = j
        nums[i], nums[min_idx] = nums[min_idx], nums[i]
    return nums
```

> 虽然总共比较次数需要`o(n^2)`，但是交换次数是`o(n)`，因此会比冒泡排序要快。

## 希尔排序

思路：通过将比较的元素分为几个区域从而提升插入排序的性能，然后逐渐递减步长，到了最后一步，即是普通的插入排序了。时间复杂度根据数组长度的变化而变化。

```python
def ShellSort(nums):
    step = len(nums) / 2
    while step:
        for i in range(step):
            nums[i::step] = InsertSort(nums[i::step])
        step /= 2
    return nums
```

优化过后，不需要每次都重新分配空间：

```python
def ShellSort1(nums):
    step = len(nums) / 2
    while step:
        for i in range(step, len(nums)):
            key, j = nums[i], i
            # 插入排序
            while j-step >= 0 and nums[j-step] > key:
                nums[j] = nums[j-step]
                j -= step
            nums[j] = key
        step /= 2
    return nums
```

## 归并排序

思路：将数组分成两部分，分别对两部分数组排序，再进行合并。因此需要两步：1.递归分解数组，2. 再进行合并。平均时间复杂度为`o(nlogn)`。

```python
def MergeSort(nums):
    def helpfunc(nums):
        start, end = 0, len(nums)-1
        if start < end:
            # 选择mid偏向于end的原因是
            # 当start最终等于mid时
            # 会使得nums[:mid]永远为空，nums[mid:]永远为nums[:]，从而进入死循环
            mid = end - (end - start) / 2
            nums1 = helpfunc(nums[:mid])
            nums2 = helpfunc(nums[mid:])
            return MergeArray(nums1, nums2)
        return nums
    nums = helpfunc(nums)
    return nums

def MergeArray(nums1, nums2):
    nums = []
    i, j = 0, 0
    while i < len(nums1) and j < len(nums2):
        if nums1[i] < nums2[j]:
            nums.append(nums1[i])
            i += 1
        else:
            nums.append(nums2[j])
            j += 1
    nums.extend(nums1[i:] or nums2[j:])
    return nums
```

## 快速排序

思路：每次选取一个基准数，分别从两端开始遍历，将比基准数小的值放在前面，比基准数大的值放在后面，递归对基准值两端进行排序。平均时间复杂度为`o(nlogn)`

```python
def QuickSort(nums):
    def helpfunc(start, end):
        if start >= end:
            return
        left, right = start, end
        # 每次选取最左端的值为基准值
        key = nums[left]
        # left始终放置比key小的值，right始终放置比key大的值
        while left < right:
        		# 从后往前遍历
            while right >= 0 and nums[right] > key:
                right -= 1
            if left < right:
                nums[left] = nums[right]
                left += 1
						# 从前往后遍历
            while left < right and nums[left] < key:
                left += 1
            if left < right:
                nums[right] = nums[left]
                right -= 1
        # 直到left=right，找到该key的所在位置
        nums[right] = key
        helpfunc(start, right-1)
        helpfunc(right+1, end)
    helpfunc(0, len(nums)-1)
    return nums
```

## 堆排序

思路：通过构造一个最大堆或者最小堆，每次将最后一个元素与堆顶元素互换，再重新调整为堆，这样每次取出的元素要么是最大，要么就是最小。 初始化建堆的时间复杂度为`o(n)`，排序重建堆的时间复杂度为`o(nlogn)`，所以总的时间复杂度为`o(n+nlogn)=o(nlogn)`。

> 对于节点`i`来说，其左右节点分别为`2*i+1`，`2*i+2`，每次调整只需要从非叶子节点开始。若数组长度为`n`，则堆从第`n/2-1`个节点开始往下调整。

```python
def HeapSort(nums):
    build_heap(nums)
    for i in range(len(nums)-1, -1, -1):
        nums[0], nums[i] = nums[i], nums[0]
        max_heapify(0, i)
    return nums

# 将较大元素下沉
def max_heapify(i, length):
    key = nums[i]
    left = 2*i+1
    while left < length:
        # 找出左右两个节点更小的那个
        if left+1 < length and nums[left] > nums[left+1]:
            left += 1
        # 根节点比子节点都小，无需调整
        if key < nums[left]:
            break
        # 需要注意python的连续赋值
        nums[left], nums[i] = nums[i], nums[left]
        i = left
        left, key = 2*i+1, nums[i]

# 构造堆
def build_heap(nums):
    for i in range(len(nums)/2-1, -1, -1):
        max_heapify(i, len(nums))
```

`References`：

[白话经典算法系列之八 MoreWindows白话经典算法之七大排序总结篇](<https://blog.csdn.net/MoreWindows/article/details/7961256>)

[堆排序时间复杂度分析](<https://muxx.me/2018/09/06/%E5%A0%86%E6%8E%92%E5%BA%8F%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90/>)

