---
layout: post
title: Leetcode 11. Container With Most Water
categories: leetcode
date: 2021-05-17 12:24 +0800
---
## 1. 问题
[container with most water](https://leetcode.com/problems/container-with-most-water/)
问题的数学描述：给定数组$S={a_1,a_2,a_3}$, 求下标i，j $(i < j)$使得$r(i,j) = min(a_i, a_j) \times |j - i|$的值最大。

## 2. 解答

1. 暴力算法，枚举所有的$(i, j)$组合，计算结果最大的;
2. 双指针，分别位于首尾向中间移动，移动规则为移动指针值较小的直到比移动之前的值大，记录此时的结果重复操作即可。

### 2.1 算法分析

考虑i, j $(i < j)$，$r(i, j) = min(a_i, a_j) \times (j - i)$是所有$i' <= i \land j <= j'$中结果最大的，这里假定$a_i <= a_j$, 可以得出：
- 对于i' < i，必有$a_{i'} < a_i$, 否则$r(i', j) = a_i \times (j - i') > r(i, j)$，矛盾;
- 同理，对于j < j'，必有$a_{j'} < a_i$，否则$r(i, j') = a_i \times (j' - i) > r(i, j)$，矛盾。
- 进而如果存在i', j' $(i' < j')$使得$r(i', j') > r(i, j)$，必有$i <= i' \land j' <= j$，由于(j'-i') < (j - i)，必有$min(a_i, a_j) < min(a_{i'}, a_{j'})$

仅仅需要一次遍历即可，时间复杂度为O(n), 空间复杂度为O(1)。

### 2.2 算法实现
```java
//java
class Solution {
    public int maxArea(int[] height) {
        int s = 0;
        int e = height.length - 1;
        int r = 0;
        while (s <= e) {
            r = Math.max(r, Math.min(height[s], height[e]) * (e - s));
            if (height[s] <= height[e]) {
                s++;
                while (s <= e && height[s] <= height[s - 1]) s++;
            } else {
                e--;
                while (s <= e && height[e] <= height[e + 1]) e--;
            }
        }
        return r;
    }
}
```