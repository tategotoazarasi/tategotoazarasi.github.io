---
title: "LeetCode 5948. 判断一个括号字符串是否有效"
date: 2021-12-26T09:22:53+08:00
draft: false
tags: ["leetcode", "中等"]
math: true
---

一个括号字符串是只由  `'('` 和  `')'`  组成的  **非空**  字符串。如果一个字符串满足下面 **任意**  一个条件，那么它就是有效的：

- 字符串为  `()`.
- 它可以表示为 `AB`（`A`  与  `B`  连接），其中`A` 和  `B`  都是有效括号字符串。
- 它可以表示为  `(A)` ，其中  `A`  是一个有效括号字符串。

给你一个括号字符串  `s`  和一个字符串  `locked` ，两者长度都为  `n` 。`locked`  是一个二进制字符串，只包含  `'0'`  和  `'1'` 。对于  `locked`  中  **每一个**  下标  `i` ：

- 如果  `locked[i]`  是  `'1'` ，你 **不能**  改变  `s[i]` 。
- 如果  `locked[i]`  是  `'0'` ，你  **可以**  将  `s[i]`  变为  `'('`  或者  `')'` 。

如果你可以将 `s`  变为有效括号字符串，请你返回  `true` ，否则返回  `false` 。

<!--more-->

**示例 1：**

![](https://tategotoazarasi.github.io/images/eg1.png)

**输入：** s = "))()))", locked = "010100"
**输出：** true
**解释：** locked[1] == '1' 和 locked[3] == '1' ，所以我们无法改变 s[1] 或者 s[3] 。
我们可以将 s[0] 和 s[4] 变为 '(' ，不改变 s[2] 和 s[5] ，使 s 变为有效字符串。

**示例 2：**

**输入：** s = "()()", locked = "0000"
**输出：** true
**解释：** 我们不需要做任何改变，因为 s 已经是有效字符串了。

**示例 3：**

**输入：** s = ")", locked = "0"
**输出：** false
**解释：** locked 允许改变 s[0] 。
但无论将 s[0] 变为 '(' 或者 ')' 都无法使 s 变为有效字符串。

**提示：**

- `n == s.length == locked.length`
- \\(1 <= n <= 10^5\\)
- `s[i]`  要么是  `'('`  要么是  `')'` 。
- `locked[i]` 要么是  `'0'`  要么是  `'1'` 。
- 一个奇数长度的字符串能有效吗？
- 从左到右，如果遇到一个锁定的')'，它必须与锁定的'('或其左边的一个未锁定的索引相平衡。如果两者都不存在，可以得出什么结论？如果两者都存在，哪一个更适合使用？
- 经过上述处理，我们可能有锁定的'('的指数和额外的未锁定的指数。你现在如何平衡锁定的'('？如果你不能平衡任何锁定的'('怎么办？

```java
class Solution {
    public boolean canBeValid(String s, String locked) {
        int left = 0;
        int right = 0;
        for (int i = 0; i < s.length(); i++) {
            if ((s.charAt(i) == '(' && locked.charAt(i) == '1') || left == 0) {
                left++;
            } else {
                left--;
            }

            if (s.charAt(i) == ')' && locked.charAt(i) == '1') {
                right--;
            } else {
                right++;
            }
            if (right < 0) {
                return false;
            }
        }
        return left == 0;
    }
}
```
