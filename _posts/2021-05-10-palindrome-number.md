---
layout: post
title: Leetcode 9. Palindrome Number
date: 2021-05-10 22:04 +0800
categories: leetcode
---
# 1. 问题

[palindrome number](https://leetcode.com/problems/palindrome-number/)，判断一个整数是否为回文数。

# 2. 解答

# 2.1 分析

1. 整数转字符串，翻转字符串判断是否相等即可
2. 不借助字符串，则需要实现翻转数字，判断是否相等即可。注意存在的溢出问题。

# 2.2 代码实现
```java
// java
class Solution {
    public boolean isPalindrome(int x) {
        return x >= 0 && String.valueOf(x).equals(new StringBuilder().append(x).reverse().toString());
    }
}

// java, reverse number
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0) {
            return false;
        }
        int r = 0;
        int x1 = x;
        while (x != 0) {
            if (r > (Integer.MAX_VALUE - x % 10) / 10) {
                return false;
            }
            r = r * 10 + x % 10;
            x /= 10;
        }
        return r == x1;
    }
}

```
