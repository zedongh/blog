---
layout: post
title: Leetcode 20. Valid Parentheses
categories: leetcode
date: 2021-06-01 21:45 +0800
---
## 1. 问题
[valid parentheses](https://leetcode.com/problems/valid-parentheses/)

## 2. 解答

借助栈检查括号匹配，当遇到左括号'(', '\{', '['压栈，遇到右括号')', '\}', ']'弹出相匹配的括号，如果没有则匹配异常，遍历结束判断此时栈是否为空。这里有个实现技巧压栈的时候可以压入对偶的括号，这样只需要出栈是否相同即可。

### 2.1 算法分析

一趟输入串遍历，时间复杂度$O(n)$，空间复杂度最坏情况为$O(n)$

### 2.2 算法实现
```java
import java.util.Stack;


class Solution {
    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        for (int i = 0; i < s.length(); i++) {
            char c = s.charAt(i);
            switch (c) {
                case '(':
                    stack.push(')');
                    break;
                case '{':
                    stack.push('}');
                    break;
                case '[':
                    stack.push(']');
                    break;
                default:
                    if (stack.isEmpty()) {
                        return false;
                    }
                    if (stack.pop() != c) {
                        return false;
                    }
                    break;
            }
        }
        return stack.isEmpty();
    }
}
```