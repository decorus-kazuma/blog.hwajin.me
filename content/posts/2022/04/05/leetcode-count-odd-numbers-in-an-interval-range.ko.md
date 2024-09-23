+++
title = "Leetcode - Count Odd Numbers in an Interval Range"
date = "2022-04-05T23:41:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given two non-negative integers `low` and `high`. Return the *count of odd numbers between* `low` *and* `high` *(inclusive)*."
keywords = ["algorithm","leetcode"]
+++

Given two non-negative integers `low` and `high`. Return the *count of odd numbers between* `low` *and* `high` *(inclusive)*.

**Example 1:**

```
Input: low = 3, high = 7
Output: 3
Explanation: The odd numbers between 3 and 7 are [3,5,7].
```

**Example 2:**

```
Input: low = 8, high = 10
Output: 1
Explanation: The odd numbers between 8 and 10 are [9].
```


 ## Solution,

생각하지 않고 풀었다.

```java
class Solution {
    public int countOdds(int low, int high) {
        if (low % 2 == 0) low += 1;
        if (high % 2 == 0) high -= 1;
        
        if (high - low == 0) return 1;
        return ((high - low) / 2) + 1;
    }
}
```
