---
layout: post
title: Leetcode 4. Median of Two Sorted Arrays
date: 2021-05-16 10:56 +0800
categories: leetcode
---
## 1. 问题 

[median of two sorted arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)，给定两个有序数组，计算两个数组的中位数。
  

## 2. 解答

两个数组已经有序了，可以自然当成优先级队列一样使用，分别比较两个数组的最小元素，然后`pop`。根据中位数的计算规则，
- 当长度$len$为奇数的情况，只需要`pop`出前$[\frac{(len - 1)}{2}]$个，下一次`pop`出来的即为中位数;
- 当长度$len$为偶的情况，只需要`pop`出前$[\frac{(len - 1)}{2}]$个，下两次`pop`出来的平均数即为中位数。

### 2.1 算法分析

有序数组的`pop`操作使用指针表示，只需要O(1)的时间，最终需要调用约$\frac{len}{2}$次，时间复杂度为$O(\frac{len}{2})$。

### 2.2 代码实现
```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int len = nums1.length + nums2.length;
        int p1 = 0; 
        int p2 = 0;
        while (p1 + p2 < (len - 1) / 2) {
            if (p1 < nums1.length && p2 < nums2.length) {
                if (nums1[p1] < nums2[p2]) {
                    p1++;
                } else {
                    p2++;
                }
            } else if (p1 < nums1.length) {
                p1++;
            } else {
                p2++;
            }
        }
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < Math.abs(len % 2 - 2); i++) {
            if (p1 < nums1.length && p2 < nums2.length) {
                if (nums1[p1] < nums2[p2]) {
                    list.add(nums1[p1++]);
                } else {
                    list.add(nums2[p2++]);
                }
            } else if (p1 < nums1.length) {
                list.add(nums1[p1++]);
            } else {
                list.add(nums2[p2++]);
            }
        }
        double r = 0;
        for (int i = 0; i < list.size(); i++) {
            r += list.get(i);
        }
        return r / list.size();
    }
}
```

### 2.3 优化

2.1的算法并没有完全利用有序数组的一些特性，有序数组上二分查找可以减少`pop`次数的调用。


TODO
