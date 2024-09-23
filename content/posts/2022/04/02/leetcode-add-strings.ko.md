+++
title = "Leetcode - Add Strings"
date = "2022-04-02T03:55:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given two non-negative integers, num1 and num2 represented as string, return *the sum of* num1 *and* num2 *as a string*."
+++

## Problem,

Given two non-negative integers, `num1` and `num2` represented as string, return *the sum of* `num1` *and*`num2` *as a string*.

You must solve the problem without using any built-in library for handling large integers (such as `BigInteger`). **You must also not convert the inputs to integers directly.**

**Example 1:**

```
Input: num1 = "11", num2 = "123"
Output: "134"
```

**Example 2:**

```
Input: num1 = "456", num2 = "77"
Output: "533"
```

**Example 3:**

```
Input: num1 = "0", num2 = "0"
Output: "0"
```

 

## Solution,

난이도 최하의 문제로 뒤에서 하나씩 더해가면 끝이다. 산수의 영역(..)

1. `num1` 의 마지막 원소와 `num2` 의 마지막 원소를 더한다
2. 두 집합 모두 같은 방식으로 순회하며 더한다

더할 때 9를 초과하는 수가 나올 수 있다. 예를 들어 보자

```
num1 = "999"
num2 = "99999"
```

이렇다면,

```
9 + 9 = 18
1 + 9 + 9 = 19
1 + 9 + 9 = 19
...
```

그러니 10을 나눈 나머지를 값으로 쓰고, 나눈 값을 올린다. 뭐 편한대로 (..) 걍 1 올려도 상관 없고요. `carry = twoSum > 9 ? 1 : 0;` 이래도 괜찮은데 뭐.. 내 취향은 아니다.

```java
class Solution {
    public String addStrings(String num1, String num2) {
        char[] alpha = num1.toCharArray();
        char[] beta = num2.toCharArray();
        var result = new StringBuilder();
        int alphaSize = alpha.length - 1;
        int betaSize = beta.length - 1;
        int carry = 0;
        for (;;) {
            if (alphaSize < 0 && betaSize < 0) break;
            int aResult = alphaSize >= 0 ? alpha[alphaSize] - '0' : 0;
            int bResult = betaSize >= 0 ? beta[betaSize] - '0' : 0;
            int twoSum = aResult + bResult + carry;
            int val = twoSum % 10;
            carry = val / 10;
            result.append(val);
            --alphaSize;
            --betaSize;
        }
        
        if (carry != 0) result.append(carry);
        
        return result.reverse().toString();
    }
    
}
```

극강의 효율을 원한다면 Native array 를 써도 괜찮을 것 같다. 다만 배열 사이즈 늘리는게 귀찮다.

---

공간 복잡도의 경우 $$O(max(n, m))$$ 이긴 하다. 자리가 바뀌는 경우가 생기니 1을 더하는 것도 맞다. Big-O 표기법에 안맞을 뿐.