---
layout: post
title: LeetCode 26. Remove Duplicates from Sorted Array
date: 2021-06-09 09:20 +0800
---
## 1. 问题

[remove duplicates from sorted array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/)

## 2. 解答

1. 设置两个指针：遍历指针（^）以及当前只想数组前缀无重复最后元素的指针（$）
2. 当两个指针指向的元素相同的时候，继续遍历; 当不同时，交换[L + 1]与[^]位置，两个指针都移动到下一个位置
3. 便利结束后[0, L]元素即去重后结果

<pre>
[1,1,2,2,3,3,4]
 L ^

[1,1,2,2,3,3,4]
 L ^

[1,1,2,2,3,3,4]
 L   ^ 
[1,2,1,2,3,3,4]
   L   ^ 
[1,2,1,2,3,3,4]
   L     ^ 
[1,2,3,2,1,3,4]
     L     ^
[1,2,3,2,1,3,4]
     L       ^ 
[1,2,3,4,1,3,2]
       L       ^ 
</pre>
### 2.1 算法分析

时间复杂度：$O(n)$
空间复杂度: $O(1)$

### 2.2 算法实现

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int c = 0;
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] != nums[c]) {
                int temp = nums[i];
                nums[i] = nums[++c];
                nums[c] = temp; 
            }
        }
        return c + 1;
    }
}
```
