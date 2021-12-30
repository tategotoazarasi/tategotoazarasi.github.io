---
title: "LeetCode 846. 一手顺子"
date: 2021-12-30T20:02:37+08:00
draft: false
tags: ["leetcode", "中等", "贪心", "数组", "哈希表", "排序"]
math: true
---

Alice 手中有一把牌，她想要重新排列这些牌，分成若干组，使每一组的牌数都是 `groupSize` ，并且由 `groupSize` 张连续的牌组成。

给你一个整数数组 `hand` 其中 `hand[i]` 是写在第 `i` 张牌，和一个整数 `groupSize` 。如果她可能重新排列这些牌，返回 `true` ；否则，返回 `false` 。

<!--more-->

**示例 1：**

> **输入：** hand = [1,2,3,6,2,3,4,7,8], groupSize = 3
> 
> **输出：** true
> 
> **解释：** Alice 手中的牌可以被重新排列为 `[1,2,3]，[2,3,4]，[6,7,8]`。

**示例 2：**

> **输入：** hand = [1,2,3,4,5], groupSize = 4
> 
> **输出：** false
> 
> **解释：** Alice 手中的牌无法被重新排列成几个大小为 4 的组。

**提示：**

- \\(1 <= hand.length <= 10^4\\)
- \\(0 <= hand[i] <= 10^9\\)
- `1 <= groupSize <= hand.length`

```cpp
class Solution {
    public:
        bool isNStraightHand(vector < int > & hand, int groupSize) {
            if (hand.size() % groupSize != 0) {
                return false;
            }
            if (groupSize == 1) {
                return true;
            }
            sort(hand.begin(), hand.end());
            cout << hand.size();
            auto len = hand.size() / groupSize;
            for (int i = 0; i < len; i++) {
                int current = * hand.begin();
                hand.erase(hand.begin());
                for (int j = 1; j < groupSize; j++) {
                    auto next = find(hand.begin(), hand.end(), current + 1);
                    if (next == hand.end()) {
                        return false;
                    } else {
                        current = * next;
                        hand.erase(next);
                    }
                }
            }
            return true;
        }
};
```
