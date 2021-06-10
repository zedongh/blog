---
layout: post
title: Leetcode 27. Remove Element
date: 2021-06-10 08:45 +0800
---
## 1. 问题

[remove element](https://leetcode.com/problems/remove-element/)

## 2. 解答

使用两个指针：指向数组最后不为`target`的指针和遍历指针，当遍历指针的元素等于`target`，交换两个指针的元素即可（尾指针`*`, 遍历指针`^`）。

<pre>
       * 
[3,2,2,3]

     * 
[3,2,2,3]  (交换元素)
 ^      

   *
[2,2,3,3] （重新计算指针位置）
   ^
</pre>

### 2.1 算法分析

- 空间复杂度：$O(1)$
- 时间复杂度：$O(n)$

### 2.2 算法实现
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int k = 0;
        int i = nums.length - 1;
        while (i >= 0 && nums[i] == val) {
            i--;
        }
        if (i < 0) {
            return 0;
        }
        while (k <= i) {
            if (nums[k] == val) {
                int temp = nums[k];
                nums[k] = nums[i];
                nums[i] = temp;
                i--;
            }
            k++;
            while (nums[i] == val) {
                i--;
            }
        }
        return k;
    }
}
```

