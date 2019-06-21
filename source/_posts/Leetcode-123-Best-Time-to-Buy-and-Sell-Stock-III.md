---
title: Leetcode 123 Best Time to Buy and Sell Stock III
date: 2019-06-21 21:15:48
tags:
- Python
- Leetcode
- DP
categories:
- 算法
---

## 题目介绍

给定每一天的股票价格，最多可以完成两次交易。求能获得的最大利润数。注意：必须要卖了股票才能进行下一次交易。

<!-- more -->

```Examples:```

```python 
Input: [3,3,5,0,0,3,1,4]
Output: 6
Explanation: (3-0)+(4-1)
  
Input: [1,2,3,4,5]
Output: 4
Explanation: 5-1
```

## `Solution`

股票系列都是采用`DP`来做，看了大神的解答：

- `b1`代表第一次购买，`b1=max(b1, -price)`=>第一次购买的盈余
- `s1`代表第一次出售，`s1=max(b1+p, s1)`=>第一次出售可以获得的最大盈利
- `b2`代表第二次购买，`b2=max(s1-p, b2)`=>第二次购买时，保证当前的盈利是最大的，这样在第二次售出时，可以获得更高的结余
- `s2`代表第二次出售，`s2=max(b2+p, s2)`=>第二次出售能获得的最大值

图解：

```python 
				1		2		4		2		5		7	 |		2		4		9	
  b1	 -1	 -1	 -1	 -1  -1  -1  |   -1  -1  -1 
  s1		0		1		3		3		4		6  |		6		6		8	
  b2	 -1  -1	 -1	  1	  1	  1  |		4		4		4	
  s2		0		1		3		3		6		8	 |		8		8	  13
```

可以发现`1->7`可以获得最大的第一次交易的利润，此时如果有两次交易，最大的利润为8。因此可以继续尝试，如果后面还能增加，那么将会是最大值。

```python 
class Solution(object):
    def maxProfit(self, prices):
        """
        :type prices: List[int]
        :rtype: int
        """
        if not prices:
            return 0
        b1 = s1 = b2 = s2 = float('-inf')
        
        for price in prices:
            b1 = max(b1, -price)
            s1 = max(b1+price, s1)
            b2 = max(s1-price, b2)
            s2 = max(b2+price, s2)
        
        return s2
```







