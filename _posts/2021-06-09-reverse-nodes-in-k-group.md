---
layout: post
title: Leetcode 25. Reverse Nodes in k-Group
date: 2021-06-06 01:44 +0800
---
## 1. 问题

[reverse node in k group](https://leetcode.com/problems/reverse-nodes-in-k-group/)

## 2. 解答

<pre>
[ ] -> [X] -> [Y] -> [Z]
        p      pn    pnn

X = c;
Y = c.next;
Z = Y.next;

[ ] -> [X] <- [Y]   [Z]
        p      pn   pnn

Y.next = X;

      +------------+
      |            *
[ ]  [X] <- [Y]   [Z]
 |           ^
 +-----------+

X = Y;
Y = Z;
Z = Y.next;

</pre>

### 2.1 算法分析

- 使用一个指针指向reverse的边界节点
- 依次反转中间指针节点指向
- 调整dummy节点位置，重复上述步骤

时间复杂度为: $O(n)$
空间复杂度为: $O(1)$

### 2.2 算法实现
```java
// java
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
    public ListNode reverseKGroup(ListNode head, int k) {
        if (head == null || head.next == null || k < 2) {
            return head;
        }
        
        ListNode dummy = new ListNode();
        dummy.next = head;
        ListNode fast = dummy;
        ListNode current = dummy;
        while (fast != null) {
            for (int i = 0; i < k; i++) {
                fast = fast.next;
                if (fast == null) {
                    return dummy.next;
                }
            }
            ListNode p = current.next;
            ListNode pn = p.next;
            ListNode pnn = pn.next;
            while (p != fast) {
                pn.next = p;
                p = pn;
                pn = pnn;
                if (pn == null) {
                    break;
                }
                pnn = pnn.next;
            }
            ListNode temp = current.next;
            current.next = p;
            current = temp;
            current.next = pn;
            fast = current;
        }
        return dummy.next;
    }
}
```