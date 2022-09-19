---
title: "LeetCode 329. 矩阵中的最长递增路径"
date: 2022-09-19T13:41:55+08:00
draft: false
tags:
  [
    "leetcode",
    "困难",
    "深度优先搜索",
    "广度优先搜索",
    "图",
    "拓扑排序",
    "记忆化搜索",
    "数组",
    "动态规划",
    "矩阵",
  ]
math: true
---

给定一个  `m x n` 整数矩阵  `matrix` ，找出其中 **最长递增路径** 的长度。

对于每个单元格，你可以往上，下，左，右四个方向移动。 你 **不能** 在 **对角线** 方向上移动或移动到 **边界外**（即不允许环绕）。

<!--more-->

**示例 1：**

![grid1](/images/329_grid1.jpg)

    输入：matrix = [[9,9,4],[6,6,8],[2,1,1]]
    输出：4
    解释：最长递增路径为 [1, 2, 6, 9]。

**示例 2：**

![tmp-grid](/images/329_tmp-grid.jpg)

    输入：matrix = [[3,4,5],[3,2,6],[2,2,1]]
    输出：4
    解释：最长递增路径是 [3, 4, 5, 6]。注意不允许在对角线方向上移动。

**示例 3：**

    输入：matrix = [[1]]
    输出：1

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 200`
- \\(0 <= matrix[i][j] <= 2^{31} - 1\\)

## 题解

#### 方法一：记忆化深度优先搜索

将矩阵看成一个有向图，每个单元格对应图中的一个节点，如果相邻的两个单元格的值不相等，则在相邻的两个单元格之间存在一条从较小值指向较大值的有向边。问题转化成在有向图中寻找最长路径。

深度优先搜索是非常直观的方法。从一个单元格开始进行深度优先搜索，即可找到从该单元格开始的最长递增路径。对每个单元格分别进行深度优先搜索之后，即可得到矩阵中的最长递增路径的长度。

但是如果使用朴素深度优先搜索，时间复杂度是指数级，会超出时间限制，因此必须加以优化。

朴素深度优先搜索的时间复杂度过高的原因是进行了大量的重复计算，同一个单元格会被访问多次，每次访问都要重新计算。由于同一个单元格对应的最长递增路径的长度是固定不变的，因此可以使用记忆化的方法进行优化。用矩阵 \\(\textit{memo}\\) 作为缓存矩阵，已经计算过的单元格的结果存储到缓存矩阵中。

使用记忆化深度优先搜索，当访问到一个单元格 \\((i,j)\\) 时，如果 \\(\textit{memo}[i][j] \neq 0\\)，说明该单元格的结果已经计算过，则直接从缓存中读取结果，如果 \\(\textit{memo}[i][j]=0\\)，说明该单元格的结果尚未被计算过，则进行搜索，并将计算得到的结果存入缓存中。

遍历完矩阵中的所有单元格之后，即可得到矩阵中的最长递增路径的长度。

```cpp
class Solution {
public:
    static constexpr int dirs[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    int rows, columns;

    int longestIncreasingPath(vector< vector<int> > &matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0) {
            return 0;
        }
        rows = matrix.size();
        columns = matrix[0].size();
        auto memo = vector< vector<int> > (rows, vector <int> (columns));
        int ans = 0;
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < columns; ++j) {
                ans = max(ans, dfs(matrix, i, j, memo));
            }
        }
        return ans;
    }

    int dfs(vector< vector<int> > &matrix, int row, int column, vector< vector<int> > &memo) {
        if (memo[row][column] != 0) {
            return memo[row][column];
        }
        ++memo[row][column];
        for (int i = 0; i < 4; ++i) {
            int newRow = row + dirs[i][0], newColumn = column + dirs[i][1];
            if (newRow >= 0 && newRow < rows && newColumn >= 0 && newColumn < columns && matrix[newRow][newColumn] > matrix[row][column]) {
                memo[row][column] = max(memo[row][column], dfs(matrix, newRow, newColumn, memo) + 1);
            }
        }
        return memo[row][column];
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(mn)\\)，其中 \\(m\\) 和 \\(n\\) 分别是矩阵的行数和列数。深度优先搜索的时间复杂度是 \\(O(V+E)\\)，其中 \\(V\\) 是节点数，\\(E\\) 是边数。在矩阵中，\\(O(V)=O(mn)\\)，\\(O(E)\approx O(4mn) = O(mn)\\)。

- 空间复杂度：\\(O(mn)\\)，其中 \\(m\\) 和 \\(n\\) 分别是矩阵的行数和列数。空间复杂度主要取决于缓存和递归调用深度，缓存的空间复杂度是 \\(O(mn)\\)，递归调用深度不会超过 \\(mn\\)。

#### 方法二：拓扑排序

从方法一可以看到，每个单元格对应的最长递增路径的结果只和相邻单元格的结果有关，那么是否可以使用动态规划求解？

根据方法一的分析，动态规划的状态定义和状态转移方程都很容易得到。方法一中使用的缓存矩阵 \\(\textit{memo}\\) 即为状态值，状态转移方程如下：

$$
\begin{aligned}
& \textit{memo}[i][j] = \max\{\textit{memo}[x][y]\} + 1 \\\\
& 其中~(x, y)~与~(i, j)~在矩阵中相邻，并且~\textit{matrix}[x][y] > \textit{matrix}[i][j]
\end{aligned}
$$

动态规划除了状态定义和状态转移方程，还需要考虑边界情况。这里的边界情况是什么呢？

如果一个单元格的值比它的所有相邻单元格的值都要大，那么这个单元格对应的最长递增路径是 \\(1\\)，这就是边界条件。这个边界条件并不直观，而是需要根据矩阵中的每个单元格的值找到作为边界条件的单元格。

仍然使用方法一的思想，将矩阵看成一个有向图，计算每个单元格对应的出度，即有多少条边从该单元格出发。对于作为边界条件的单元格，该单元格的值比所有的相邻单元格的值都要大，因此作为边界条件的单元格的出度都是 \\(0\\)。

基于出度的概念，可以使用拓扑排序求解。从所有出度为 \\(0\\) 的单元格开始广度优先搜索，每一轮搜索都会遍历当前层的所有单元格，更新其余单元格的出度，并将出度变为 \\(0\\) 的单元格加入下一层搜索。当搜索结束时，搜索的总层数即为矩阵中的最长递增路径的长度。

```cpp
class Solution {
public:
    static constexpr int dirs[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    int rows, columns;

    int longestIncreasingPath(vector< vector<int> > &matrix) {
        if (matrix.size() == 0 || matrix[0].size() == 0) {
            return 0;
        }
        rows = matrix.size();
        columns = matrix[0].size();
        auto outdegrees = vector< vector<int> > (rows, vector <int> (columns));
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < columns; ++j) {
                for (int k = 0; k < 4; ++k) {
                    int newRow = i + dirs[k][0], newColumn = j + dirs[k][1];
                    if (newRow >= 0 && newRow < rows && newColumn >= 0 && newColumn < columns && matrix[newRow][newColumn] > matrix[i][j]) {
                        ++outdegrees[i][j];
                    }
                }
            }
        }
        queue < pair<int, int> > q;
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < columns; ++j) {
                if (outdegrees[i][j] == 0) {
                    q.push({i, j});
                }
            }
        }
        int ans = 0;
        while (!q.empty()) {
            ++ans;
            int size = q.size();
            for (int i = 0; i < size; ++i) {
                auto cell = q.front(); q.pop();
                int row = cell.first, column = cell.second;
                for (int k = 0; k < 4; ++k) {
                    int newRow = row + dirs[k][0], newColumn = column + dirs[k][1];
                    if (newRow >= 0 && newRow < rows && newColumn >= 0 && newColumn < columns && matrix[newRow][newColumn] < matrix[row][column]) {
                        --outdegrees[newRow][newColumn];
                        if (outdegrees[newRow][newColumn] == 0) {
                            q.push({newRow, newColumn});
                        }
                    }
                }
            }
        }
        return ans;
    }
};
```

**复杂度分析**

- 时间复杂度：\\(O(mn)\\)，其中 \\(m\\) 和 \\(n\\) 分别是矩阵的行数和列数。拓扑排序的时间复杂度是 \\(O(V+E)\\)，其中 \\(V\\) 是节点数，\\(E\\) 是边数。在矩阵中，\\(O(V)=O(mn)\\)，\\(O(E)\approx O(4mn) = O(mn)\\)。

- 空间复杂度：\\(O(mn)\\)，其中 \\(m\\) 和 \\(n\\) 分别是矩阵的行数和列数。空间复杂度主要取决于队列，队列中的元素个数不会超过 \\(mn\\)。

#### 思考题

为了让大家更好地理解这道题，小编出了四道思考题，欢迎感兴趣的同学在评论区互动哦。

- 「方法一」中使用了记忆化存储和深度优先搜索，这里的深度优先搜索可以替换成广度优先搜索吗？

- 「方法二」中基于拓扑排序对排序后的有向无环图做了层次遍历，如果没有拓扑排序直接进行广度优先搜索会发生什么？

- 「方法二」中如果不使用拓扑排序，而是直接按照矩阵中元素的值**从大到小**进行排序，并依此顺序进行状态转移，那么可以得到正确的答案吗？如果是**从小到大**进行排序呢？

- 「变式」给定一个整数矩阵，找出符合以下条件的路径的数量：这个路径是严格递增的，且它的长度至少是 \\(3\\)。矩阵的边长最大为 \\(10^3\\)，答案对 \\(10^9 + 7\\) 取模。其他条件和题目相同。**思考：是否可以借鉴这道题的方法？**
