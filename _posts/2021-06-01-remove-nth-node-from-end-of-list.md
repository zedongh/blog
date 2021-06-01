---
layout: post
title: LeetCode 19. Remove Nth Node From End of List
categories: leetcode
date: 2021-06-01 13:14 +0800
---
## 1. 问题
[remove nth node from end of list](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

## 2. 解答

单向链表节点删除，这里需要考虑nth(1-based)是从尾节点计算，可以使用两个间距的pointer，先走节点达到尾节点，后走的节点正好指向要删除的节点的前驱节点，
但是需要删除头节点特殊情况，可以考虑使用一个占位的HEAD结点避免特判。

<pre>
[HEAD] -> [1] -> [2] -> [3] -> [4] -> [5] -> NIL
                         ^                    ^ (n = 2, delta = n + 1 = 3)
</pre>
                              
节点删除只需要先驱节点的指针只想next.next即可
<pre>
[3] ->      [5] -> NIL
             ^
       [4] - +    
</pre>
### 2.1 算法分析

只需要一趟遍历即可$O(n)$

### 2.2 算法实现

```java
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
    public ListNode removeNthFromEnd(ListNode head, int n) {
        // HEAD placeholder
        ListNode place = new ListNode();
        place.next = head;
        ListNode quick = place;
        ListNode slow = place;
        while (n-- >= 0) {
            quick = quick.next;
        }
        while (quick != null) {
            quick = quick.next;
            slow = slow.next;
        }
        slow.next = slow.next.next;
        return place.next;
    }
}
```