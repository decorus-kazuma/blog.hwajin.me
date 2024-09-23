+++
title = "Leetcode - Swap Nodes In Pairs"
date = "2022-04-04T23:18:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "Given a linked list, swap every two adjacent nodes and return its head. You must solve the problem without modifying the values in the list's nodes (i.e., only nodes themselves may be changed.)"
+++

Given a linked list, swap every two adjacent nodes and return its head. You must solve the problem without modifying the values in the list's nodes (i.e., only nodes themselves may be changed.) 

**Example 1:**

```
Input: head = [1,2,3,4]
Output: [2,1,4,3]
```

**Example 2:**

```
Input: head = []
Output: []
```

**Example 3:**

```
Input: head = [1]
Output: [1]
```


## Solution,

일단 통과했으니 올리고, 한 달 후에 다시 풀어봐야겠다. `jump` 때문에 더러워졌어..

```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        int i = 1;
        ListNode eNode = new ListNode(-1, head != null && head.next != null ? head.next : head);
        ListNode target = head;
        ListNode before = null;
        ListNode jump = null;
        while (target != null) {
            if (i % 2 != 0) {
                jump = before;
                before = target;
                target = target.next;
                ++i;
                continue;
            }
            ListNode temp = target.next;
            target.next = before;
            before.next = temp;
            if (jump != null) jump.next = target;
            
            target = temp;
            
            ++i;
        }
        
        return eNode.next;
    }
}
```
