+++
title = "Leetcode - Merge Intervals"
date = "2022-04-06T23:29:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given an array of `intervals` where `intervals[i] = [starti, endi]`, merge all overlapping intervals, and return *an array of the non-overlapping intervals that cover all the intervals in the input*."
+++

Given an array of `intervals` where `intervals[i] = [starti, endi]`, merge all overlapping intervals, and return *an array of the non-overlapping intervals that cover all the intervals in the input*.

**Example 1:**

```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
```

**Example 2:**

```
Input: intervals = [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considered overlapping.
```

**Constraints:**

- `1 <= intervals.length <= 104`
- `intervals[i].length == 2`
- `0 <= starti <= endi <= 104`



## Solution,

`[1,3], [2,4]` 와 같이 정렬되어 있을 때 `[i][0]` 은 `[i - 1][1]` 보다 작을 때 범위에 속한 것이니, 정렬만 있다면 문제가 없다.

```java
class Solution {
    public int[][] merge(int[][] arr) {
        if (arr.length < 2) return arr;
        Arrays.sort(arr, new Comparator<int[]>() {
            public int compare(int[] a, int[] b) {
                return a[0] - b[0];
            } 
        });
        
        int t = 0, ok = 1;
        for (int i = 1; i < arr.length; ++i) {
            if (arr[t][1] >= arr[i][0]) {
                arr[t][1] = Math.max(arr[t][1], arr[i][1]);
                arr[i][0] = -1;
                continue;
            }
            t = i;
            ++ok;
        }
        
        int[][] a = new int[ok][2];
        int c = 0, p = 0;
        while (c < ok) {
            if (arr[p][0] == -1) { ++p; continue; }
            a[c][0] = arr[p][0];
            a[c][1] = arr[p][1];
            ++c;
            ++p;
        }
        
        return a;
    }
}
```

ArrayList 같이 `List` 자료구조를 안써서 짜봤다. 