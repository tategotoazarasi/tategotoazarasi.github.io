---
title: "LeetCode 168. Excel表列名称"
date: 2021-12-24T13:18:18+08:00
draft: false
tags: ["leetcode", "简单", "数学", "字符串"]
math: true
---

给你一个整数  `columnNumber` ，返回它在 Excel 表中相对应的列名称。

例如：

A -> 1
B -> 2
C -> 3
...
Z -> 26
AA -> 27
AB -> 28
...

<!--more-->

**示例 1：**

**输入：** columnNumber = 1
**输出：** "A"

**示例 2：**

**输入：** columnNumber = 28
**输出：** "AB"

**示例 3：**

**输入：** columnNumber = 701
**输出：** "ZY"

**示例 4：**

**输入：** columnNumber = 2147483647
**输出：** "FXSHRXW"

**提示：**

- \\(1 <= columnNumber <= 2^31 - 1\\)

```cpp
class Solution {
public:
    string convertToTitle(int columnNumber) {
        string ans = string();
        bool round = false;
        while (columnNumber != 0)
        {
            char ch;
            if (round)
            {
                ch = static_cast<char>(columnNumber % 26 + 63);
                round = false;
            }
            else ch = static_cast<char>(columnNumber % 26 + 64);
            if (ch == '@' && columnNumber >= 26)
            {
                ch = 'Z';
                round = true;
            }
            else if (ch == '?' && columnNumber >= 26)
            {
                ch = 'Y';
                round = true;
            }
            if ('A' <= ch && ch <= 'Z')
            {
                ans.insert(0, 1, ch);
            }
            columnNumber /= 26;
        }
        return ans;
    }
};
```
