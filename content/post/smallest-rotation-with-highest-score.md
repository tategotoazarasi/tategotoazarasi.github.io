---
title: "LeetCode 798. Smallest Rotation with Highest Score"
date: 2022-03-09T14:47:26+08:00
draft: false
tags: ["leetcode", "困难", "数组", "前缀和"]
math: true
---

You are given an array `nums`. You can rotate it by a non-negative integer `k` so that the array becomes `[nums[k], nums[k + 1], ... nums[nums.length - 1], nums[0], nums[1], ..., nums[k-1]]`. Afterward, any entries that are less than or equal to their index are worth one point.

- For example, if we have `nums = [2,4,1,3,0]`, and we rotate by `k = 2`, it becomes `[1,3,0,2,4]`. This is worth `3` points because `1 > 0` [no points], `3 > 1` [no points], `0 <= 2` [one point], `2 <= 3` [one point], `4 <= 4` [one point].

Return _the rotation index_ `k` _that corresponds to the highest score we can achieve if we rotated_ `nums` _by it_. If there are multiple answers, return the smallest such index `k`.

<!--more-->

**Example 1:**

> **Input:** nums = [2,3,1,4,0]
>
> **Output:** 3
>
> **Explanation:** Scores for each k are listed below:
> k = 0, nums = [2,3,1,4,0], score 2
> k = 1, nums = [3,1,4,0,2], score 3
> k = 2, nums = [1,4,0,2,3], score 3
> k = 3, nums = [4,0,2,3,1], score 4
> k = 4, nums = [0,2,3,1,4], score 3
> So we should choose k = 3, which has the highest score.

**Example 2:**

> **Input:** nums = [1,3,0,2,4]
>
> **Output:** 0
>
> **Explanation:** nums will always have 3 points no matter how it shifts.
> So we will choose the smallest k, which is 0.

**Constraints:**

- \\(1 <= nums.length <= 10^5\\)
- `0 <= nums[i] < nums.length`

## Solution

为了方便，令 n 为 nums 长度。

对于给定的 nums 而言，有效的轮调范围为 [0,n−1]，即对于任意 nums[i] 而言，可取的下标共有 n 种。

假定当前下标为 i，轮调次数为 k，那么轮调后下标为 i−k，当新下标为负数时，相当于 nums[i] 出现在比原数组更“靠后”的位置，此时下标等价于 (i−k+n)modn。

考虑什么情况下 nums[i] 能够得分？

首先新下标的取值范围为 [0,n−1]，即有 0⩽i−k⩽n−1 。由此可分析出 k 的取值范围为：

0⩽i−k⇔k⩽i

i−k⩽n−1⇔i−(n−1)⩽k

即由新下标取值范围可知 k 的上下界分别为 i 和 i−(n−1)。

同时为了满足得分定义，还有 nums[i]⩽i−k，进行变形可得：

nums[i]⩽i−k⇔k⩽i−nums[i]

此时我们有两个关于 k 的上界 k⩽i 和 k⩽i−nums[i]，由于 nums[i] 取值范围为 [0,n)，则有 i−nums[i]⩽i，由于必须同时满足「合法移动（有效下标）」和「能够得分」，我们仅考虑范围更小（更严格）由 nums[i]⩽i−k 推导而来的上界 k⩽i−nums[i] 即可。

综上，nums[i] 能够得分的 k 的取值范围为 [i−(n−1),i−nums[i]]。

最后考虑 [i−(n−1),i−nums[i]]（均进行加 n 模 n 转为正数）什么情况下为合法的连续段：

- 当 i−(n−1)⩽i−nums[i] 时，[i−(n−1),i−nums[i]] 为合法连续段；
- 当 i−(n−1)>i−nums[i] 时，根据负数下标等价于 (i−k+n)modn，此时 [i−(n−1),i−nums[i]] 等价于 [0,i−nums[i]] 和 [i−(n−1),n−1] 两段。

至此，我们分析出原数组的每个 nums[i] 能够得分的 k 的取值范围，假定取值范围为 [l,r]，我们可以对 [l,r] 进行 +1 标记，代表范围为 k 能够得 1 分，当处理完所有的 nums[i] 后，找到标记次数最多的位置 k 即是答案。

标记操作可使用「差分」实现，而找标记次数最多的位置可对差分数组求前缀和再进行遍历即可。

```cpp
class Solution {
public:
    int bestRotation(vector<int>& nums) {
        const int n = nums.size();
        vector<int> k_score_diff(n + 1, 0);
        for(int i = 0; i < n; i++) {
            int low  = (i + 1) % n;
            int high = (i - nums[i] + n) % n;
            k_score_diff[low]++;
            k_score_diff[high + 1]--;
            if(low > high) {
                k_score_diff[0]++;
                k_score_diff[n]--;
            }
        }
        int ans       = 0;
        int max_score = 0;
        int score     = 0;
        for(int i = 0; i < n; i++) {
            score += k_score_diff[i];
            if(score > max_score) {
                ans       = i;
                max_score = score;
            }
        }
        return ans;
    }
};
```
