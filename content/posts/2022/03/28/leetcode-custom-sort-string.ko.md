+++
title = "Leetcode - Custom Sort String"
date = "2022-03-28T21:44:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "You are given two strings order and s. All the words of `order` are **unique** and were sorted in some custom order previously."
keywords = ["algorithm","leetcode"]
+++

Permute the characters of `s` so that they match the order that `order` was sorted. More specifically, if a character `x` occurs before a character `y` in `order`, then `x` should occur before `y` in the permuted string.

Return *any permutation of* `s` *that satisfies this property*.



## Solution,

교집합만 뽑아내서 하면 된다. 이것도 하도 옛날에 풀어서 다시 풀면 더 이쁘게 할 수 있을 것 같다.

```java
class Solution {
    public String customSortString(String order, String str) {
        Map<Integer, Character> a = new HashMap<>();
        Map<Character, Integer> o = new HashMap<>();
        for(int i = 0; i < order.length(); ++i) {
            o.put(order.charAt(i), i);
            a.put(i, order.charAt(i));
        }
        
        StringBuilder result = new StringBuilder();
        List<Integer> j = new ArrayList<>();
        for(int i = 0; i < str.length(); ++i) {
            char item = str.charAt(i);
            if (o.containsKey(item)) j.add(o.get(item));
            else result.append(item);
        }
        
        Collections.sort(j);
        for(Integer item : j) {
            result.append(a.get(item));
        }
        
        return result.toString();
    }
}
```
