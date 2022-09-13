---
title: "LeetCode 174. Dungeon Game"
date: 2022-09-13T17:00:13+08:00
draft: false
tags: ["leetcode", "困难", "数组", "动态规划","矩阵"]
math: true
---

The demons had captured the princess and imprisoned her in **the bottom-right corner** of a `dungeon`. The `dungeon` consists of `m x n` rooms laid out in a 2D grid. Our valiant knight was initially positioned in **the top-left room** and must fight his way through `dungeon` to rescue the princess.

The knight has an initial health point represented by a positive integer. If at any point his health point drops to `0` or below, he dies immediately.

Some of the rooms are guarded by demons (represented by negative integers), so the knight loses health upon entering these rooms; other rooms are either empty (represented as 0) or contain magic orbs that increase the knight's health (represented by positive integers).

To reach the princess as quickly as possible, the knight decides to move only **rightward** or **downward** in each step.

Return *the knight's minimum initial health so that he can rescue the princess*.

<!--more-->

**Note** that any room can contain threats or power-ups, even the first room the knight enters and the bottom-right room where the princess is imprisoned.

**Example 1:**

![dungeon-grid-1](/images/dungeon-grid-1.jpg)

    Input: dungeon = [[-2,-3,3],[-5,-10,1],[10,30,-5]]
    Output: 7
    Explanation: The initial health of the knight must be at least 7 if he follows the optimal path: RIGHT-> RIGHT -> DOWN -> DOWN.

**Example 2:**

    Input: dungeon = [[0]]
    Output: 1

**Constraints:**

- `m == dungeon.length`
- `n == dungeon[i].length`
- `1 <= m, n <= 200`
- `-1000 <= dungeon[i][j] <= 1000`

## Solution

#### 方法一：动态规划

**思路及算法**

几个要素：「\\(M \times N\\) 的网格」「每次只能向右或者向下移动一步」，让人很容易想到该题使用动态规划的方法。

但是我们发现，如果按照从左上往右下的顺序进行动态规划，对于每一条路径，我们需要同时记录两个值。第一个是「从出发点到当前点的路径和」，第二个是「从出发点到当前点所需的最小初始值」。而这两个值的重要程度相同，参看下面的示例：

![fig1](/images/174_fig1.png)

从 \\((0,0)\\) 到 \\((1,2)\\) 有多条路径，我们取其中最有代表性的两条：

![fig2](/images/174_fig2.png)

- 绿色路径「从出发点到当前点的路径和」为 \\(1\\)，「从出发点到当前点所需的最小初始值」为 \\(3\\)。

- 蓝色路径「从出发点到当前点的路径和」为 \\(-1\\)，「从出发点到当前点所需的最小初始值」为 \\(2\\)。

我们希望「从出发点到当前点的路径和」尽可能大，而「从出发点到当前点所需的最小初始值」尽可能小。这两条路径各有优劣。

在上图中，我们知道应该选取绿色路径，因为蓝色路径的路径和太小，使得蓝色路径需要增大初始值到 \\(4\\) 才能走到终点，而绿色路径只要 \\(3\\) 点初始值就可以直接走到终点。但是如果把终点的 \\(-2\\) 换为 \\(0\\)，蓝色路径只需要初始值 \\(2\\)，绿色路径仍然需要初始值 \\(3\\)，最优决策就变成蓝色路径了。

因此，如果按照从左上往右下的顺序进行动态规划，我们无法直接确定到达 \\((1,2)\\) 的方案，因为有两个重要程度相同的参数同时影响后续的决策。也就是说，这样的动态规划是不满足「无后效性」的。

于是我们考虑从右下往左上进行动态规划。令 \\(\textit{dp}[i][j]\\) 表示从坐标 \\((i,j)\\) 到终点所需的最小初始值。换句话说，当我们到达坐标 \\((i,j)\\) 时，如果此时我们的路径和不小于 \\(\textit{dp}[i][j]\\)，我们就能到达终点。

这样一来，我们就无需担心路径和的问题，只需要关注最小初始值。对于 \\(\textit{dp}[i][j]\\)，我们只要关心 \\(\textit{dp}[i][j+1]\\) 和 \\(\textit{dp}[i+1][j]\\) 的最小值 \\(\textit{minn}\\)。记当前格子的值为 \\(\textit{dungeon}(i,j)\\)，那么在坐标 \\((i,j)\\) 的初始值只要达到 \\(\textit{minn}-\textit{dungeon}(i,j)\\) 即可。同时，初始值还必须大于等于 \\(1\\)。这样我们就可以得到状态转移方程：

$$
\textit{dp}[i][j] = \max(\min(\textit{dp}[i+1][j], \textit{dp}[i][j + 1]) - \textit{dungeon}(i, j), 1)
$$

最终答案即为 \\(\textit{dp}[0][0]\\)。

边界条件为，当 \\(i=n-1\\) 或者 \\(j=m-1\\) 时，\\(\textit{dp}[i][j]\\) 转移需要用到的 \\(\textit{dp}[i][j+1]\\) 和 \\(\textit{dp}[i+1][j]\\) 中有无效值，因此代码实现中给无效值赋值为极大值。特别地，\\(\textit{dp}[n-1][m-1]\\) 转移需要用到的 \\(\textit{dp}[n-1][m]\\) 和 \\(\textit{dp}[n][m-1]\\) 均为无效值，因此我们给这两个值赋值为 \\(1\\)。

**代码**

```cpp
class Solution {
public:
    int calculateMinimumHP(vector<vector<int>>& dungeon) {
        int n = dungeon.size(), m = dungeon[0].size();
        vector<vector<int>> dp(n + 1, vector<int>(m + 1, INT_MAX));
        dp[n][m - 1] = dp[n - 1][m] = 1;
        for (int i = n - 1; i >= 0; --i) {
            for (int j = m - 1; j >= 0; --j) {
                int minn = min(dp[i + 1][j], dp[i][j + 1]);
                dp[i][j] = max(minn - dungeon[i][j], 1);
            }
        }
        return dp[0][0];
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(N \times M)\\)，其中 \\(N,M\\) 为给定矩阵的长宽。

- 空间复杂度：\\(O(N \times M)\\)，其中 \\(N,M\\) 为给定矩阵的长宽，注意这里可以利用滚动数组进行优化，优化后空间复杂度可以达到 \\(O(N)\\)。
