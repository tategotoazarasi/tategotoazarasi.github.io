---
title: execution-of-all-suffix-instructions-staying-in-a-grid
tags: [Import-7704]
created: '2021-12-26T07:10:59.796Z'
modified: '2021-12-30T13:58:01.038Z'
---

---
title: "LeetCode 5964. 执行所有后缀指令"
date: 2021-12-26T15:10:59+08:00
draft: false
tags: ["leetcode", "中等"]
math: true
---

现有一个 `n x n` 大小的网格，左上角单元格坐标 `(0, 0)` ，右下角单元格坐标 `(n - 1, n - 1)` 。给你整数 `n` 和一个整数数组 `startPos` ，其中 `startPos = [startrow, startcol]` 表示机器人最开始在坐标为 `(startrow, startcol)` 的单元格上。

另给你一个长度为 `m` 、下标从 **0** 开始的字符串 `s` ，其中 `s[i]` 是对机器人的第 `i` 条指令：`'L'`（向左移动），`'R'`（向右移动），`'U'`（向上移动）和 `'D'`（向下移动）。

机器人可以从 `s` 中的任一第 `i` 条指令开始执行。它将会逐条执行指令直到 `s` 的末尾，但在满足下述条件之一时，机器人将会停止：

- 下一条指令将会导致机器人移动到网格外。
- 没有指令可以执行。

返回一个长度为 `m` 的数组 `answer` ，其中 `answer[i]` 是机器人从第 `i`  条指令 **开始** ，可以执行的 **指令数目** 。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/leetcode_5964_1.png)

> **输入：** n = 3, startPos = [0,1], s = "RRDDLU"
>
> **输出：** [1,5,4,3,1,0]
>
> **解释：** 机器人从 startPos 出发，并从第 i 条指令开始执行：

- 0: "**_R_** RDDLU" 在移动到网格外之前，只能执行一条 "R" 指令。
- 1: "**_RDDLU_**" 可以执行全部五条指令，机器人仍在网格内，最终到达 (0, 0) 。
- 2: "**_DDLU_**" 可以执行全部四条指令，机器人仍在网格内，最终到达 (0, 0) 。
- 3: "**_DLU_**" 可以执行全部三条指令，机器人仍在网格内，最终到达 (0, 0) 。
- 4: "**_L_** U" 在移动到网格外之前，只能执行一条 "L" 指令。
- 5: "U" 如果向上移动，将会移动到网格外。

**示例 2：**

![](https://tategotoazarasi.github.io/images/leetcode_5964_2.png)

> **输入：** n = 2, startPos = [1,1], s = "LURD"
>
> **输出：** [4,1,0,0]
>
> **解释：**

- 0: "**_LURD_**"
- 1: "**_U_** RD"
- 2: "RD"
- 3: "D"

**示例 3：**

![](https://tategotoazarasi.github.io/images/leetcode_5964_3.png)

> **输入：** n = 1, startPos = [0,0], s = "LRUD"
>
> **输出：** [0,0,0,0]
>
> **解释：** 无论机器人从哪条指令开始执行，都会移动到网格外。

**提示：**

- `m == s.length`
- `1 <= n, m <= 500`
- `startPos.length == 2`
- `0 <= startrow, startcol < n`
- `s` 由 `'L'`、`'R'`、`'U'` 和 `'D'` 组成
- 这些约束不是很大。我们可以通过从s的每个索引开始模拟执行吗？
- 在满足任何一个停止条件之前，停止该指数的模拟，并设置该指数的答案。

```java
class Solution {
    public int[] executeInstructions(int n, int[] startPos, String s) {
        int[] ans = new int[s.length()];
        for (int i = 0; i < s.length(); i++) {
            int x = startPos[1];
            int y = startPos[0];
            for (int j = i; j < s.length(); j++) {
                switch (s.charAt(j)) {
                    case 'L':
                        x--;
                        break;
                    case 'R':
                        x++;
                        break;
                    case 'U':
                        y--;
                        break;
                    case 'D':
                        y++;
                        break;
                }

                if (x < 0 || x >= n || y < 0 || y >= n) {
                    ans[i] = j - i;
                    break;
                }
                if (j == s.length() - 1) {
                    ans[i] = s.length() - i;
                    break;
                }
            }
        }
        return ans;
    }
}
```
