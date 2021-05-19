---
layout: post
title: Leetcode 16. 3Sum Closest
date: 2021-05-19 08:50 +0800
categories: leetcode
---
## 1. 问题

[3 sum closest](https://leetcode.com/problems/3sum-closest/)

### 2. 解法

参考3sum解法

### 2.1 算法实现
```java
// java
import java.util.Arrays;

class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        int delta = Integer.MAX_VALUE;
        for (int i = 0; i < nums.length - 2; i++) {
            int j = i + 1;
            int k = nums.length - 1;
            while (j < k) {
                int diff = nums[i] + nums[j] + nums[k] - target;
                if (diff > 0) {
                    k--;
                } else if (diff < 0) {
                    j++;
                } else {
                    return target;
                }
                if (Math.abs(diff) < Math.abs(delta)) {
                    delta = diff;
                }
            }
        }
        return target + delta;
    }
}
```
