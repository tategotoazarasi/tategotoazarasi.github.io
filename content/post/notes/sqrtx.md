---
title: sqrtx
tags: [Import-7704]
created: '2021-12-18T11:18:20.412Z'
modified: '2021-12-30T13:58:04.444Z'
---

---
title: "LeetCode 69. Sqrt(x)"
date: 2021-12-10T11:13:13+08:00
draft: false
tags: ["leetcode", "简单", "数学", "二分查找"]
math: true
---

给你一个非负整数 `x` ，计算并返回  `x`  的 **算术平方根** 。

由于返回类型是整数，结果只保留 **整数部分** ，小数部分将被 **舍去 。**

<!--more-->

**注意：** 不允许使用任何内置指数函数和算符，例如 `pow(x, 0.5)` 或者 `x ** 0.5` 。

**示例 1：**

> **输入：** x = 4
> 
> **输出：** 2

**示例 2：**

> **输入：** x = 8
> 
> **输出：** 2
> 
> **解释：** 8 的算术平方根是 2.82842..., 由于返回类型是整数，小数部分将被舍去。

**提示：**

- `0 <= x <= 231 - 1`
- 尝试探索所有的整数。(鸣谢：@annujoshi)
- 使用整数的排序属性来减少搜索空间。(鸣谢: @annujoshi)

```java
class Solution {
    public int mySqrt(int x) {
        if (x == 1) {
            return 1;
        }
        long l = 0;
        long r = x / 2;
        long mid = (int) (1 / invSqrt(x));
        if (mid * mid == (long) x || (mid * mid < (long) x && (mid + 1) * (mid + 1) > (long) x)) {
            return (int) mid;
        }
        if ((mid * mid) > (long) x) {
            r = mid;
        } else {
            l = mid;
        }

        while (true) {
            mid = (l + r) / 2;
            if (mid * mid == (long) x || (mid * mid < (long) x && (mid + 1) * (mid + 1) > (long) x)) {
                return (int) mid;
            }
            if ((mid * mid) > (long) x) {
                r = mid;
            } else {
                l = mid;
            }
        }
    }

    public float invSqrt(float x) {
        float xhalf = 0.5f * x;
        int i = Float.floatToIntBits(x);
        i = 0x5f3759df - (i >> 1);
        x = Float.intBitsToFloat(i);
        x *= (1.5f - xhalf * x * x);
        return x;
    }
}
```
