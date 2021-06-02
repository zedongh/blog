---
layout: post
title: Leetcode 24. Swap Nodes in Pairs
categories: leetcode
date: 2021-06-02 22:40 +0800
---
## 1. 问题

[swap nodes in pairs](https://leetcode.com/problems/swap-nodes-in-pairs/)

## 2. 解答

[1] -> [2] -> [3] -> [4] -> [ ]
    x             x
[2] -> [1] -> [4] -> [3] -> [ ]

减而治之：
- 划分子问题： [3] -> [4] -> [ ]
- 交换列表的前两个元素：[1] - [2] (交换方式有两种：节点值交换，节点指针交换)
- 将子问题的答案和解决的部分问题结合

### 2.1 算法分析

时间复杂度为递推方程为：$T(n) = T(n-2) + O(1)$

### 2.2 算法实现（交换节点值）
```java
//java

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        // 值交换方式
        int temp = head.val;
        head.val = head.next.val;
        head.next.val = temp;
        head.next.next = swapPairs(head.next.next);
        return head;
    }
}

```

### 2.3 非递归实现（交换节点指针）
```java
//java

/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode result = head == null ? null : (head.next == null ? head : head.next);
        ListNode prev = new ListNode(); // dummy node
        while (head != null && head.next != null) {
            // head -> head.next -> tail(head.next.next)
            prev.next = head.next;
            ListNode tail = head.next.next;
            // head <-> head.next  tail
            head.next.next = head;
            // head.next -> head -> tail
            head.next = tail;
            prev = prev.next.next;
            head = tail;
        }
        return result;
    }
}

```