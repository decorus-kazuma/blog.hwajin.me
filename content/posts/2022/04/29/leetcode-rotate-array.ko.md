+++
title = "Leetcode - Rotate Array"
date = "2022-04-29 01:31:12"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given an array, rotate the array to the right by `k` steps, where `k` is non-negative."
keywords = ["algorithm","leetcode"]
+++

## Problem,

Given an array, rotate the array to the right by `k` steps, where `k` is non-negative. 

**Example 1:**

```
Input: nums = [1,2,3,4,5,6,7], k = 3
Output: [5,6,7,1,2,3,4]
Explanation:
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]
```

**Example 2:**

```
Input: nums = [-1,-100,3,99], k = 2
Output: [3,99,-1,-100]
Explanation: 
rotate 1 steps to the right: [99,-1,-100,3]
rotate 2 steps to the right: [3,99,-1,-100]
```

**Constraints:**

- `1 <= nums.length <= 10^5`
- `-231 <= nums[i] <= 231 - 1`
- `0 <= k <= 10^5`

**Follow up:**

- Try to come up with as many solutions as you can. There are at least **three** different ways to solve this problem.
- Could you do it in-place with `O(1)` extra space?


## Solution,

자, 배열 `[1,2,3,4,5,6,7]` 를 잘 살펴보자. `k=3` 일 때, 결과가 `[5,6,7,1,2,3,4]` 라면, `1,2,3,4` 와 `5,6,7` 을 따로 두고 생각할 수 있을 것이다.

그렇다면, 극강의 꼼수를 부려볼 수 있다. 왜냐하면, 잘 봐라. "Rotate" 라고 하지 않는가. 일단 뒤집고 생각해보자.

`[7,6,5,4,3,2,1]` 를 잘 보면 해답이 보인다. 바로 `k=3` 이라는 점에서, 우리는 이 뒤집어진 배열을 두 번만 더 뒤집으면 끝난다. 어딜 기점으로? `k=3` 즉, `k - 1` 을 기준으로.

그런데, 놀랍게도 `k` 는 `10^5` 까지 입력을 받을 수 있다. 즉, `nums.length` 보다 더 클 수 있다. 이러면 당연히 에러가 발생한다. 나머지 연산을 통해서 N값보다 클 때 언제나 N값으로 나눈 나머지, 즉 실제 우리가 회전시켜야 하는 수만 뽑아낼 수 있다.

```java
class Solution {
    public void rotate(int[] nums, int k) {
        if (nums.length == 1 || k == nums.length) {
            return;
        }
        reverse(nums, 0, nums.length - 1);
        reverse(nums, 0, k % nums.length - 1);
        reverse(nums, k % nums.length, nums.length - 1);
    }
    
    private void reverse(int[] n, int l, int r) {
        while (l < r) {
            int tmp = n[l];
            n[l] = n[r];
            n[r] = tmp;
            ++l;
            --r;
        }
    }
}
```

참고로 이 접근법은 엄청난 논쟁을 야기시킨 방법이었다.. 이 글 쓰면서 Solution 보고 알았다.