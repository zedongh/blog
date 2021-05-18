---
layout: post
title: Leetcode 14. Longest Common Prefix
categories: leetcode
date: 2021-05-18 22:28 +0800
---
## 1. 问题

[longest common prefix](https://leetcode.com/problems/longest-common-prefix/)

## 2. 解答

依次按位判断字符是否相同即可

### 2.1 算法分析

两层遍历：字符串数组长度n, 最短字符串长度m，时间复杂度为O(nm)

### 2.2 算法实现
```java
// java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i< strs[0].length(); i++) {
            char c = strs[0].charAt(i);
            for (String s: strs) {
                if (s.length() <= i) {
                    return sb.toString();
                }
                if (s.charAt(i) != c) {
                    return sb.toString();
                }
            }
            sb.append(c);            
        }
        return sb.toString();
    }
}
```
