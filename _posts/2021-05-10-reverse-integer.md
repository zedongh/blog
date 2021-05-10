---
layout: post
title: Leetcode 7. Reverse Integer
date: 2021-05-10 09:16 +0800
categories: leetcode
---
# 1. 问题

[Reverse Integer](https://leetcode.com/problems/reverse-integer/)

# 2. 解答

## 2.1 工程思维

最大程度上利用语言机制和标准库实现算法，一般不考虑空间和时间复杂度（输入有限，不是系统的关键路径，不考虑最优实现方案）

## 2.1.1 算法分析

1. Math.abs: -123, 123 => 123, 需要注意Math.abs(-2147483648) == -2147483648.
2. toString 
3. reverse
4. parseLong: 前导0自动处理　parseLong("0012") == 12
5. check overflow: Integer.MAX_VALUE, Integer.MIN_VALUE

## 2.2.2 代码实现
```java
// java
class Solution {
    public int reverse(int x) {
        long val = (x >= 0 ? 1 : -1) * Long.parseLong(new StringBuilder().append(Math.abs((long)x)).reverse().toString());
        if (val > Integer.MAX_VALUE || val < Integer.MIN_VALUE) {
            return 0;
        }
        return (int) val;
    }
}
```

## 2.2 优化算法

### 2.1.1 算法分析

依次获取输入的位数数字，反向累计反转数字的值

### 2.2.2 代码实现
```java
// java
class Solution {
    public int reverse(int x) {
        long r = 0;　// 这里使用了更大的数字范围(long)
        while (x != 0) {
            int d = x % 10;
            x /= 10;
            r = r * 10 + d;
            if (r > Integer.MAX_VALUE || r < Integer.MIN_VALUE) { // overflow check
               return 0;
            }
        }
        
        return (int) r;
    }
}
```
