---
layout: post
title: Leetcode 12. Integer to Roman
date: 2021-05-17 23:02 +0800
categories: leetcode
---
## 1. 问题
[integer to roman](https://leetcode.com/problems/integer-to-roman/)

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
units = [
    ['M', None, None],
    ['C','D','M'], # 百位
    ['X','L','C'], # 十位
    ['I','V','X']  # 个位
]
```
记录出现的符号模式，使用数组下标记录，这里考虑可以使用bits记录，压缩内存（使用两个bit表示一个下标）：
```python
# [A,AA,AAA,AB,B,BA,BAA,BAAA,AC], # 位串模式
# 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
pattern = [
    [],           # empty if 0
    [0],          # A
    [0, 0],       # AA
    [0, 0, 0],    # AAA
    [0, 1],       # AB
    [1],          # B
    [1, 0],       # BA
    [1, 0, 0],    # BAA
    [1, 0, 0, 0], # BAAA
    [0, 2],       # AC
], # 位串模式下标表示
```
最后只需要从十进制数字中获取相应的数字按照表格转换即可：
```python
def int2Roman(result: int)-> str:
    m = 1000
    s = ''
    for unit in units:
        p = result // m
        for idx in pattern[p]:
            s += unit[idx]
        result %= m
        m /= 10
    return s
```

### 2.1 算法分析

设输入数字的十进制长度为n，时间复杂度仅仅一次位便利即O(n)，空间是固定的常数O(k)。

### 2.2 算法实现
```java
// java
public class Solution {
    private static char[][] units = new char[][]{
            {'M', (char) 0, (char) 0},
            {'C','D','M'}, // 百位
            {'X','L','C'}, // 十位
            {'I','V','X'}  // 个位
    };

    private static int[][] pattern = new int[][]{
        {},           // empty if 0
        {0},          // A
        {0, 0},       // AA
        {0, 0, 0},    // AAA
        {0, 1},       // AB
        {1},          // B
        {1, 0},       // BA
        {1, 0, 0},    // BAA
        {1, 0, 0, 0}, // BAAA
        {0, 2},       // AC
    };

    public String intToRoman(int num) {
        int m = 1000;
        StringBuilder sb = new StringBuilder();
        for (char[] unit: units) {
            int r = num / m;
            for (int i: pattern[r]) {
                sb.append(unit[i]);
            }
            num %= m;
            m /= 10;
        }
        return sb.toString();
    }
}
```