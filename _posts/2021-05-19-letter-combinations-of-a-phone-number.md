---
layout: post
title: Leetcode 17. Letter Combinations of a Phone Number
categories: leetcode
date: 2021-05-19 10:08 +0800
---
## 1. 问题

[letter combinations of a phone number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

## 2. 解答

这是一个求迪卡尔集问题：

$$
"23" = \{a, b, c\} \times \{d, e, f\} = \{(a, d), (a, e), (a, f), ...\}
$$

递归方程，（注意空字符串对应的不是空集）:

$$
letter\_comb(s) =
\begin{cases}
\{\{\}\}, len(s) = 0 \\
\{\{x\} \cup y | x \in Mapping[s[0]], y \in letter\_comb(s[1:]) \}, len(s) \ne 0 \\
\end{cases}
$$

### 2.1 算法分析

组合问题，算法结果指数膨胀

### 2.2 算法实现
```java
// java
import java.util.List;
import java.util.ArrayList;


class Solution {

    private static final String[] mapping = {
        "", // 0,
        "", // 1,
        "abc", // 2
        "def", // 3
        "ghi", // 4
        "jkl", // 5
        "mno", // 6
        "pqrs", // 7
        "tuv", // 8
        "wxyz", // 9
    };

    public List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<>();
        if (digits.length() == 0) {
            return result;
        }
        return _letterCombinations(digits);
    }
    
    private List<String> _letterCombinations(String digits) {
        List<String> result = new ArrayList<>();
        if (digits.length() == 0) {
            result.add("");
            return result;
        }
        String letter = mapping[digits.charAt(0) - '0'];
        List<String> subResult = _letterCombinations(digits.substring(1));
        for (int i = 0; i < letter.length(); i++) {
            for (String string: subResult) {
                result.add(letter.charAt(i) + string);
            }
        }
        return result;
    }
}
```

### 2.3 深度优先遍历

题目中隐藏的树结构：

<figure class="image">
  <img src="{{site.baseurl}}/images/letter-comb-tree.svg" alt="letter combination tree">
  <figcaption>"22"字符组合树结构</figcaption>
</figure>

使用树深度优先遍历算法，获取树的全部路径也就是问题结果。

### 2.3 深度优先遍历实现

```java
// java
import java.util.List;
import java.util.ArrayList;
import java.util.Stack;


class Solution {

    private static final String[] mapping = {
        "", // 0,
        "", // 1,
        "abc", // 2
        "def", // 3
        "ghi", // 4
        "jkl", // 5
        "mno", // 6
        "pqrs", // 7
        "tuv", // 8
        "wxyz", // 9
    };

    public List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<>();
        if (digits.length() == 0) {
            return result;
        }
        Stack<Character> stack = new Stack<>();
        dfs(digits, stack, result);
        return result;
    }
    
    private void dfs(String digits, Stack<Character> stack, List<String> result) {
        if (stack.size() >= digits.length()) {
            StringBuilder sb = new StringBuilder();
            for (char c: stack) {
                sb.append(c);
            }
            result.add(sb.toString());
            return;
        }
        String letter = mapping[digits.charAt(stack.size()) - '0'];
        for (int i = 0; i < letter.length(); i++) {
            // push on stack
            stack.push(letter.charAt(i));
            dfs(digits, stack, result);
            // don't forget restore
            stack.pop();
        }
    }
}
```