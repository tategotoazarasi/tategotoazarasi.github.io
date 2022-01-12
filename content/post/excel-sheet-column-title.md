---
title: "LeetCode 168. Excel表列名称"
date: 2021-12-25T10:33:54+08:00
draft: false
tags: ["leetcode", "简单", "数学", "字符串"]
math: true
---

给你一个整数 `columnNumber` ，返回它在 Excel 表中相对应的列名称。

<!--more-->

例如：

A -> 1
B -> 2
C -> 3
...
Z -> 26
AA -> 27
AB -> 28
...

**示例 1：**

> **输入：** columnNumber = 1
>
> **输出：** "A"

**示例 2：**

> **输入：** columnNumber = 28
>
> **输出：** "AB"

**示例 3：**

> **输入：** columnNumber = 701
>
> **输出：** "ZY"

**示例 4：**

> **输入：** columnNumber = 2147483647
>
> **输出：** "FXSHRXW"

**提示：**

- \\(1 <= columnNumber <= 2^31 - 1\\)

## 题解

如果列名称的长度为 n，每一位对应的序号为 \\([a_{n-1}, a_{n-2}, \ldots, a_0]\\)，其中对于任意 \\(0 \le i < n\\) 都有 \\(1 \le a_i \le 26\\)，则列名称对应的列序号为：

\\(\textit{number} = \sum\_{i=0}^{n-1} a_i \times 26^i\\)

将列序号转换成列名称，则是在已知 number 的情况下，解出 \\(a\_{0}\\) 到 \\(a\_{n-1}\\) 的值。

分离出 \\(a\_{0}\\) 项，提出其余项的公因数 26，上式可以改写为：

\\(\textit{number} = a_0 + 26 \times \sum\_{i=1}^{n-1} a_i \times 26^{i-1}\\)

将等式两边同时减 1，得：

\\( \textit{number} - 1 = (a_0 - 1) + 26 \times \Big(\sum\_{i=1}^{n-1} a_i \times 26^{i-1}\Big) \\)

由于 \\(0 \le a_0 - 1 \le 25\\)，由上式可知，\\(a_0 - 1\\) 是 number−1 除以 26 的余数。

这样我们就得到了 \\(a\_{0}\\) 的值。

在得到 \\(a\_{0}\\) 的值之后，令 \\(\textit{number}' = \dfrac{\textit{number} - a_0}{26}\\)，则有：

\\(\textit{number}' = \sum\_{i=1}^{n-1} a_i \times 26^{i-1} = a_1 + 26 \times \sum\_{i=2}^{n-1} a_i \times 26^{i-2}\\)

于是使用同样的方法，可以得到 \\(a_1\\) 的值。

上述过程是一个循环的过程，直至 number′=0 时停止。此时我们就得到了 \\(a\_{0}\\) 到 \\(a\_{n-1}\\) 的值。拼接这些值对应的字母，即得到了答案。

代码实现时，由于我们计算列名称的顺序是从右往左，因此需要将拼接后的结果反转。

```cpp
string convertToTitle(int columnNumber) {
    auto ans   = string();
    bool round = false;
    while(columnNumber != 0) {
        char ch = 0;
        if(round) {
            ch    = static_cast<char>(columnNumber % 26 + 63);
            round = false;
        } else {
            ch = static_cast<char>(columnNumber % 26 + 64);
        }
        if(ch == '@' && columnNumber >= 26) {
            ch    = 'Z';
            round = true;
        } else if(ch == '?' && columnNumber >= 26) {
            ch    = 'Y';
            round = true;
        }
        if('A' <= ch && ch <= 'Z') {
            ans.insert(0, 1, ch);
        }
        columnNumber /= 26;
    }
    return ans;
}
```
