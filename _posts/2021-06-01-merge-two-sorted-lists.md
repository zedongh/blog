---
layout: post
title: Leetcode 21. Merge Two Sorted Lists
categories: leetcode
date: 2021-06-01 22:10 +0800
---
## 1. 问题

[merge two sorted lists](https://leetcode.com/problems/merge-two-sorted-lists/)

## 2. 解答

有序单链表的递归解法是很trivial的，把递归解法转化成尾递归形式，进而再消除尾递归可以得到非递归的解法：

- 递归

```haskell
merge :: Ord a => [a] -> [a] -> [a]
merge [] ys = ys
merge xs [] = xs
merge xs@(x:xs') ys@(y:ys') 
    | x <= y    = x : merge xs' ys
    | otherwise = y : merge xs ys'
```

- 尾递归: 引入一个参数作为结果纪录（结果是逆序存储的)

```haskell
_merge :: Ord a => [a] -> [a] -> [a] -> [a]
_merge [] [] result = result
_merge (x:xs) [] result = _merge xs [] (x:result)
_merge [] ys result = _merge ys [] result
_merge xs@(x:xs') ys@(y:ys') result
     | x <= y    = _merge xs' ys (x:result)
     | otherwise = _merge xs ys' (y:result)

merge :: Ord a => [a] -> [a] -> [a]
merge xs ys = reverse (_merge xs ys [])
```

- 消除尾递归：尾递归容易转换成循环，结果参数可以使用局部变量，更进一步可以消除最后的reverse操作


### 2.1 算法分析

- 时间复杂度为$O(n + m)$
- 空间复杂度与具体实现相关：是否允许对参数数据进行修改，即算法对输入是否有副作用等因素决定是够copy。

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
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode result = new ListNode(); // dummy node
        ListNode tail = result;
        while (!(l1 == null && l2 == null)) {
            if (l1 == null) { 
                tail.next = l2;
                return result.next;
            }
            if (l2 == null) {
                tail.next =  l1;
                return result.next;
            }
            if (l1.val <= l2.val) {
                tail.next = l1;
                tail = l1;
                l1 = l1.next;
                tail.next = null;
            } else {
                // just swap l1 and l2
                ListNode temp = l1;
                l1 = l2;
                l2 = temp;
            }
        }
        return result.next;
    }
}
```