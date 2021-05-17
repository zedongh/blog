---
layout: post
title: Leetcode 13. Roman to Integer
date: 2021-05-18 00:03 +0800
categories: leetcode
---
## 1. 问题
[roman to integer](https://leetcode.com/problems/roman-to-integer/)

罗马数字表示：

| Symbol | Value |
|--|--|
|I|1|
|IV|4|
|V|5|
|IX|9|
|X|10|
|XL|40|
|L|50|
|XC|90|
|C|100|
|CD|400|
|D|500|
|CM|900|
|M|1000|


## 2. 解答
直接暴力打表，记录十进制下每个位置上可能出现的符号：
```python
M = {
    'I': 1,
    'V': 5,
    'X': 10,
    'L': 50,
    'C': 100,
    'D': 500,
    'M': 1000,
}
```
一般情况下,如$M[LXXX] = M[L] + M[X] + M[X] + M[X]$，特殊情况$M[XL] = - M[X] + M[L]$, 可以总结出来模式串“AB”中，
- 如果$M[A] < M[B]$，则$M[AB] = -M[A] + M[B]$
- 如果$M[A] >= M[B]$，则$M[AB] = M[A] + M[B]$

### 2.1 算法分析

设输入数字的十进制长度为n，时间复杂度仅仅一次位遍历即O(n)，空间是固定的常数O(k)。

### 2.2 算法实现
```java
// java
import java.util.Map;
import java.util.HashMap;


public class Solution {
    private static final Map<Character, Integer> MAPPING = new HashMap<>() {
        {
            put('I', 1);
            put('V', 5);
            put('X', 10);
            put('L', 50);
            put('C', 100);
            put('D', 500);
            put('M', 1000);
        }
    };

    public int romanToInt(String roman) {
        int prev = 0;
        int result = 0;
        for (int i = 0; i < roman.length(); i++) {
            int current = MAPPING.get(roman.charAt(i));
            result += current;
            if (current > prev) {
                result -= 2 * prev;
            }
            prev = current;
        }
        return result;
    }
}
```