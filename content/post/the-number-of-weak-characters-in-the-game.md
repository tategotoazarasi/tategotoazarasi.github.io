---
title: "LeetCode 1996. 游戏中弱角色的数量"
date: 2022-01-28T10:27:51+08:00
draft: false
tags: ["leetcode", "中等", "栈", "贪心", "数组", "排序", "单调栈"]
math: true
---

你正在参加一个多角色游戏，每个角色都有两个主要属性：**攻击** 和 **防御** 。给你一个二维整数数组 `properties` ，其中 \\(properties[i] = [attack_i, defense_i]\\) 表示游戏中第 `i` 个角色的属性。

如果存在一个其他角色的攻击和防御等级 **都严格高于** 该角色的攻击和防御等级，则认为该角色为 **弱角色** 。更正式地，如果认为角色 `i` **弱于** 存在的另一个角色 `j` ，那么 \\(attack_j > attack_i\\) 且 \\(defense_j > defense_i\\) 。

返回 **弱角色** 的数量。

<!--more-->

**示例 1：**

> **输入：** properties = [[5,5],[6,3],[3,6]]
>
> **输出：** 0
>
> **解释：** 不存在攻击和防御都严格高于其他角色的角色。

**示例 2：**

> **输入：** properties = [[2,2],[3,3]]
>
> **输出：** 1
>
> **解释：** 第一个角色是弱角色，因为第二个角色的攻击和防御严格大于该角色。

**示例 3：**

> **输入：** properties = [[1,5],[10,4],[4,3]]
>
> **输出：** 1
>
> **解释：** 第三个角色是弱角色，因为第二个角色的攻击和防御严格大于该角色。

**提示：**

- \\(2 <= properties.length <= 10^5\\)
- `properties[i].length == 2`
- \\(1 <= attacki, defensei <= 10^5\\)

## 题解

**思路**

对于每个角色，我们需要判断是否存在一个攻击值和防御值都高于它的角色，从而确定该角色是否为弱角色。我们可以按角色的某个属性（比如攻击值）从大到小的顺序遍历，同时记录已经遍历过的角色防御值的最大值 maxDef。对于当前角色 p，如果 p 的防御值严格小于 maxDef，那么说明存在防御值比 p 高的角色（记作 q），如果此时 q 的攻击值也严格大于 p，则可以确定 p 为弱角色。

如何保证当 q 的防御值 maxDef 严格大于 p 的防御值时，q 的攻击值一定大于 p 的攻击值，这是比较难处理的一点，因为可能存在攻击值相同的角色。

- 最简单的处理办法就是将攻击值相同的角色进行单独分组，严格保证攻击值相同的角色被分到同一组，遍历时记录攻击值严格大于当前分组且防御值最大的角色 q。遍历当前分组时，如果发现角色 p 的防御值严格小于 q 的防御值，则此时可以肯定角色 p 属于弱角色。
- 实际处理时，对于攻击值相同的角色，我们按照其防御值从小到大进行排序且按照攻击值从大到小开始遍历，这样就可以保证当前已经遍历过的最大防御值角色 q 的防御值 maxDef 严格大于当前角色 p 的防御值时，则此时 q 的攻击值一定严格大于 p 的攻击值。因为相同的攻击值按照防御值从大到小进行排列，如果出现已经遍历过的角色 q 的防御值大于 p 的防御值，则此时我们可以肯定可以推理出角色 q 与角色 p 攻击值一定不相同。

**复杂度分析**

- 时间复杂度：O(nlogn)，其中 n 为数组的长度。排序的时间复杂度为 O(nlogn)，遍历数组的时间为 O(n)，总的时间复杂度为 O(nlogn+n)=O(nlogn)。

- 空间复杂度：O(logn)，其中 n 为数组的长度。排序时使用的栈空间为 O(logn)。

```cpp
class Solution {
public:
    int numberOfWeakCharacters(vector<vector<int>>& properties) {
        sort(properties.begin(), properties.end(), [](const vector<int> & a, const vector<int> & b) {
            return a[0] == b[0] ? (a[1] < b[1]) : (a[0] > b[0]);
        });

        int maxDef = 0;
        int ans = 0;
        for (auto & p : properties) {
            if (p[1] < maxDef) {
                ans++;
            } else {
                maxDef = p[1];
            }
        }
        return ans;
    }
};
```
