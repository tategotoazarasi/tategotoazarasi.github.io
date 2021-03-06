---
title: "LeetCode 1414. 和为 K 的最少斐波那契数字数目"
date: 2022-02-03T20:03:54+08:00
draft: false
tags: ["leetcode", "中等", "贪心"]
math: true
---

给你数字 `k` ，请你返回和为  `k`  的斐波那契数字的最少数目，其中，每个斐波那契数字都可以被使用多次。

斐波那契数字定义为：

- \\(F_1 = 1\\)
- \\(F_2 = 1\\)
- \\(F_n = F\_{n-1} + F\_{n-2}\\)， 其中 n > 2 。

数据保证对于给定的 `k` ，一定能找到可行解。

<!--more-->

**示例 1：**

> **输入：** k = 7
>
> **输出：** 2
>
> **解释：** 斐波那契数字为：1，1，2，3，5，8，13，……
> 对于 k = 7 ，我们可以得到 2 + 5 = 7 。

**示例 2：**

> **输入：** k = 10
>
> **输出：** 2
>
> **解释：** 对于 k = 10 ，我们可以得到 2 + 8 = 10 。

**示例 3：**

> **输入：** k = 19
>
> **输出：** 3
>
> **解释：** 对于 k = 19 ，我们可以得到 1 + 5 + 13 = 19 。

**提示：**

- \\(1 <= k <= 10^9\\)

## 题解

**思路**

首先找到所有不超过 k 的斐波那契数字，然后每次贪心地选取不超过 k 的最大斐波那契数字，将 k 减去该斐波那契数字，重复该操作直到 k 变为 0，此时选取的斐波那契数字满足和为 k 且数目最少。

**证明**

为了证明上述方案选取的斐波那契数字数目最少，只需要证明存在一种选取斐波那契数字数目最少的方案，该方案选取了不超过 k 的最大斐波那契数字。

1. 当选取斐波那契数字数目最少时，不可能选取两个相邻的斐波那契数。

   假设选取了两个相邻的斐波那契数字 \\(F_x\\) 和 \\(F\_{x+1}\\)，则根据斐波那契数字的定义，这两个斐波那契数字之和为后一个斐波那契数字：

   \\(F\_{x + 2} = F_x + F\_{x + 1}\\)

   因此可以用 \\(Fx+2\\) 代替 \\(Fx\\) 和 \\(Fx+1\\)，选取的斐波那契数字的总和不变，选取的斐波那契数字的数目减少 1 个，比选取 \\(Fx\\) 和 \\(Fx+1\\) 的方案更优。

2. 一定存在一种选取斐波那契数字数目最少的方案，使得选取的每个斐波那契数字各不相同。

   假设 \\(F_x\\) 被选取了两次。当 x≤2 时，\\(F_x=1\\)，可以用 \\(F_3=2\\) 代替两个 \\(F_x\\)，此时选取的斐波那契数字的数目减少 1 个。当 x>2 时，存在以下关系：

   \\( 2 \times F_x = (F\_{x - 2} + F\_{x - 1}) + F_x = F\_{x - 2} + (F\_{x - 1} + F_x) = F\_{x - 2} + F\_{x + 1} \\)

   因此当 x>2 时，如果 \\(F_x\\) 被选取了两次，则可以换成 \\(F\_{x−2}\\) 和 \\(F\_{x+1}\\)。

3. 根据上述两个结论，必须选取不超过 k 的最大斐波那契数字，才能使得选取的斐波那契数字满足和为 k 且数目最少。

   用 \\(F_m\\) 表示不超过 k 的最大斐波那契数字。如果不选择 \\(F_m\\)，则考虑选取的斐波那契数字之和可能的最大值，记为 N。根据上述两个结论，选取的斐波那契数字中不存在相邻的斐波那契数字，也不存在重复的斐波那契数字，因此可以得到 N 的表达式：

   \\( N = \begin{cases} F\_{m - 1} + F\_{m - 3} + \ldots + F_4 + F_2, &m~是奇数 \\ F\_{m - 1} + F\_{m - 3} + \ldots + F_3 + F_1, &m~是偶数 \end{cases} \\)

   当 m 是奇数时，N 的值计算如下：

   \\(N = F\_{m - 1} + F\_{m - 3} + \ldots + F_4 + F_2\\)

   \\(= F\_{m - 1} + F\_{m - 3} + \ldots + F_4 + F_2 + F_1 - F_1\\)

   \\(= F\_{m - 1} + F\_{m - 3} + \ldots + F_4 + F_3 - F_1\\)

   \\(= F\_{m - 1} + F\_{m - 3} + \ldots + F_5 - F_1\\)

   \\(= \ldots\\)

   \\(= F\_{m - 1} + F\_{m - 2} - F_1\\)

   \\(= F_m - 1\\)

   \\(< F_m\\)

   此时 \\(N<F_m\\)，由于 \\(F_m \le k\\)，因此 N<k。如果不选择 \\(F_m\\)，则选取的斐波那契数字之和一定小于 k，因此必须选择 \\(F_m\\)。

   当 m 是偶数时，N 的值计算如下：

   \\(N = F\_{m - 1} + F\_{m - 3} + \ldots + F_3 + F_1\\)

   \\(= F\_{m - 1} + F\_{m - 3} + \ldots + F_3 + F_2\\)

   \\(= F\_{m - 1} + F\_{m - 3} + \ldots + F_4\\)

   \\(= \ldots\\)

   \\(= F\_{m - 1} + F\_{m - 2}\\)

   \\(= F_m\\)

   此时 \\(N=F_m\\)，2m 个斐波那契数字之和等于 \\(F_m\\)，用一个 \\(F_m\\) 替换 2m 个斐波那契数字，选取的斐波那契数字数目不变或减少（只有当 m=2 时，选取的斐波那契数字数目不变）。

   综上所述，无论 m 是奇数还是偶数，都需要选取 \\(F_m\\)，即不超过 k 的最大斐波那契数字，才能使得选取的斐波那契数字满足和为 k 且数目最少。

**复杂度分析**

- 时间复杂度：O(logk)，其中 k 为给定的整数。需要找到所有不超过 k 的斐波那契数字，然后计算和为 k 的最少斐波那契数字数目，不超过 k 的斐波那契数字的个数是 O(logk) 个。

- 空间复杂度：O(logk)，其中 k 为给定的整数。需要 O(logk) 的空间存储所有不超过 k 的斐波那契数字。

```cpp
class Solution {
public:
    int find_min(set<int, greater<>> &fibb, int k, set<int, greater<>>::iterator begin) {
        const auto i = lower_bound(begin, fibb.end(), k, greater<int>());
        if(k == *i) {
            return 1;
        }
        return 1 + find_min(fibb, k - *i, i);
    }

    int findMinFibonacciNumbers(int k) {
        auto fibb = set<int, greater<>>();
        fibb.insert(1);
        int prev_1 = 1;
        int next   = 2;
        while(next <= k) {
            fibb.insert(next);
            const int prev_2 = prev_1;
            prev_1           = next;
            next             = prev_1 + prev_2;
        }
        return find_min(fibb, k, fibb.begin());
    }
};
```
