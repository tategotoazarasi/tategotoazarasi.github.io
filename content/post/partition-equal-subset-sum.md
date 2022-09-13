---
title: "LeetCode 416. 分割等和子集"
date: 2022-09-13T15:17:46+08:00
draft: false
tags: ["leetcode", "中等", "数组", "动态规划"]
math: true
---

给你一个 **只包含正整数** 的 **非空** 数组  `nums`。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。

<!--more-->

**示例 1：**

    输入：nums = [1,5,11,5]
    输出：true
    解释：数组可以分割成 [1, 5, 5] 和 [11] 。

**示例 2：**

    输入：nums = [1,2,3,5]
    输出：false
    解释：数组不能分割成两个元素和相等的子集。

**提示：**

- `1 <= nums.length <= 200`
- `1 <= nums[i] <= 100`

## 题解

#### 前言

**作者在这里希望读者认真阅读前言部分。**

本题是经典的「[NP 完全问题](https://baike.baidu.com/item/NP%E5%AE%8C%E5%85%A8%E9%97%AE%E9%A2%98)」，也就是说，如果你发现了该问题的一个[多项式算法](https://baike.baidu.com/item/%E5%A4%9A%E9%A1%B9%E5%BC%8F%E7%AE%97%E6%B3%95)，那么恭喜你证明出了 P=NP，可以期待一下图灵奖了。

正因如此，我们不应期望该问题有多项式时间复杂度的解法。我们能想到的，例如基于贪心算法的「将数组降序排序后，依次将每个元素添加至当前元素和较小的子集中」之类的方法都是错误的，可以轻松地举出反例。因此，我们必须尝试非多项式时间复杂度的算法，例如时间复杂度与元素大小相关的**动态规划**。

#### 方法一：动态规划

**思路与算法**

这道题可以换一种表述：给定一个只包含正整数的非空数组 \\(\textit{nums}[0]\\)，判断是否可以从数组中选出一些数字，使得这些数字的和等于整个数组的元素和的一半。因此这个问题可以转换成「\\(0-1\\) 背包问题」。这道题与传统的「\\(0-1\\) 背包问题」的区别在于，传统的「\\(0-1\\) 背包问题」要求选取的物品的重量之和**不能超过**背包的总容量，这道题则要求选取的数字的和**恰好等于**整个数组的元素和的一半。类似于传统的「\\(0-1\\) 背包问题」，可以使用动态规划求解。

在使用动态规划求解之前，首先需要进行以下判断。

- 根据数组的长度 \\(n\\) 判断数组是否可以被划分。如果 \\(n<2\\)，则不可能将数组分割成元素和相等的两个子集，因此直接返回 \\(\text{false}\\)。

- 计算整个数组的元素和 \\(\textit{sum}\\) 以及最大元素 \\(\textit{maxNum}\\)。如果 \\(\textit{sum}\\) 是奇数，则不可能将数组分割成元素和相等的两个子集，因此直接返回 \\(\text{false}\\)。如果 \\(\textit{sum}\\) 是偶数，则令 \\(\textit{target}=\frac{\textit{sum}}{2}\\)，需要判断是否可以从数组中选出一些数字，使得这些数字的和等于 \\(\textit{target}\\)。如果 \\(\textit{maxNum}>\textit{target}\\)，则除了 \\(\textit{maxNum}\\) 以外的所有元素之和一定小于 \\(\textit{target}\\)，因此不可能将数组分割成元素和相等的两个子集，直接返回 \\(\text{false}\\)。

创建二维数组 \\(\textit{dp}\\)，包含 \\(n\\) 行 \\(\textit{target}+1\\) 列，其中 \\(\textit{dp}[i][j]\\) 表示从数组的 \\([0,i]\\) 下标范围内选取若干个正整数（可以是 \\(0\\) 个），是否存在一种选取方案使得被选取的正整数的和等于 \\(j\\)。初始时，\\(\textit{dp}\\) 中的全部元素都是 \\(\text{false}\\)。

在定义状态之后，需要考虑边界情况。以下两种情况都属于边界情况。

- 如果不选取任何正整数，则被选取的正整数等于 \\(0\\)。因此对于所有 \\(0 \le i < n\\)，都有 \\(\textit{dp}[i][0]=\text{true}\\)。

- 当 \\(i==0\\) 时，只有一个正整数 \\(\textit{nums}[0]\\) 可以被选取，因此 \\(\textit{dp}[0]\textit{nums}[0]]=\text{true}\\)。

对于 \\(i>0\\) 且 \\(j>0\\) 的情况，如何确定 \\(\textit{dp}[i][j]\\) 的值？需要分别考虑以下两种情况。

- 如果 \\(j \ge \textit{nums}[i]\\)，则对于当前的数字 \\(\textit{nums}[i]\\)，可以选取也可以不选取，两种情况只要有一个为 \\(\text{true}\\)，就有 \\(\textit{dp}[i][j]=\text{true}\\)。

  - 如果不选取 \\(\textit{nums}[i]\\)，则 \\(\textit{dp}[i][j]=\textit{dp}[i-1][j]\\)；
  - 如果选取 \\(\textit{nums}[i]\\)，则 \\(\textit{dp}[i][j]=\textit{dp}[i-1]j-\textit{nums}[i]]\\)。

- 如果 \\(j < \textit{nums}[i]\\)，则在选取的数字的和等于 \\(j\\) 的情况下无法选取当前的数字 \\(\textit{nums}[i]\\)，因此有 \\(\textit{dp}[i][j]=\textit{dp}[i-1][j]\\)。

状态转移方程如下：

$$
\textit{dp}[i][j]=\begin{cases}
\textit{dp}[i-1][j]~|~\textit{dp}[i-1][j-\textit{nums}[i]], & j \ge \textit{nums}[i] \\\\
\textit{dp}[i-1][j], & j < \textit{nums}[i]
\end{cases}
$$

最终得到 \\(\textit{dp}[n-1][\textit{target}]\\) 即为答案。

![ppt1](/images/416_1.png)

![ppt2](/images/416_2.png)

![ppt3](/images/416_3.png)

![ppt4](/images/416_4.png)

![ppt5](/images/416_5.png)

![ppt6](/images/416_6.png)

![ppt7](/images/416_7.png)

![ppt8](/images/416_8.png)

![ppt9](/images/416_9.png)

![ppt10](/images/416_10.png)

![ppt11](/images/416_11.png)

![ppt12](/images/416_12.png)

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();
        if (n < 2) {
            return false;
        }
        int sum = accumulate(nums.begin(), nums.end(), 0);
        int maxNum = *max_element(nums.begin(), nums.end());
        if (sum & 1) {
            return false;
        }
        int target = sum / 2;
        if (maxNum > target) {
            return false;
        }
        vector<vector<int>> dp(n, vector<int>(target + 1, 0));
        for (int i = 0; i < n; i++) {
            dp[i][0] = true;
        }
        dp[0][nums[0]] = true;
        for (int i = 1; i < n; i++) {
            int num = nums[i];
            for (int j = 1; j <= target; j++) {
                if (j >= num) {
                    dp[i][j] = dp[i - 1][j] | dp[i - 1][j - num];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[n - 1][target];
    }
};
```

上述代码的空间复杂度是 \\(O(n \times \textit{target})\\)。但是可以发现在计算 \\(\textit{dp}\\) 的过程中，每一行的 \\(dp\\) 值都只与上一行的 \\(dp\\) 值有关，因此只需要一个一维数组即可将空间复杂度降到 \\(O(\textit{target})\\)。此时的转移方程为：

$$
\textit{dp}[j]=\textit{dp}[j]\ |\ dp[j-\textit{nums}[i]]
$$

且需要注意的是第二层的循环我们需要**从大到小计算**，因为如果我们从小到大更新 \\(\textit{dp}\\) 值，那么在计算 \\(\textit{dp}[j]\\) 值的时候，\\(\textit{dp}[j-\textit{nums}[i]]\\) 已经是被更新过的状态，不再是上一行的 \\(\textit{dp}\\) 值。

**代码**

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int n = nums.size();
        if (n < 2) {
            return false;
        }
        int sum = 0, maxNum = 0;
        for (auto& num : nums) {
            sum += num;
            maxNum = max(maxNum, num);
        }
        if (sum & 1) {
            return false;
        }
        int target = sum / 2;
        if (maxNum > target) {
            return false;
        }
        vector<int> dp(target + 1, 0);
        dp[0] = true;
        for (int i = 0; i < n; i++) {
            int num = nums[i];
            for (int j = target; j >= num; --j) {
                dp[j] |= dp[j - num];
            }
        }
        return dp[target];
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(n \times \textit{target})\\)，其中 \\(n\\) 是数组的长度，\\(\textit{target}\\) 是整个数组的元素和的一半。需要计算出所有的状态，每个状态在进行转移时的时间复杂度为 \\(O(1)\\)。

- 空间复杂度：\\(O(\textit{target})\\)，其中 \\(\textit{target}\\) 是整个数组的元素和的一半。空间复杂度取决于 \\(\textit{dp}\\) 数组，在不进行空间优化的情况下，空间复杂度是 \\(O(n \times \textit{target})\\)，在进行空间优化的情况下，空间复杂度可以降到 \\(O(\textit{target})\\)。
