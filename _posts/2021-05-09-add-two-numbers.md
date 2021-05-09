---
layout: post
title: Leetcode 2. Add Two Numbers
date: 2021-05-09 12:41 +0800
categories: leetcode
---
# 1. 问题

[add two numbers](https://leetcode.com/problems/add-two-numbers/)

# 2. 解答

回顾下小学整数加法: 

> 整数加法有规律，相同数位要对齐。
> 和不满十落原位，满十上位要进一。
> 凑十余数落下来，加到哪位落哪位。
> 进位加数加一起，结果不差半分厘。

即：
- 从最低位置对应位置依次开始相加
- 低位加法向高位提供carry位

<figure class="image">
  <img src="{{site.baseurl}}/images/add.svg" alt="number plus">
  <figcaption>数字加法</figcaption>
</figure>

## 2.1 分析

数字$\overline{abcd}$<sup>[1]</sup>与$\overline{xyz}$的加法，可以看成是$\overline{abcd}$, $\overline{xyz}$与进位(carry)为0的加法，首先计算$\overline{d}$,$\overline{z}$,carry得到个位数字$\overline{w}$和新的carry位(carry'), 问题转化成计算$\overline{abc}$、$\overline{xy}$与carry'的加法计算子问题, 子问题的结果添加个位数字$\overline{w}$即可得到原问题的结果。

## 2.2　代码

### 2.2.1 递归实现

```java
/**
 * java
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
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
       return addTwoNumbers(l1, l2, 0);
    }
    
    public ListNode addTwoNumbers(ListNode l1, ListNode l2, int carry) {
        if (l1 == null && l2 == null) {
            if (carry == 0) {
                return null;
            } 
            else {
                return new ListNode(carry);
            }
        }
        int l = l1 == null ? 0 : l1.val;
        int r = l2 == null ? 0 : l2.val;
        int value = l + r + carry;
        ListNode head = new ListNode(value % 10);
        ListNode rest = addTwoNumbers(l1 == null ? null : l1.next, l2 == null ? null : l2.next, value / 10);
        head.next = rest;
        return head;
    }
    
}

```

### 2.2.2 无递归实现

```java
// java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode n1 = l1;
        ListNode n2 = l2;
        int carry = 0;
        ListNode head = new ListNode(); // placeholder head node
        ListNode prev = head;
        while (n1 != null || n2 != null) {
            int l = n1 == null ? 0 : n1.val;
            int r = n2 == null ? 0 : n2.val;
            int value = l + r + carry;
            prev.next = new ListNode(value % 10);
            prev = prev.next;
            n1 = n1 == null ? null : n1.next;
            n2 = n2 == null ? null : n2.next;
            carry = value / 10;
        }
        if (carry != 0) {
            prev.next = new ListNode(carry);
        }
        return head.next;
    }
    
}

```

---

[1] $\overline{abcd}$在程序中表示是逆序的，即d->c->b->a->null