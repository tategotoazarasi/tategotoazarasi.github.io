---
title: "LeetCode 6011. 完成比赛的最少时间"
date: 2022-02-27T16:35:07+08:00
draft: false
tags: ["leetcode", "困难", "动态规划"]
math: true
---

给你一个下标从 **0**  开始的二维整数数组  `tires` ，其中  \\(tires[i] = [f_i, r_i]\\)  表示第  `i`  种轮胎如果连续使用，第  `x`  圈需要耗时  \\(f_i \* r_i^{(x-1)}\\)  秒。

- 比方说，如果  \\(f_i = 3\\)  且  \\(r_i = 2\\) ，且一直使用这种类型的同一条轮胎，那么该轮胎完成第  `1`  圈赛道耗时 `3`  秒，完成第 `2`  圈耗时  `3 * 2 = 6`  秒，完成第 `3`  圈耗时  \\(3 \* 2^2 = 12\\\)  秒，依次类推。

同时给你一个整数  `changeTime`  和一个整数  `numLaps` 。

比赛总共包含  `numLaps`  圈，你可以选择 **任意**  一种轮胎开始比赛。每一种轮胎都有 **无数条** 。每一圈后，你可以选择耗费 `changeTime`  秒 **换成**  任意一种轮胎（也可以换成当前种类的新轮胎）。

请你返回完成比赛需要耗费的 **最少**  时间。

<!--more-->

**示例 1：**

> **输入：** tires = [[2,3],[3,4]], changeTime = 5, numLaps = 4
>
> **输出：** 21
>
> **解释：**
> 第 1 圈：使用轮胎 0 ，耗时 2 秒。
> 第 2 圈：继续使用轮胎 0 ，耗时 2 _ 3 = 6 秒。
> 第 3 圈：耗费 5 秒换一条新的轮胎 0 ，然后耗时 2 秒完成这一圈。
> 第 4 圈：继续使用轮胎 0 ，耗时 2 _ 3 = 6 秒。
> 总耗时 = 2 + 6 + 5 + 2 + 6 = 21 秒。
> 完成比赛的最少时间为 21 秒。

**示例 2：**

> **输入：** tires = [[1,10],[2,2],[3,4]], changeTime = 6, numLaps = 5
>
> **输出：** 25
>
> **解释：**
> 第 1 圈：使用轮胎 1 ，耗时 2 秒。
> 第 2 圈：继续使用轮胎 1 ，耗时 2 _ 2 = 4 秒。
> 第 3 圈：耗时 6 秒换一条新的轮胎 1 ，然后耗时 2 秒完成这一圈。
> 第 4 圈：继续使用轮胎 1 ，耗时 2 _ 2 = 4 秒。
> 第 5 圈：耗时 6 秒换成轮胎 0 ，然后耗时 1 秒完成这一圈。
> 总耗时 = 2 + 4 + 6 + 2 + 4 + 6 + 1 = 25 秒。
> 完成比赛的最少时间为 25 秒。

**提示：**

- \\(1 <= tires.length <= 10^5\\)
- `tires[i].length == 2`
- \\(1 <= f_i, changeTime <= 10^5\\)
- \\(2 <= r_i <= 10^5\\)
- `1 <= numLaps <= 1000`
- 在不更换轮胎的情况下，我们在赛道上绕行的最大次数是多少？
- 我们能预先计算在不更换轮胎的情况下绕赛道 x 圈的最短时间吗？
- 我们能不能用动态规划的方法，利用预先计算的数值有效地解决这个问题？

## 题解

**提示**

因为第 x 圈需要耗时 \\(f_i \times r_i^{(x-1)}\\) 秒，可以预感到一个轮胎不会用太久（指数膨胀的规律）。

**解法：动态规划**

注意到，最优解法中，一个轮胎不会连续使用超过 19 天。因为题目规定 \\(r_i \ge 2\\)，而当 x>19 时，有 \\(f_i \times r_i^{18} \ge 1 \times 2^{18} = 262144\\)，而换一个新轮胎的花费最大为 \\(changeTime + f_i \le 2\times 10 ^ 5\\)，因此肯定不如换一个新轮胎划算。

定义 mintimes[i]= 用一个轮胎跑 i 圈的最小总花费时间（这里假设用第一个轮胎前也需要更换），则我们可以直接枚举轮胎，然后根据定义来求出 mintimes[i] 即可。

然后，再定义 dp[i]= 跑 i 圈需要的最小花费时间。则状态转移非常简单：

\\(dp[i] = \min\_{1 \le j \le 19} (dp[i - j] + mintimes[j])\\)

最后，答案就是 dp[numLaps]−changeTime。这是因为之前我们假设用第一个轮胎之前也需要换轮胎的时间，最后答案需要去掉它。

时间复杂度：\\(O((n+changeTime)log(M)), M = 2 \times 10^5\\)

```cpp
class Solution {
public:
    int minimumFinishTime(vector<vector<int>>& tires, int changeTime, int numLaps) {
        /* min_times[i] = 用一个轮胎跑 i 圈的最小花费 */
        vector<long long> min_times(20, 1e9);
        for(auto &v: tires) {
            long long f       = v[0];
            long long r       = v[1];
            long long cost    = f + changeTime;///< 第一次用该轮胎的耗费
            long long current = f;
            long long sum     = cost;
            for(int i = 1; i <= 19; i++) {
                min_times[i] = min(min_times[i], sum);
                current *= r;
                if(current > cost) {
                    current = f;
                    sum += cost;
                } else {
                    sum += current;
                }
            }
        }
        /* dp[i] = 跑 i 圈的最小花费 */
        vector<long long> dp(numLaps + 1, 1e9);
        dp[0] = 0;
        for(int i = 1; i <= numLaps; i++) {
            for(int j = 1; j <= min(19, i); j++) {
                dp[i] = min(dp[i], dp[i - j] + min_times[j]);
            }
        }
        /* 最后需要减去一次换轮胎的时间 */
        return dp[numLaps] - changeTime;
    }
};
```
