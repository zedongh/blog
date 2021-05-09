---
layout: post
title: Leetcode 3. Longest Substring Without Repeating Characters
date: 2021-05-09 14:41 +0800
categories: leetcode
---
# 1. 问题

[longest substring without repeating characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)，获取给定字符串最长无重复子串的长度。

# 2. 解答

## 2.1 Brute-Force

### 2.1.1 算法分析

- 暴力解法: 枚举所有子串，判断是否包含重复字符，获取无重复字符字串的最大长度。
- 时间复杂度：需要枚举全部子串，时间复杂度上至少需要$O(n^2)$, 实际上字串判断是否包含重复字符需要额外的O(n)时间，最终时间复杂度空间为$O(n^2) * O(n) = O(n^3)$。
- 空间复杂度：取决于字串判断是否包含重复字符算法，这里用set判断，空间复杂度为$O(n)$.


### 2.１.2 代码实现
``` python
# python
def solution(s: str) -> int:
    result = 0
    for i in range(len(s)):
        for j in range(i + 1, len(s) + 1):
            if j - i == len({*s[i:j]}):  # 判断是否包含重复字串，使用set暴力判断size
                result = result if j - i <= result else j - i
    return result
```

### 2.2 优化暴力算法

### 2.2.1 算法分析

暴力算法中存在许多没有必要的冗余计算，考虑字符串`abcdadba`，可以设定两个指针，分别指向无重复字串的开始位置（黑色箭头）和结束位置（白色箭头）。移动白色箭头过程中，检查指向的字符是否已经存在与当前的子串中，如果已经存在，那么下一个更长的无重复子串的开始位置应当位于子串重复字符的下一个位置。

<figure class="image">
  <img src="{{site.baseurl}}/images/longest-subsring-without-repeating-characters.svg" alt="longest substring without repeating characters">
  <figcaption>最长无重复字符子串</figcaption>
</figure>

- 时间复杂度: 一次遍历即可，复杂度为$O(n)$
- 空间复杂度: 借助Map存储字符和最近访问的下标映射，复杂度为$O(n)$

### 2.2.2 代码实现
```java
// java
import java.util.Map;
import java.util.HashMap;


class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> map = new HashMap<>();
        int start = 0;
        int result = 0;
        int i = 0;
        for (; i < s.length();i++) {
            char c = s.charAt(i);
            if (map.containsKey(c) && map.get(c) >= start) {　// 位于start左侧的重复字符并不在子串中
                result = Math.max(result, i - start);
                start = map.get(c) + 1;
            }
            map.put(c, i);  // 记录遍历的字符和最新位置
        }
        result = Math.max(result, i - start);　// 最后检查，此时 i == s.length()
        return result;
    }
}
```
