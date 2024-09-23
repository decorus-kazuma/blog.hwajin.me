+++
title = "Leetcode - 215.Kth Largest Element in an Array"
date = "2024-01-10 00:52:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given an integer array nums and an integer k, return the kth largest element in the array."
+++

## Problem,

Given an integer array nums and an integer k, return the kth largest element in the array.
Note that it is the kth largest element in the sorted order, not the kth distinct element.
Can you solve it without sorting?


**Example 1:**

Input: nums = [3,2,1,5,6,4], k = 2
Output: 5

**Example 2:**

Input: nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4

## Solution,

まず、媒介変数の値が $$-10^4 <=N <=10^4$$ を保障するという内容がある。 なので、 小さな空間複雑度を犠牲にする選択がよさそうだ。 合計 $$O(2n)$$

```java
class Solution {
    private static final int MAGIC = 10000;
    public int findKthLargest(int[] nums, int k) {
        int[] order = new int[20001];
        for (int num : nums) {
            order[num < 0 ? (num + MAGIC) : num + MAGIC] += 1;
        }

        for (int i = order.length - 1, m = k, s = 0; i >= 0; --i) {
            if (order[i] > 0) 
                m -= order[i];
            if (m <= 0) 
                return i < MAGIC ? i - MAGIC : i - MAGIC;
        }

        return 0;
    }
}
```