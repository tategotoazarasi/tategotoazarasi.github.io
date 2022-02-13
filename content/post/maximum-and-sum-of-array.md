---
title: "LeetCode 6007. 数组的最大与和"
date: 2022-02-13T15:34:49+08:00
draft: false
math: true
tags: ["leetcode", "困难", "动态规划"]
---

给你一个长度为  `n`  的整数数组  `nums`  和一个整数  `numSlots` ，满足`2 * numSlots >= n` 。总共有  `numSlots`  个篮子，编号为  `1`  到  `numSlots` 。

你需要把所有  `n`  个整数分到这些篮子中，且每个篮子 **至多**  有 2 个整数。一种分配方案的 **与和**  定义为每个数与它所在篮子编号的 **按位与运算**  结果之和。

- 比方说，将数字  `[1, 3]`  放入篮子  **_`1`_**  中，`[4, 6]` 放入篮子  **_`2`_**  中，这个方案的与和为  `(1 AND 1) + (3 AND 1) + (4 AND 2) + (6 AND 2) = 1 + 1 + 0 + 2 = 4` 。

请你返回将 `nums`  中所有数放入 `numSlots`  个篮子中的最大与和。

<!--more-->

**示例 1：**

**输入：** nums = [1,2,3,4,5,6], numSlots = 3
**输出：** 9
**解释：** 一个可行的方案是 [1, 4] 放入篮子 **_1_**  中，[2, 6] 放入篮子 **_2_**  中，[3, 5] 放入篮子 **_3_** 中。
最大与和为 (1 AND **_1_**) + (4 AND **_1_**) + (2 AND **_2_**) + (6 AND **_2_**) + (3 AND **_3_**) + (5 AND **_3_**) = 1 + 0 + 2 + 2 + 3 + 1 = 9 。

**示例 2：**

**输入：** nums = [1,3,10,4,7,1], numSlots = 9
**输出：** 24
**解释：** 一个可行的方案是 [1, 1] 放入篮子 **_1_** 中，[3] 放入篮子 **_3_** 中，[4] 放入篮子 **_4_** 中，[7] 放入篮子 **_7_** 中，[10] 放入篮子 **_9_**  中。
最大与和为 (1 AND **_1_**) + (1 AND **_1_**) + (3 AND **_3_**) + (4 AND **_4_**) + (7 AND **_7_**) + (10 AND **_9_**) = 1 + 1 + 3 + 4 + 7 + 8 = 24 。
注意，篮子 2 ，5 ，6 和 8 是空的，这是允许的。

**提示：**

- `n == nums.length`
- `1 <= numSlots <= 9`
- `1 <= n <= 2 * numSlots`
- `1 <= nums[i] <= 15`
- 你能想出一个解决这个问题的动态规划方法吗？
- 你能用位掩码来表示篮子的状态吗？
- 由于每个篮子至多可以放 222 个整数，我们可以视作有 \\( 2\cdot\textit{numSlots} \\) 个篮子。
- 数据范围很小，考虑状压 DP。
- 以谁作为状态定义的对象呢？注意篮子编号是不能变的，而 \\( 2\cdot\textit{numSlots} \\) 中元素的位置信息是不重要的。

## 题解

由于每个篮子至多可以放 2 个整数，我们可以视作有 \\( 2\cdot\textit{numSlots} \\) 个篮子。由于篮子个数很少，我们可以用二进制数 x 表示这 \\( 2\cdot\textit{numSlots} \\) 个篮子中放了数字的篮子集合，其中 x 从低到高的第 i 位为 1 表示第 i 个篮子放了数字，为 0 表示第 i 个篮子为空。

设 i 的二进制中的 1 的个数为 c，定义 f[i] 表示将 nums 的前 c 个数字放到篮子中，且放了数字的篮子集合为 i 时的最大与和。初始值 f[0]=0。

考虑将 nums[c] 放到一个空篮子时的状态转移方程（下标从 0 开始，此时 nums[c] 还没被放入篮中），我们可以枚举 i 中的 0，即空篮子的位置 j，该空篮子对应的编号为 \\( \dfrac{j}{2}+1 \\)，则有

\\( f[i+2^j] = \max(f[i+2^j],\ f[i] + (\dfrac{j}{2}+1)\&\textit{nums}[c]) \\)

设 nums 的长度为 n，最后答案为 \\( \max\_{c=n}(f) \\)。

代码实现时需要注意，若 \\( c\ge n \\) 则 f[i] 无法转移，需要跳过。

```cpp
class Solution {
public:
    int maximumANDSum(vector<int> &nums, int numSlots) {
        int ans = 0;
        vector<int> max_and_sum_of_status(1 << numSlots * 2);
        for(unsigned int status = 0; status < max_and_sum_of_status.size(); ++status) {
            //status:已经放置数字的篮子的集合
            const int one_count = __builtin_popcount(status);
            if(one_count >= nums.size()) {
                continue;
            }
            for(int next_slot = 0; next_slot < numSlots * 2; ++next_slot) {
                if((status & 1 << next_slot) == 0) {
                    // next_slot是空的
                    const int next_status              = status | 1 << next_slot;                                                                              ///< 加上next_slot之后的状态
                    auto slot_num                      = next_slot / 2 + 1;                                                                                    ///< slot编号
                    max_and_sum_of_status[next_status] = max(max_and_sum_of_status[next_status], max_and_sum_of_status[status] + (slot_num & nums[one_count]));//放置第onecount个数字
                    ans                                = max(ans, max_and_sum_of_status[next_status]);
                }
            }
        }
        return ans;
    }
};
```
