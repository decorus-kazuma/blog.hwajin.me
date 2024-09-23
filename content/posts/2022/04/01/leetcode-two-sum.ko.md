+++
title = "Leetcode - Two Sum"
date = "2022-04-01T19:43:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target. You may assume that each input would have exactlyvone solution, and you may not use the same element twice."
keywords = ["algorithm","leetcode"]
toc = false
+++

Given an array of integers `nums` and an integer `target`, return *indices of the two numbers such that they add up to `target`*. You may assume that each input would have ***exactly\* one solution**, and you may not use the *same* element twice.

You can return the answer in any order.

**Example 1:**

```
Input: nums = [2,7,11,15], target = 9
Output: [0,1]
Explanation: Because nums[0] + nums[1] == 9, we return [0, 1].
```

**Example 2:**

```
Input: nums = [3,2,4], target = 6
Output: [1,2]
```

**Example 3:**

```
Input: nums = [3,3], target = 6
Output: [0,1]
```



## Solution,

1. 먼저 모두 순회시켜서 Map 에 넣는다
2. 다시 순회하면서 계산해본다

```kotlin
class Solution {
    fun twoSum(nums: IntArray, target: Int): IntArray {
        val temp = mutableMapOf<Int, Int>();
        for ((i, v) in nums.withIndex()) {
            temp.put(v, i);
        }
        for ((i, v) in nums.withIndex()) {
            if (temp.contains(target - v) && temp.get(target - v)!! != i) {
                return intArrayOf(temp.get(target - v)!!, i)
            }
        }
        
        return intArrayOf()
    }
}
```