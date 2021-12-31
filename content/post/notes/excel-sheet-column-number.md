---
title: excel-sheet-column-number
tags: [Import-7704]
created: '2021-12-25T02:33:54.475Z'
modified: '2021-12-30T13:58:00.821Z'
---

---
title: "LeetCode 171. Excel 表列序号"
date: 2021-12-25T10:33:54+08:00
draft: false
tags: ["leetcode", "简单", "数学", "字符串"]
math: true
---

给你一个字符串  `columnTitle` ，表示 Excel 表格中的列名称。返回该列名称对应的列序号。

例如，

    A -> 1
    B -> 2
    C -> 3
    ...
    Z -> 26
    AA -> 27
    AB -> 28
    ...

<!--more-->

**示例 1:**

> **输入:** columnTitle = "A"
>
> **输出:** 1

**示例  2:**

> **输入:** columnTitle = "AB"
>
> **输出:** 28

**示例  3:**

> **输入:** columnTitle = "ZY"
>
> **输出:** 701

**示例 4:**

> **输入:** columnTitle = "FXSHRXW"
>
> **输出:** 2147483647

**提示：**

- `1 <= columnTitle.length <= 7`
- `columnTitle` 仅由大写英文组成
- `columnTitle` 在范围 `["A", "FXSHRXW"]` 内

```cpp
class Solution {
public:
    int titleToNumber(string columnTitle) {
        int sum = 0;
        int length = static_cast<int>(columnTitle.length());
        for (char c: columnTitle) {
            sum += static_cast<int>(static_cast<double>(c - 'A' + 1) * pow(26, length-- - 1));
        }
        return sum;
    }
};
```
