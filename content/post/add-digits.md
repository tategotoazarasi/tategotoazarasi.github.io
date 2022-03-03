---
title: "LeetCode 258. Add Digits"
date: 2022-03-03T13:13:27+08:00
draft: false
tags: ["leetcode", "简单", "数学", "数论", "模拟"]
math: true
---

Given an integer `num`, repeatedly add all its digits until the result has only one digit, and return it.

<!--more-->

**Example 1:**

**Input:** num = 38
**Output:** 2
**Explanation:** The process is
38 --> 3 + 8 --> 11
11 --> 1 + 1 --> 2
Since 2 has only one digit, return it.

**Example 2:**

**Input:** num = 0
**Output:** 0

**Constraints:**

- \\(0 <= num <= 2^{31} - 1\\)

**Follow up:**

- Could you do it without any loop/recursion in `O(1)` runtime?
- A naive implementation of the above process is trivial. Could you come up with other methods?
- What are all the possible results?
- How do they occur, periodically or randomly?
- You may find this [Wikipedia article](https://en.wikipedia.org/wiki/Digital_root) useful.

## Solution

假设整数 num 的十进制表示有 n 位，从最低位到最高位依次是 \\(a_0\\)​ 到 \\(a\_{n−1}\\)​，则 num 可以写成如下形式：

\\(\textit{num} \\)

\\(= \sum\_{i = 0}^{n - 1} a_i \times 10^i \\)

\\(= \sum\_{i = 0}^{n - 1} a_i \times (10^i - 1 + 1) \\)

\\(= \sum\_{i = 0}^{n - 1} a_i \times (10^i - 1) + \sum\_{i = 0}^{n - 1} a_i\\)

当 i=0 时，\\(10^i−1=0\\) 是 9 的倍数；当 i 是正整数时，\\(10^i−1\\) 是由 i 位 9 组成的整数，也是 9 的倍数。因此对于任意非负整数 i，\\(10^i−1\\) 都是 9 的倍数。由此可得 num 与其各位相加的结果模 9 同余。重复计算各位相加的结果直到结果为一位数时，该一位数即为 num 的数根，num 与其数根模 9 同余。

我们对 num 分类讨论：

- num 不是 9 的倍数时，其数根即为 num 除以 9 的余数。

- num 是 9 的倍数时：

  - 如果 num=0，则其数根是 0；

  - 如果 num>0，则各位相加的结果大于 0，其数根也大于 0，因此其数根是 9。

**细节**

根据上述分析可知，当 num>0 时，其数根的结果在范围 [1,9] 内，因此可以想到计算 num−1 除以 9 的余数然后加 1。由于当 num>0 时，num−1≥0，非负数除以 9 的余数一定也是非负数，因此计算 num−1 除以 9 的余数然后加 1 的结果是正确的。

当 num=0 时，num−1=−1<0，负数对 9 取余或取模的结果的正负在不同语言中有所不同。

- 对于取余的语言，结果的正负和左操作数相同，则 num−1 对 9 取余的结果为 −1，加 1 后得到结果 0，可以得到正确的结果；

- 对于取模的语言，结果的正负和右操作数相同，则 num−1 对 9 取模的结果为 8，加 1 后得到结果 9，无法得到正确的结果，此时需要对 num=0 的情况专门做处理。

**复杂度分析**

- 时间复杂度：O(1)。

- 空间复杂度：O(1)。

```cpp
class Solution {
    public int addDigits(int num) {
        return (num - 1) % 9 + 1;
    }
}
```
