+++
title = "Leetcode - Longest Substring Without Repeating Charactes"
date = "2022-03-28T20:12:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "문자열 `s` 가 주어졌을 때 겹치는 문자가 없는 부분 문자열의 길이를 반환하라."
keywords = ["algorithm","leetcode"]
toc = false
+++

문자열 `s` 가 주어졌을 때 겹치는 문자가 없는 부분 문자열의 길이를 반환하라.


## Solution,

보자마자 알 수 있었다. Sliding Window 로 풀 수 있다는 것을. Two Pointer 알고리즘은 워낙 쉽고 좋아해서(쉬워서 좋아함..) 그냥 풀 수 있다.

```java
class Solution {
    public int lengthOfLongestSubstring(String sa) {
        int start = 0, pointer = 0, max = 0;
        Set<Character> use = new HashSet<>();
        int l = sa.length();
        char[] arr = sa.toCharArray();
        
        for (;;) {
            if (l == pointer) break;
            if (use.contains(arr[pointer])) {
                ++start;
                pointer = start;
                use.clear();
                continue;
            }
            use.add(arr[pointer]);
            max = Math.max(max, use.size());

            ++pointer;
        }
        
        return max;
    }
}
```

하지만 메모리 사용률이 엄청 높게 나왔다. 크게 생각하고 푼게 아니라, `use` 를 굳이 안쓰고 풀 수 있을텐데, 우선 이대로 제출했다.