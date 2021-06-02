---
layout: post
title: Leetcode 23. Merge k Sorted Lists
categories: leetcode heap priority-queue
date: 2021-06-02 09:52 +0800
---
## 1. 问题

[merge k sorted lists](https://leetcode.com/problems/merge-k-sorted-lists/)

## 2. 解答

使用优先级队列(堆)，将这些list放入到优先级队列中，按照首元素的大小冒泡，从优先级队列中取出最小的元素，并将去首的list再次放入到优先级队列中，直到优先级队列为空为止。

### 2.1 算法分析

优先级队列元素最多维持为k个，pop出最小元素的时间复杂度为$O(log(k))$, 需要操作$O(n_1 + n_2 + ... + n_k)$次，复杂度为$O((n_1 + n_2 + ... + n_k)log(k))$

### 2.2 算法实现
```java
// java
import java.util.Comparator;
import java.util.List;
import java.util.PriorityQueue;

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
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> priorityQueue = new PriorityQueue<>(new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                return o1.val - o2.val;
            }
        });
        ListNode result = new ListNode();
        ListNode tail = result;
        for (ListNode node : lists) {
            if (node != null) {
                priorityQueue.offer(node);
            }
        }
        while (!priorityQueue.isEmpty()) {
            ListNode poll = priorityQueue.poll();
            tail.next = new ListNode(poll.val);
            tail = tail.next;
            if (poll.next != null) {
                priorityQueue.offer(poll.next);
            }
        }
        return result.next;
    }
}
```