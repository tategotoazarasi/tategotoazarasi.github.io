---
title: "LeetCode 6015. 统计可以被 K 整除的下标对数目"
date: 2022-02-20T12:30:41+08:00
draft: false
tags: ["leetcode", "困难", "计数", "数学"]
math: true
---

给你一个下标从 **0** 开始、长度为 `n` 的整数数组 `nums` 和一个整数 `k` ，返回满足下述条件的下标对 `(i, j)` 的数目：

- `0 <= i < j <= n - 1` 且
- `nums[i] * nums[j]` 能被 `k` 整除。

<!--more-->

**示例 1：**

> **输入：** nums = [1,2,3,4,5], k = 2
>
> **输出：** 7
>
> **解释：**
> 共有 7 对下标的对应积可以被 2 整除：
> (0, 1)、(0, 3)、(1, 2)、(1, 3)、(1, 4)、(2, 3) 和 (3, 4)
> 它们的积分别是 2、4、6、8、10、12 和 20 。
> 其他下标对，例如 (0, 2) 和 (2, 4) 的乘积分别是 3 和 15 ，都无法被 2 整除。

**示例 2：**

> **输入：** nums = [1,2,3,4], k = 5
>
> **输出：** 0
>
> **解释：** 不存在对应积可以被 5 整除的下标对。

**提示：**

- \\(1 <= nums.length <= 10^5\\)
- \\(1 <= nums[i], k <= 10^5\\)

## 题解

考虑两个数 x 和 y 满足 xy 被 k 整除。若 x 和 k 是已知的，那么 y 必须满足什么条件呢？

我们先将 x 和 k 进行质因数分解，把 x 和 k 的公共质因数都去掉，k 剩下的质因数就是 y 必须包含的质因数。换成数学语言也就是说，y 是 \\(\frac{k}{gcd(x,k)} \\) 的倍数。这里 gcd(x,k) 是 x 和 k 的最大公因数。

因此只需要维护每个数的倍数出现了几次即可。这是一个经典的 O(nlnn) 的算法（复杂度分析见参考代码一节）。之后枚举 `nums` 中的每个数 x，与它能形成好二元组的 y 的数量，即为 \\(\frac{k}{gcd(x,k)} \\)​ 倍数的数量。

当然，这个计数方法中没有考虑 `i < j` 的限制。可以发现，这个计数方法中：

- `nums[i] * nums[j]` 能被 k 整除的下标对，在答案中统计了两次；
- `nums[i] * nums[i]` 能被 k 整除的下标，在答案中统计了一次。

因此我们需要再枚举一次 `nums`，O(n) 排除掉第二种情况，最后答案除以 2 即可。

```cpp
class Solution {
public:
    long long coutPairs(vector<int>& nums, int k) {
        long long ans = 0;
        // 统计每个数的倍数出现的次数
        auto count = unordered_map<int, int>();
        for(const auto num: nums) {
            count[num]++;
        }
        const int maximum = *max_element(nums.begin(), nums.end());
        // 为什么这个算法是 O(nlnn) 的？因为这个算法的循环次数是 n(1 + 1/2 + 1/3 + ...)，由调和级数可知括号内趋向 lnn
        for(int i = 1; i <= maximum; i++) {
            for(int j = i * 2; j <= maximum; j += i) {
                count[i] += count[j];
            }
        }
        for(const auto num: nums) {
            // 对于每个数统计与它形成好二元组的数有几个
            ans += count[k / gcd(k, num)];
            if(static_cast<long long>(num) * num % k == 0) {
                // 排除自己和自己形成好二元组的情况
                ans--;
            }
        }
        return ans / 2;
    }
};
```
