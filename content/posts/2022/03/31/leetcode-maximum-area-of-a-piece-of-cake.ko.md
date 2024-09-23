+++
title = "Leetcode - Maximum Area of a Piece of Cake After Horizontal and Vertical Cuts"
date = "2022-03-31T20:41:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "You are given a rectangular cake of size h x w and two arrays of integers horizontalCuts and verticalCuts where"
keywords = ["algorithm","leetcode"]
toc = false
+++

You are given a rectangular cake of size h x w and two arrays of integers horizontalCuts and verticalCuts where:

horizontalCuts[i] is the distance from the top of the rectangular cake to the ith horizontal cut and similarly, and
verticalCuts[j] is the distance from the left of the rectangular cake to the jth vertical cut.
Return the maximum area of a piece of cake after you cut at each horizontal and vertical position provided in the arrays horizontalCuts and verticalCuts. Since the answer can be a large number, return this modulo 109 + 7.

![Leetcode](https://user-images.githubusercontent.com/8151366/161118991-15352994-9a21-4a1b-afad-70b85dbb3680.png)

## Solution,

오늘 본 코딩 테스트에서 나온 문제인데, 이야 이걸 못풀었다는게 참 멍청했다. 우선, `hc` 와 `vc` 가 정렬되어 있다는 가정 하에 (안되어 있을 수 있으니 정렬 때리고) 시작한다.

사람은 태어나 서울로 보내라 하던가. 얘도 결국 핵심은 하나다. 모든 경우의 수를 다 탐색할 이유가 없다.

1. 내가 이 케잌의 끝을 안다
2. 잘리는 범위를 안다

즉, 최대로 자를 수 있는 범위를 우선 구한다. 그리고 내가 자른 것보다 남은게 더 크면 그게 최대 값일 것이다.

```java
import java.util.*;
import java.math.*;

class Solution {
    public int maxArea(int h, int w, int[] hc, int[] vc) {
        Arrays.sort(hc);
        Arrays.sort(vc);
        int curHeight = 0;
        long maxHeight = 0;
        for (int i = 0; i < hc.length; ++i) {
            maxHeight = Math.max(hc[i] - curHeight, maxHeight);
            curHeight = hc[i];
        }
        
        int curWidth = 0;
        long maxWidth = 0;
        for (int i = 0; i < vc.length; ++i) {
            maxWidth = Math.max(vc[i] - curWidth, maxWidth);
            curWidth = vc[i];
        }
        
        maxWidth = Math.max(maxWidth, w - vc[vc.length - 1]);
        maxHeight = Math.max(maxHeight, h - hc[hc.length - 1]);
        long result = maxHeight * maxWidth;
        
        return (int) ((maxHeight * maxWidth) % 1000000007);
    }
}
```

시간 복잡도는 `O(n + log n + m + log m)` 정도 같다. 

집에서 10분에 푼걸 못푸네.. 삶이란 참 기구한 법이다..