---
layout: post
title: Leetcode 5. Longest Palindromic Substring
date: 2021-05-09 17:20 +0800
categories: leetcode
---
# 1. 问题

[longest palindromic substring](https://leetcode.com/problems/longest-palindromic-substring/)

# 2. 解答

## 2.1 Brute-Force

### 2.1.1 算法分析

- 暴力算法：遍历所有子串，判断是否为回文串，获取最长回文子串
- 时间复杂度：$O(n^3)$
- 空间复杂度：$O(n)$

### 2.1.2 代码实现

```python
# python
def solution(s: str) -> str:
    result = ""
    for i in range(len(s)):
        for j in range(i + 1 + len(result), len(s) + 1):
            if s[i:j] == s[i:j][::-1]:
                result = s[i:j] if j - i > len(result) else result
    return result

```

## 2.2 动态规划

### 2.2.1 算法分析

- dp: 子串s[i, j)为回文子串，当且进当s[i] == s[j-1]且s[i+1, j-1)也是回文子串

$$
is\_palindromic(s[i, j)) = 
\begin{cases}
true, \ i = j \lor i = j - 1 \\
is\_palindromic(s[i+1, j-1)) \land s[i] = s[j],
\end{cases}
$$

- 表格法：使用2维度矩阵计算子串是否为回文串，计算matrix[i][j]时依赖matrix[i-1][j-1]的结果

<figure class="image">
  <img src="{{site.baseurl}}/images/longest-palindromic-substring.svg" alt="longest palindromic substring">
  <figcaption>最长回文子串</figcaption>
</figure>

- 时间复杂度: 需要遍历构建存储矩阵，最坏情况下时间复杂度为:$O(n*(n+1))$
- 空间复杂度：使用2维矩阵存储，空间复杂度为$O(n*(n+1))$

### 2.2.2 代码实现

```java
// java
class Solution {
    public String longestPalindrome(String s) {
        boolean[][] matrix = new boolean[s.length()][s.length() + 1];
         int start = 0, end = 0;
        for (int i = 0; i < matrix.length; i++) {
            matrix[i][i+1] = true;
            start = i; end = i + 1;
            matrix[i][i] = true;
        }
       
        for (int j = 2; j < matrix.length + 1; j++) {
            for (int i = 0; i < j - 1; i++) {
                if (matrix[i+1][j-1] && s.charAt(i) == s.charAt(j-1)) {
                    matrix[i][j] = true;
                    if (j - i > end - start) {
                        start = i;
                        end = j;
                    }
                }
            }
        }
        return s.substring(start, end);
    }
}
```

## 2.3 中心拓展法

### 2.3.1 算法分析

由于回文串是中心对称的，可以选择子串的中心位置，尝试拓展两端，寻找以此为中心最大回文字串。
- 时间复杂度：需要遍历一次字符串O(n)，确定中心位置，然后以次中心位置做两次拓展(O(2n))，最坏情况下时间复杂度为O(2n^2)
- 空间复杂度：由于利用额外空间存储辅助信息，空间复杂度为O(1)

### 2.3.2 代码实现
```java
class Solution {
    public String longestPalindrome(String s) {
        int[] pos = new int[]{0, -1};
        for (int i = 0; i < s.length(); i ++) {
            expandMax(s, i, i, pos);   // bcacb, a -> cac -> bcacb 
            if (i + 1 < s.length() && s.charAt(i) == s.charAt(i+1)) {
                expandMax(s, i, i + 1, pos); // bccb, cc -> bccb
            }
        }
        return s.substring(pos[0], pos[1] + 1);
    }
    // argument pos as passing and mutable return value
    private void expandMax(String s, int left, int right, int[] pos) {
        while (left > 0 && right < s.length() - 1 && s.charAt(left - 1) == s.charAt(right + 1)) {
            left--;
            right++;
        }
        if (right - left > pos[1] - pos[0]) {
            pos[0] = left;
            pos[1] = right;
        }
    }
}
```

## 2.4 Manacher算法

### 2.4.1 算法分析

可以在算法2.3中看到回文数判断，过程中考虑回文串的奇偶性，以及边界处理，Manacher算法首先对字符串进行预处理成奇串：通过间隔插入相同字符，（这里选择了`#`），以及首位插入不同的字符（这里选择`^`和`$`)，使得算法可以做到很好统一。
<figure class="image">
  <img src="{{site.baseurl}}/images/manacher-proprocessing.svg" alt="manacher algorithm proprocessing">
  <figcaption>Manacher算法预处理</figcaption>
</figure>
预处理后字符串，可以计算出以每个字符为中心的最长回文串的半径$p[i]$，由于插入字符的原因，可以得到原字符串以该位置为中心的最长回文串的长度为$p[i]-1$

这里预处理字符串需要$O(2n+3)$的空间，计算`p`也需要$O(2n+3)$，即空间上共需要$O(4n+6)$；如果一次遍历暴力计算最长回文半径数组`p`，时间复杂度为$O(n^2)$，这似乎相比其他算法并没有太大的改进。Manacher算法巧妙的地方在于可以加速数组p的计算逻辑，降低时间复杂度直到$O(n)$。

<figure class="image">
  <img src="{{site.baseurl}}/images/manacher-compute-p-array.svg" alt="manacher algorithm compute p array">
  <figcaption>Manacher算法计算数组p</figcaption>
</figure>

当计算$p[i]$时，所有的$p[k], k < i$已经被计算过了，同时计算过程中维护了历史中的最长回文字串的中心位置`c`，进而可以得到这个最长回文字串的左右边界`M'`和`M`。此时，｀i｀的位置分为两种情况，

1. `i`在`M`的右侧，此时历史信息无法提供帮助，只能使用expand from center计算以`i`为中心的最长回文子串，同时更新维护历史信息
2. `i`在`M`的左侧，这里可以根据历史信息作出一些推断，如$s[M'] \neq s[M]$， 还可以根据对称性，得到`i`关于位置`c`的对称位置`j`, 由于`p[j]`已经被计算过了，可以得到$s[j - p[j] - 1] \neq s[j + p[j] + 1]$，从而推断`p[i]`长度至少为`p[j]`或者$M - i$

### 2.4.2 代码实现
```java
// java
class Solution {
    
    public String longestPalindrome(String s) {
        // do initial work
        char[] chars = new char[2 * s.length() + 3];
        chars[0] = '^';
        chars[1] = '#';
        chars[chars.length - 1] = '$';
        for (int i = 0; i < s.length(); i++) {
            chars[i * 2 + 2] = s.charAt(i);
            chars[i * 2 + 3] = '#';
        }
        int[] p = new int[chars.length];
        p[0] = 1;
        p[p.length - 1] = 1;
        // algorithm core
        int c = 0, max = 0;
        for (int i = 1; i < p.length - 1; i++) {
            int M = p[c] + c;
            p[i] = M > i ? Math.min(p[2 * c - i], M - i) : 1;
            // ^, $ will keep index access safety
            while (chars[i + p[i]] == chars[i - p[i]]) p[i]++;
            if (i + p[i] > M) {
                c = i;
            }
            max = p[i] > p[max] ? i : max;
        }
        int id =  (max - 1) / 2;
        int bias = (p[max] - 1) / 2;
        return s.substring(id - bias, id - bias + p[max] - 1);
    }

}
```
