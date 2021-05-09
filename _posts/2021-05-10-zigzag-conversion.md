---
layout: post
title: Leetcode 6. ZigZag Conversion
date: 2021-05-09 23:50 +0800
---
## 1. 问题

[ZigZag Conversion](https://leetcode.com/problems/zigzag-conversion/)

## 2. 解答

## 2.1 算法分析

列举出几轮样列，观察数据的规律：可以得到位于同一行中，竖直方向的位置的下标位置的间隔都是相同，为$\Delta = 2 * (\\# rows - 1)$，从第二行到倒数第二行，两个竖直方向元素之间还插入一个元素，可以轻易得到该元素与左边紧邻元素相差$\Delta　=　2 * (\\# rows - r - 1)$, 其中`r`为第几行（下标０开始）。

根据上述的观察，即可以按行，依次计算得到元素的下标位置。

- 时间复杂度：虽然按行计算每个下标，实际上仅仅对字符串每个元素仅仅遍历一次，空间复杂度为O(n)
- 空间复杂度：需要O(n)空间存储最终的计算结果数组

## 2.2 代码实现
```java
// java
class Solution {
    public String convert(String s, int numRows) {
        if (numRows == 1) {
            return s;
        }
        // 2 * (R - i - 1)  + 2 * i = 2 * (R-1)
        char[] chars = new char[s.length()];
        for (int i = 0, r = 0; r < numRows; r++) {
            for (int j = r; j < s.length(); j += 2 * (numRows -1)) {
                chars[i++] = s.charAt(j);
                int delta = 2 * (numRows - r - 1);
                if (delta != 0 && delta != 2 * (numRows - 1) && j + delta < s.length()) {
                    chars[i++] = s.charAt(j + delta);
                }
            }
        }
        return new String(chars);
    }
}

```