+++
title = "Leetcode - Maximum Length of a Concatenated String with Unique Characters"
date = "2022-04-06T21:46:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "You are given an array of strings `arr`. A string `s` is formed by the **concatenation** of a **subsequence** of `arr` that has **unique characters**."
+++

You are given an array of strings `arr`. A string `s` is formed by the **concatenation** of a **subsequence** of `arr` that has **unique characters**.

Return *the **maximum** possible length* of `s`.

A **subsequence** is an array that can be derived from another array by deleting some or no elements without changing the order of the remaining elements.

 
**Example 1:**

```
Input: arr = ["un","iq","ue"]
Output: 4
Explanation: All the valid concatenations are:
- ""
- "un"
- "iq"
- "ue"
- "uniq" ("un" + "iq")
- "ique" ("iq" + "ue")
Maximum length is 4.
```

**Example 2:**

```
Input: arr = ["cha","r","act","ers"]
Output: 6
Explanation: Possible longest valid concatenations are "chaers" ("cha" + "ers") and "acters" ("act" + "ers").
```

**Example 3:**

```
Input: arr = ["abcdefghijklmnopqrstuvwxyz"]
Output: 26
Explanation: The only string in arr has all 26 characters.
```

 

**Constraints:**

- `1 <= arr.length <= 16`
- `1 <= arr[i].length <= 26`
- `arr[i]` contains only lowercase English letters.



## Solution,

면접때 못푼 문제 (..). 좀 고민하니까 풀리긴 했는데, 잊을만 할 때 다시 풀어봐야겠다. 우선, String 을 풀어서 문자로 바꿔야 한다. Bitmask 를 사용하면 추가 공간을 크게 잡아먹지 않는다. 

1. ASCII 코드상에서 a~z는 26개. Java 의 int 는 32bit 라서 시프트 연산으로 a~z를 숫자로 변환 후 마스킹한다
2. 전부 마스킹 후 체크를 시작한다.
3. 지금 보는 노드보다 전의 노드는 볼 필요가 없다.
4. 하위 노드도 동일한 규칙을 가진다.

```java
class Solution {
    public int maxLength(List<String> val) {
        int[] arr = new int[val.size()];
        for (int i = 0; i < val.size(); ++i) arr[i] = stringToBitmask(val.get(i));

        return check(arr, 0, 0, 0);
    }

    private int stringToBitmask(String str) {
        char[] arr = str.toCharArray();
        int mask = 0;
        for (char item : arr) {
            if ((mask & (1 << item - 'a')) > 0) return 0;
            mask += 1 << (item - 'a');
        }

        return mask;
    }

    private int check(int[] items, int me, int position, int max) {
        for (int i = position; i < items.length; ++i) {
            if ((me & items[i]) > 0) {
                continue;
            }
            max = Math.max(max, check(items, me + items[i], i + 1, Math.max(max, Integer.bitCount(me + items[i]))));
        }

        return max;
    }
}
```

![Leetcode](https://user-images.githubusercontent.com/8151366/161979785-b54db03c-f35f-4bfb-baee-850687f29dfd.png)

Leetcode 는 Runtime이 지맘대로라 믿을 수가 없다(..)

백트래킹에 약해서 더 풀어봐야겠다. 사실 그래프 탐색 다 못하는편.