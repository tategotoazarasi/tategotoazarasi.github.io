---
title: "LeetCode 1231. 分享巧克力"
date: 2022-08-25T16:59:33+08:00
draft: false
math: true
tags: ["leetcode", "困难", "数组", "二分查找"]
---

你有一大块巧克力，它由一些甜度不完全相同的小块组成。我们用数组  `sweetness`  来表示每一小块的甜度。

你打算和  `K`  名朋友一起分享这块巧克力，所以你需要将切割  `K`  次才能得到`K+1`  块，每一块都由一些 **连续**的小块组成。

为了表现出你的慷慨，你将会吃掉  **总甜度最小**的一块，并将其余几块分给你的朋友们。

请找出一个最佳的切割策略，使得你所分得的巧克力  **总甜度最大**，并返回这个**最大总甜度**。

<!--more-->

**示例 1：**

    输入：sweetness = [1,2,3,4,5,6,7,8,9], K = 5
    输出：6
    解释：你可以把巧克力分成 [1,2,3], [4,5], [6], [7], [8], [9]。

**示例 2：**

    输入：sweetness = [5,6,7,8,9,1,2,3,4], K = 8
    输出：1
    解释：只有一种办法可以把巧克力分成 9 块。

**示例 3：**

    输入：sweetness = [1,2,2,1,2,2,1,2,2], K = 2
    输出：5
    解释：你可以把巧克力分成 [1,2,2], [1,2,2], [1,2,2]。

**提示：**

- \\(0 <= K < sweetness.length <= 10^4\\)
- \\(1 <= sweetness[i] <= 10^5\\)

## 题解

对甜度进行二分，初始左值 l 为甜度的最小值，右值 r 为所有甜度的总和，判断每一个甜度最少能被切成多少份。

```cpp
class Solution {
public:
  int maximizeSweetness(vector<int> &sweetness, int k) {
    int l = sweetness[0];
    int r = 0;
    for (auto s : sweetness) {
      l = min(l, s);
      r += s;
    }
    while (l < r) {
      if (l + 1 == r) {
        return (count(sweetness, r) == k + 1) ? r : l;
      }
      int m = (l + r) / 2;
      int c = count(sweetness, m);
      if (c < k + 1) {
        r = m - 1;
      } else {
        l = m;
      }
    }
    return l;
  }

  int count(vector<int> &sweetness, int x) {
    int ans = 0;
    int current = 0;
    for (auto s : sweetness) {
      current += s;
      if (current >= x) {
        current = 0;
        ans++;
      }
    }
    return ans;
  }
};
```
