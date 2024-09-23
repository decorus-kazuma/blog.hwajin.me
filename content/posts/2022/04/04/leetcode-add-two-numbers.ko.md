+++
title = "Leetcode - Add Two Numbers"
date = "2022-04-04T23:18:00+09:00"
author = "Hwajin Lee"
tags = ["algorithm"]
description = "ou are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list."
+++

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order**, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself. 



**Example 1:**

```
Input: l1 = [2,4,3], l2 = [5,6,4]
Output: [7,0,8]
Explanation: 342 + 465 = 807.
```

**Example 2:**

```
Input: l1 = [0], l2 = [0]
Output: [0]
```

**Example 3:**

```
Input: l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
Output: [8,9,9,9,0,0,0,1]
```

## Solution,

Reverse Order 라길래 멍청하게 처음에는 Stack 으로 짰다 (..). 무척이나 개발자 스럽게도 `0` 번 인덱스를 고려한 것이었다. 제발 이런 멍청한 실수는 하지 말자.

연결 리스트를 끝까지 탐색하며 더하면 끝.

```java
import java.util.*;

class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode p1 = l1;
        ListNode p2 = l2;
        ListNode result = new ListNode(-1);
        ListNode ret = result;
        int carry = 0;
        while (p1 != null || p2 != null) {
            int val = (p1 != null ? p1.val : 0) + (p2 != null ? p2.val : 0) + carry;
            result.next = new ListNode(val % 10);
            result = result.next;
            carry = val / 10;
            
            if (p1 != null) p1 = p1.next;
            if (p2 != null) p2 = p2.next;
        }
        
        if (carry != 0) result.next = new ListNode(1);
        
        return ret.next;
    }
}
```
