---
layout: post
title: Leetcode 10. Regular Expression Matching
date: 2021-05-16 22:03 +0800
categories: leetcode
---
## 1. 问题
[regular expression matching](https://leetcode.com/problems/regular-expression-matching/)，仅支持`.`和`*`的正则匹配。

## 2. 解答

给定字符串`source`和模式串`pat`，不考虑正则的普通匹配算法实现为：

$$
is\_match(source, pat) = 
\begin{cases}
is\_empty(source) \land is\_empty(pat), \\
false, \  is\_empty(source) \lor is\_empty(pat), \\
source[0] = pat[0] \land is\_match(source[1:], pat[1:]),
\end{cases}
$$

当模式串支持`.`时，匹配算法为：

$$
is\_match(source, pat) = 
\begin{cases}
is\_empty(source) \land is\_empty(pat), \\
false, \  is\_empty(source) \lor is\_empty(pat), \\
(source[0] = pat[0] \lor pat[0] = '.') \land is\_match(source[1:], pat[1:]),
\end{cases}
$$

当模式串再支持`*`时，由于`x*`再匹配过程中可能没有作用，存在更多的子情况，具体匹配算法为：

$first\\_match(source, pat) = source[0] = pat[0] \lor pat[0] = '.'$

$$
is\_match(source, pat) = 
\begin{cases}
is\_empty(source) \land is\_empty(pat), \\
false, \  \lnot is\_empty(source) \land is\_empty(pat), \\
is\_match(source, pat[2:]) \lor ( \\
\ \ \ \lnot is\_empty(source) \land first\_match(source, pat) \land is\_match(source[1:], pat)\\
\ \ \ ), len(pat) > 1 \land pat[2] = '*' \\
first\_match(source, pat) \land is\_match(source[1:], pat[1:]), len(pat) < 2 \lor pat[2] \neq '*' \\
\end{cases}
$$

即：
- 当`source`和`pat`都是空串，结果匹配
- 当仅`pat`为空串，结果为不匹配
- 当`source`为空串需要考虑`pat`是否为`.*`开头，结果为子问题`source`与`pat[2:]`是否匹配
- 当`source`和`pat`都不会空串的情况分为两种情况：
    - `pat`为`a*`开头(`pat[1] = '*'`)，结果可能为子问题`source`与`pat[2:]`, 或者当source与pat的首字符匹配时，即`source[1] == pat[1] or pat[1] == '.'`, 结果可能为子问题`source[1]`与`pat`，否则不匹配。
    - 当`pat`并非`a*`开头的字符串时，即`pat[1] != '*'`，需要`source`与`pat`首字符匹配且子问题`source[1:]`与`pat[1:]`也匹配

## 2.1 算法分析

递归实现算法，输入规模为输入串长度`m`，模式串长度`n`, 关键路径上有$T(m,n) = T(m, n-2) + T(m-1, n) + O(1)$，最坏时间复杂度为$T(m,n) = O(m^2 + n^2)$

## 2.2 算法实现
```java
// java
public class Solution {
    public boolean isMatch(String text, String pattern) {
        if (pattern == null && text == null) {
            return true;
        }
        else if (pattern == null || text == null) {
            return false;
        }
        else if (pattern.isEmpty() && text.isEmpty()) {
            return true;
        }
        else if (pattern.length() > 1 && pattern.charAt(1) == '*') {
            if (isMatch(text, pattern.substring(2))) {
                return true;
            }
            else if (text.isEmpty()) {
                return false;
            }
            else if (pattern.charAt(0) == text.charAt(0) || pattern.charAt(0) == '.') {
                return  isMatch(text.substring(1), pattern);
            } else {
                return false;
            }
        } else if (pattern.isEmpty() || text.isEmpty()) {
            return false;
        } else {
            return (pattern.charAt(0) == text.charAt(0) || pattern.charAt(0) == '.')
                    && isMatch(text.substring(1), pattern.substring(1));
        }
    }
}
```

### 递归问题DP优化

由于递归实现过程中的重叠子问题可以`memo`，利用空间换时间。这里实现中使用了辅助空间`matrix`，`matrix[i][j]`的含义是子问题`source[i:]`与`pat[j:]`的匹配结果。

```java
// java
class Solution {
    public boolean isMatch(String text, String pattern) {
        
        boolean[][] matrix = new boolean[text.length() + 1][pattern.length() + 1];
        
        for (int i = text.length(); i >= 0; i--) {
            for (int j = pattern.length(); j >= 0 ; j--) {
                boolean b = false;
                if (pattern.length() > j + 1 && pattern.charAt(j + 1) == '*') {
                    b = matrix[i][j+2];
                    if (!b && i != text.length() && (text.charAt(i) == pattern.charAt(j) || pattern.charAt(j) == '.')) {
                        b = matrix[i+1][j];
                    }
                } else if (i != text.length() && j != pattern.length()
                        && (text.charAt(i) == pattern.charAt(j) || pattern.charAt(j) == '.')) {
                    b = matrix[i+1][j+1];
                } else if (i == text.length() && j == pattern.length()) {
                    b = true;
                }
                matrix[i][j] = b;
            }
        }
        return matrix[0][0];
    }
}
```

