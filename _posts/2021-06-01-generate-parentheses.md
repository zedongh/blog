---
layout: post
title: Leetcode 22. Generate Parentheses
categories: leetcode backtrack
date: 2021-06-01 22:54 +0800
---
## 1. 问题

[generate parentheses](https://leetcode.com/problems/generate-parentheses/)

## 2. 解答

生成合法的4对括号情况:

- 多叉树表示：
<pre>
n = 1
 ()
n = 2
 ( )     .
  |     / \
  1    () ()
 (())  ()()
n = 3
 ( )       .       .        .
  |       / \     / \     / | \
  2      () (1) (1) ()   () () ()
 ((()))  ()(())  (())()  ()()()
 (()())
n = 4
 ( )        .        .        .         .          .         .          .
  |        / \      / \      / \      / | \      / | \     / | \     / | | \
  3       ()  (2)  (1) (1)  (2)(1)  (1) () ()  () (1) () () () (1) () () () ()
(((()))) ()((())) (())(()) ((()))() (())()()  ()(())()   ()()(())  ()()()()
((()())) ()(()())          (()())()
(()(()))
((())())
(()()())
</pre>

- 二叉树表示：
<pre>
n = 1
 ()
n = 2
 ( )     .
  |     / \
  1    ()  1
 (())  ()()
n = 3
 ( )       .       .        
  |       / \     / \ 
  2      ()  2   (1) ()
 ((()))  ()(())  (())()
 (()())  ()()()
n = 4
 ( )        .        .        .
  |        / \      / \      / \
  3       ()  3   (1)  2    (2) 1 
(((()))) ()((())) (())(()) ((()))()        
((()())) ()(()()) (())()() (()())()
(()(())) ()(())()
((())()) ()()(())
(()()()) ()()()()
</pre>

- 递归算法：(拓展了n=0的情况)

$$
gen\_paren(n) =
\begin{cases}
\{""\}, n = 0, \\ 
\{"()"\}, \ n = 1, \\
\{ concat("(", x, ")", y) | x \in gen\_paren(i-1), y \in gen\_paren(n-i), i \in \{1...n-1\} \} \  \cup \ \\ 
\ \ \ \ \{ concat("(", x, ")") | x \in gen\_paren(n-1) \}, n > 1
\end{cases}
$$

### 2.1 算法分析

时间复杂度递归方程：$T(n) = T(n - 1) + \sum_{i=1}^{n-1}{T(i)T(n-i)}$

### 2.2 算法实现

```java
// java
import java.util.List;
import java.util.ArrayList;


class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        if (n == 0) {
            result.add("");
            return result;
        }
        if (n == 1) {
            result.add("()");
            return result;
        }
        List<String> n1 = generateParenthesis(n-1);
        for (String e: n1) {
            result.add("(" + e + ")");
        }
        for (int i = 1; i < n; i++) {
            List<String> a = generateParenthesis(i - 1);
            List<String> b = generateParenthesis(n - i);
            for (String e1: a) {
                for (String e2: b) {
                    result.add("(" + e1 + ")" + e2);
                }
            }
        }
        return result;
    }
}
```

### 2.3 非暴力解法

递归解法重复计算了许多子问题，如果做`memo`，空间复杂度会很高，考虑使用回溯的方式。
回溯解法需要借助状态变量记录状态，回溯的时候恢复状态。

### 2.4 回溯实现
```java
// java
import java.util.List;
import java.util.ArrayList;


class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        StringBuilder sb = new StringBuilder();
        // result record final result parentheses
        // sb for backtracking state store
        dfs(n, sb, 0, 0, result);
        return result;
    }

    private void dfs(int n, StringBuilder sb, int open, int close, List<String> result) {
        if (sb.length() == n * 2) {
           result.add(sb.toString());
           return; 
        }
        if (open < n) {
            sb.append('(');
            dfs(n, sb, open + 1, close, result);
            sb.deleteCharAt(sb.length() - 1); // remove '('
        }
        if (open > close) {
            sb.append(')');
            dfs(n, sb, open, close + 1, result);
            sb.deleteCharAt(sb.length() - 1); // remove ')'
        }
    }
}
```

## 3. 算法拓展

生成括号不再局限于单一种类，可以是$()$[]{}。

### 3.1 回溯实现
```java
// java
import java.util.List;
import java.util.ArrayList;


class Solution {

    private static char[] lparens = { '(', '[', '{' };
    private static char[] rparens = { ')', ']', '}' };

    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        StringBuilder sb = new StringBuilder();
        Stack<Integer> stack = new Stack<>();
        // result record final result parentheses
        // sb for backtracking state store
        dfs(n, sb, 0, stack, result);
        return result;
    }

    private void dfs(int n, StringBuilder sb, int open, Stack<Integer> stack, List<String> result) {
        if (sb.length() == n * 2) {
           result.add(sb.toString());
           return; 
        }
        if (open < n) {
            for (int i = 0; i < lparens.length; i++) {
                sb.append(lparens[i]);
                stack.push(i);
                dfs(n, sb, open + 1, stack, result);
                stack.pop();
                sb.deleteCharAt(sb.length() - 1); // remove '('
            }
        }
        if (!stack.isEmpty()) {
            int i = stack.pop();
            sb.append(rparens[i]);
            dfs(n, sb, open, stack, result);
            sb.deleteCharAt(sb.length() - 1); // remove ')'
            stack.push(i);
        }
    }
}
```