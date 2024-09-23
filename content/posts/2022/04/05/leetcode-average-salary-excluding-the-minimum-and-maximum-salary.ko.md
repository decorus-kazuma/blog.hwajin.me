+++
title = "Leetcode - Average Salary Excluding the Minimum and Maximum Salary"
date = "2022-04-05T23:41:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "You are given an array of **unique** integers `salary` where `salary[i]` is the salary of the `ith`employee."
keywords = ["algorithm","leetcode"]
+++

You are given an array of **unique** integers `salary` where `salary[i]` is the salary of the `ith`employee.

Return *the average salary of employees excluding the minimum and maximum salary*. Answers within `10-5` of the actual answer will be accepted.

**Example 1:**

```
Input: salary = [4000,3000,1000,2000]
Output: 2500.00000
Explanation: Minimum salary and maximum salary are 1000 and 4000 respectively.
Average salary excluding minimum and maximum salary is (2000+3000) / 2 = 2500
```

**Example 2:**

```
Input: salary = [1000,2000,3000]
Output: 2000.00000
Explanation: Minimum salary and maximum salary are 1000 and 3000 respectively.
Average salary excluding minimum and maximum salary is (2000) / 1 = 2000
```



## Solution,

첨에는 딱 중간 값만 뽑으면 TC가 $$O(n \log_n)$$ 이라서 개꿀 이랬는데 음~ 문제 읽고 바로 나의 착각이란걸 알았죠~

```java
class Solution {
    public double average(int[] salary) {
        int min = Integer.MAX_VALUE, max = Integer.MIN_VALUE, sl = salary.length;
        double s = 0.0;
        for (int i = 0; i < sl; ++i) {
            min = Math.min(min, salary[i]);
            max = Math.max(max, salary[i]);
            s += salary[i];
        }
        
        return (s - min - max) / (salary.length - 2);
    }
}
```
