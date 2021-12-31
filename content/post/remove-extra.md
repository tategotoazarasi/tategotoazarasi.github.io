---
title: "LintCode 686. 删除多余的空格"
date: 2021-12-31T09:19:41+08:00
draft: false
tags: ["lintcode", "简单", "字符串", "模拟"]
math: true
---

从句子中删除多余空格

<!--more-->

## 样例

**样例 1:**

```
输入: s = "The  sky   is blue"
输出: "The sky is blue"
```

**样例 2:**

```
输入: s = "  low               ercase  "
输出: "low ercase"
```

```cpp
class Solution {
    public:
        string removeExtra(string & s) {
            ostringstream output = ostringstream();
            bool start = true;
            bool flag = false;
            for (char c: s) {
                if (c != ' ') {
                    if (flag && !start) {
                        output << ' ';
                    }
                    output << c;
                    start = false;
                    flag = false;
                } else {
                    flag = true;
                }
            }
            return output.str();
        }
};
```
