---
title: "LeetCode 6040. Maximum Total Beauty of the Gardens"
date: 2022-04-10T12:20:00+08:00
draft: false
tags: ["leetcode", "困难", "前缀和", "贪心", "二分"]
math: true
---

Alice is a caretaker of `n` gardens and she wants to plant flowers to maximize the total beauty of all her gardens.

You are given a **0-indexed** integer array `flowers` of size `n`, where `flowers[i]` is the number of flowers already planted in the `ith` garden. Flowers that are already planted **cannot** be removed. You are then given another integer `newFlowers`, which is the **maximum** number of flowers that Alice can additionally plant. You are also given the integers `target`, `full`, and `partial`.

A garden is considered **complete** if it has **at least** `target` flowers. The **total beauty** of the gardens is then determined as the **sum** of the following:

- The number of **complete** gardens multiplied by `full`.
- The **minimum** number of flowers in any of the **incomplete** gardens multiplied by `partial`. If there are no incomplete gardens, then this value will be `0`.

Return _the **maximum** total beauty that Alice can obtain after planting at most_ `newFlowers` _flowers._

<!--more-->

**Example 1:**

> **Input:** flowers = [1,3,1,1], newFlowers = 7, target = 6, full = 12, partial = 1
>
> **Output:** 14
>
> **Explanation:** Alice can plant

- 2 flowers in the 0th garden
- 3 flowers in the 1st garden
- 1 flower in the 2nd garden
- 1 flower in the 3rd garden
  The gardens will then be [3,6,2,2]. She planted a total of 2 + 3 + 1 + 1 = 7 flowers.
  There is 1 garden that is complete.
  The minimum number of flowers in the incomplete gardens is 2.
  Thus, the total beauty is 1 \* 12 + 2 \* 1 = 12 + 2 = 14.
  No other way of planting flowers can obtain a total beauty higher than 14.

**Example 2:**

> **Input:** flowers = [2,4,5,3], newFlowers = 10, target = 5, full = 2, partial = 6
>
> **Output:** 30
>
> **Explanation:** Alice can plant

- 3 flowers in the 0th garden
- 0 flowers in the 1st garden
- 0 flowers in the 2nd garden
- 2 flowers in the 3rd garden
  The gardens will then be [5,4,5,5]. She planted a total of 3 + 0 + 0 + 2 = 5 flowers.
  There are 3 gardens that are complete.
  The minimum number of flowers in the incomplete gardens is 4.
  Thus, the total beauty is 3 \* 2 + 4 \* 6 = 6 + 24 = 30.
  No other way of planting flowers can obtain a total beauty higher than 30.
  Note that Alice could make all the gardens complete but in this case, she would obtain a lower total beauty.

**Constraints:**

- \\(1 <= flowers.length <= 10^5\\)
- \\(1 <= flowers[i], target <= 10^5\\)
- \\(1 <= newFlowers <= 10^{10}\\)
- \\(1 <= full, partial <= 10^5\\)
- Say we choose k gardens to be complete, is there an optimal way of choosing which gardens to plant more flowers to achieve this?
- For a given k, we should greedily fill-up the k gardens with the most flowers planted already. This gives us the most remaining flowers to fill up the other gardens.
- After sorting flowers, we can thus try every possible k and what is left is to find the highest minimum flowers we can obtain by planting the remaining flowers in the other gardens.
- To find the highest minimum in the other gardens, we can use binary search to find the most optimal way of planting.

```cpp
class Solution {
public:
    long long maximumBeauty(vector<int>& flowers, long long newFlowers, int target, int full, int partial) {
        const int n = flowers.size();
        sort(flowers.begin(), flowers.end());
        for(int i = 0; i < n; i++) {
            flowers[i] = min(flowers[i], target);
        }
        long long ans = 0;
        vector<long long> sum(n + 1, 0);///< 当前项之前的和
        for(int i = 0; i < n; i++) {
            sum[i + 1] = sum[i] + flowers[i];
        }
        for(int i = 0, j = 0; i <= n; i++) {
            const long long rest = newFlowers - (static_cast<long long>(target) * (n - i) - (sum[n] - sum[i]));///< 补齐后n-i项到target后剩余的
            if(rest >= 0) {
                while(j < i && rest >= static_cast<long long>(flowers[j]) * j - sum[j]) {
                    //将前j项补齐到flowers[j]
                    j++;
                }
                ans = max(ans, static_cast<long long>(full) * (n - i) + (j == 0 ? 0 : static_cast<long long>(partial) * min(static_cast<long long>(target - 1), (rest + sum[j]) / j)));
            }
            if(i < n && flowers[i] == target) {
                break;
            }
        }
        return ans;
    }
};
```
