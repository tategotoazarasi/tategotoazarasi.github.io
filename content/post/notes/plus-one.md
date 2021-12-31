---
title: plus-one
tags: [Import-7704]
created: '2021-12-18T11:18:20.412Z'
modified: '2021-12-30T13:58:03.458Z'
---

---
title: "LeetCode 66. 加一"
date: 2021-12-09T10:44:24+08:00
draft: false
tags: ["leetcode", "简单", "数组", "数学"]
math: true
---

给定一个由 **整数** 组成的 **非空** 数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储**单个**数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

<!--more-->

**示例  1：**

> **输入：** digits = [1,2,3]
> 
> **输出：** [1,2,4]
> 
> **解释：** 输入数组表示数字 123。

**示例  2：**

> **输入：** digits = [4,3,2,1]
> 
> **输出：** [4,3,2,2]
> 
> **解释：** 输入数组表示数字 4321。

**示例 3：**

> **输入：** digits = [0]
> 
> **输出：** [1]

**提示：**

- `1 <= digits.length <= 100`
- `0 <= digits[i] <= 9`

```java
class Solution {
    public int[] plusOne(int[] digits) {
        for (int i = digits.length - 1; i >= 0; i--) {
            digits[i]++;
            if (digits[i] >= 10) {
                digits[i] -= 10;
                if (i == 0) {
                    int[] ret = new int[digits.length + 1];
                    ret[0] = 1;
                    System.arraycopy(digits, 0, ret, 1, digits.length);
                    return ret;
                }
            } else {
                return digits;
            }
        }
        return digits;
    }
}
```
