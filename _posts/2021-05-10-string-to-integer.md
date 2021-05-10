---
layout: post
title: Leetcode 8. String to Integer $(atoi)$
date: 2021-05-10 12:30 +0800
categories: leetcode automata
---
# 1. 问题

[string to integer （atoi）](https://leetcode.com/problems/string-to-integer-atoi/)，裁剪容错版本的Integer.parseInt。

# 2. 解答

## 2.1 算法分析

根据问题规则描述构建自动机图：
<figure class="image">
  <img src="{{site.baseurl}}/images/string-to-integer.svg" alt="string to integer">
  <figcaption>String to Integer Automata</figcaption>
</figure>
其中`*`表示可以匹配0次或多次，`\s`表示空格字符, `\d`表示数字字符，即0-9。按照上图实现相关匹配逻辑即可。

## 2.2 代码实现
```java
// java
class Solution {
    public int myAtoi(String s) {
        
        int i = 0;
        long r = 0;　　// long type
        int sign = 1;
        // skip space (\s)
        while (i < s.length() && Character.isSpaceChar(s.charAt(i))) {
            i++;
        }
        // sign
        if (i < s.length()) {
            if (s.charAt(i) == '-') {
                sign = -1;
                i++;
            } else if (s.charAt(i) == '+') {
                i++;
            }
        }
        // digit
        while (i < s.length() && Character.isDigit(s.charAt(i))) {
            r = r * 10 + (s.charAt(i) - '0');
            i++;
            if (r * sign > Integer.MAX_VALUE) {
                return Integer.MAX_VALUE;
            } else if (r * sign < Integer.MIN_VALUE) {
                return Integer.MIN_VALUE;
            }
        }
        r *= sign;
        // 并非在最后统一检查，因为数据范围上long也可能会overflow
        return (int) r;
    }
}
```

## 2.3 空间优化

上面实现中使用long做结果存储，方便检查累计过程中溢出情况，实际上可以直接使用`r > (Integer.MAX_VALUE - digit) / 10`判断是否会溢出。

```java 
class Solution {
    public int myAtoi(String s) {
        
        int i = 0;
        int r = 0;  // integer type
        int sign = 1;
        // skip space (\s)
        while (i < s.length() && Character.isSpaceChar(s.charAt(i))) {
            i++;
        }
        // sign
        if (i < s.length()) {
            if (s.charAt(i) == '-') {
                sign = -1;
                i++;
            } else if (s.charAt(i) == '+') {
                i++;
            }
        }
        // digit
        for (;i < s.length() && Character.isDigit(s.charAt(i)); i++) {
            int digit = (s.charAt(i) - '0');
            if (r > (Integer.MAX_VALUE - digit) / 10) {
                return sign == 1 ? Integer.MAX_VALUE : Integer.MIN_VALUE;    
            }
            r = r * 10 + digit;
        }
        return r * sign;
    }
}
```
